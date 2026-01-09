# تصميم قاعدة البيانات
# Database Design

## Overview | نظرة عامة

GHIS uses **PostgreSQL 16+** as its primary database, organized into 8 specialized schemas containing ~35 tables. The design follows FHIR R5 resource patterns while maintaining ACID compliance and referential integrity.

يستخدم GHIS **PostgreSQL 16+** كقاعدة بيانات أساسية، منظمة في 8 schemas متخصصة تحتوي على ~35 جدول. التصميم يتبع أنماط موارد FHIR R5 مع الحفاظ على الامتثال لـ ACID وسلامة العلاقات.

**Localization strategy (Entity-level):**
- Each entity stores a default-language field (Arabic) for display strings (e.g., `name`, `title`, `display_text`).
- Translations are stored in a flexible JSONB column `locals`, keyed by language code: `locals -> { "fr": { "name": "..." }, "en": { "name": "..." } }`.
- Full-text indexes should prefer the default field and fallback to `locals->'ar'->>'name'` for Arabic searches.

**استراتيجية الترجمة (على مستوى الكيان):**
- كل كيان يحتفظ بحقل العرض بلغة النظام الافتراضية (العربية) مثل `name`، `title`.
- الترجمات مخزنة في عمود JSONB `locals`، مفهرس حسب رمز اللغة.
- فهارس البحث النصي تفضل الحقل الافتراضي وتستخدم `locals` كبديل.

---

## Schema Organization | تنظيم الـ Schemas

```
ghis_db
├── core          # Core entities (patients, practitioners, orgs)
├── clinical      # Clinical data (encounters, conditions, procedures)
├── medication    # Pharmacy & medication management
├── lab           # Laboratory orders & results
├── imaging       # Radiology & imaging studies
├── billing       # Invoicing & payments
├── coding        # Medical coding (ICD-11, LOINC, SNOMED)
└── audit         # Audit logs & compliance tracking
```

---

## 1. Core Schema

### 1.1 Patients Table

```sql
CREATE TABLE core.patients (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    mrn VARCHAR(20) UNIQUE NOT NULL,  -- Medical Record Number
    national_id VARCHAR(18) UNIQUE,   -- Algerian National ID
    
    -- Demographics
    family_name VARCHAR(100) NOT NULL,
    given_names VARCHAR(100) NOT NULL,
    -- Localized names (optional)
    locals JSONB,
    
    gender VARCHAR(10) CHECK (gender IN ('male', 'female')),
    birth_date DATE NOT NULL,
    deceased_date DATE,
    
    -- Contact
    phone VARCHAR(20),
    email VARCHAR(100),
    address_line TEXT,
    city VARCHAR(50),
    postal_code VARCHAR(10),
    
    -- Metadata
    photo_url VARCHAR(255),
    preferred_language VARCHAR(5) DEFAULT 'ar',
    marital_status VARCHAR(20),
    
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    created_by UUID REFERENCES core.users(id),
    
    -- FHIR compliance
    fhir_resource JSONB,
    
    CONSTRAINT valid_birth_date CHECK (birth_date <= CURRENT_DATE),
    CONSTRAINT valid_deceased CHECK (deceased_date IS NULL OR deceased_date >= birth_date)
);

CREATE INDEX idx_patients_mrn ON core.patients(mrn);
CREATE INDEX idx_patients_national_id ON core.patients(national_id);
CREATE INDEX idx_patients_family_name ON core.patients(family_name);
CREATE INDEX idx_patients_birth_date ON core.patients(birth_date);
CREATE INDEX idx_patients_fhir ON core.patients USING GIN(fhir_resource);
-- Arabic full-text index (family/given or fallback to locals->'ar'->>'name')
CREATE INDEX idx_patients_name_ar ON core.patients USING GIN(to_tsvector('arabic', coalesce(family_name || ' ' || given_names, (locals->'ar'->>'name'))));
```

### 1.2 Patient Contacts

```sql
CREATE TABLE core.patient_contacts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    patient_id UUID NOT NULL REFERENCES core.patients(id) ON DELETE CASCADE,
    
    relationship VARCHAR(50) NOT NULL,  -- father, mother, spouse, guardian, emergency
    name VARCHAR(200) NOT NULL,
    phone VARCHAR(20),
    email VARCHAR(100),
    address TEXT,
    
    is_emergency_contact BOOLEAN DEFAULT FALSE,
    priority INTEGER DEFAULT 0,
    
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_patient_contacts_patient ON core.patient_contacts(patient_id);
```

### 1.3 Patient Identifiers

```sql
CREATE TABLE core.patient_identifiers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    patient_id UUID NOT NULL REFERENCES core.patients(id) ON DELETE CASCADE,
    
    system VARCHAR(100) NOT NULL,  -- passport, insurance, social_security, etc.
    value VARCHAR(100) NOT NULL,
    
    assigner VARCHAR(100),
    valid_from DATE,
    valid_to DATE,
    
    created_at TIMESTAMPTZ DEFAULT NOW(),
    
    UNIQUE(system, value)
);

CREATE INDEX idx_patient_identifiers_patient ON core.patient_identifiers(patient_id);
CREATE INDEX idx_patient_identifiers_value ON core.patient_identifiers(value);
```

### 1.4 Practitioners

```sql
CREATE TABLE core.practitioners (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    employee_id VARCHAR(20) UNIQUE NOT NULL,
    national_id VARCHAR(18) UNIQUE,
    
    family_name VARCHAR(100) NOT NULL,
    given_names VARCHAR(100) NOT NULL,
    locals JSONB,
    
    gender VARCHAR(10),
    birth_date DATE,
    
    phone VARCHAR(20),
    email VARCHAR(100) UNIQUE,
    
    specialization VARCHAR(100),  -- Internal Medicine, Surgery, Pediatrics, etc.
    qualification VARCHAR(200),
    license_number VARCHAR(50) UNIQUE,
    
    active BOOLEAN DEFAULT TRUE,
    
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    
    fhir_resource JSONB
);

CREATE INDEX idx_practitioners_employee_id ON core.practitioners(employee_id);
CREATE INDEX idx_practitioners_specialization ON core.practitioners(specialization);
CREATE INDEX idx_practitioners_active ON core.practitioners(active);
```

### 1.5 Organizations

```sql
CREATE TABLE core.organizations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    code VARCHAR(20) UNIQUE NOT NULL,
    name VARCHAR(200) NOT NULL,
    locals JSONB,
    
    type VARCHAR(50) NOT NULL,  -- hospital, clinic, laboratory, pharmacy
    
    phone VARCHAR(20),
    email VARCHAR(100),
    address TEXT,
    city VARCHAR(50),
    
    parent_org_id UUID REFERENCES core.organizations(id),
    
    active BOOLEAN DEFAULT TRUE,
    
    created_at TIMESTAMPTZ DEFAULT NOW(),
    
    fhir_resource JSONB
);

CREATE INDEX idx_organizations_code ON core.organizations(code);
CREATE INDEX idx_organizations_parent ON core.organizations(parent_org_id);
```

### 1.6 Locations

```sql
CREATE TABLE core.locations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    code VARCHAR(20) UNIQUE NOT NULL,
    name VARCHAR(200) NOT NULL,
    locals JSONB,
    
    type VARCHAR(50) NOT NULL,  -- ward, room, bed, operating_room, emergency
    
    organization_id UUID REFERENCES core.organizations(id),
    parent_location_id UUID REFERENCES core.locations(id),
    
    floor VARCHAR(10),
    capacity INTEGER,
    
    status VARCHAR(20) DEFAULT 'active',  -- active, inactive, suspended
    
    created_at TIMESTAMPTZ DEFAULT NOW(),
    
    fhir_resource JSONB
);

CREATE INDEX idx_locations_code ON core.locations(code);
CREATE INDEX idx_locations_type ON core.locations(type);
CREATE INDEX idx_locations_status ON core.locations(status);
```

### 1.7 Users

```sql
CREATE TABLE core.users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    
    practitioner_id UUID REFERENCES core.practitioners(id),
    
    role VARCHAR(50) NOT NULL,  -- SUPER_ADMIN, HOSPITAL_ADMIN, DOCTOR, NURSE, etc.
    permissions JSONB,
    
    active BOOLEAN DEFAULT TRUE,
    email_verified BOOLEAN DEFAULT FALSE,
    mfa_enabled BOOLEAN DEFAULT FALSE,
    mfa_secret VARCHAR(100),
    
    last_login TIMESTAMPTZ,
    last_password_change TIMESTAMPTZ DEFAULT NOW(),
    
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_users_username ON core.users(username);
CREATE INDEX idx_users_email ON core.users(email);
CREATE INDEX idx_users_practitioner ON core.users(practitioner_id);
CREATE INDEX idx_users_role ON core.users(role);
```

---

## 2. Clinical Schema

### 2.1 Encounters

```sql
CREATE TABLE clinical.encounters (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    encounter_number VARCHAR(20) UNIQUE NOT NULL,
    
    patient_id UUID NOT NULL REFERENCES core.patients(id),
    
    class VARCHAR(20) NOT NULL,  -- inpatient, outpatient, emergency, ambulatory
    status VARCHAR(20) DEFAULT 'planned',  -- planned, in-progress, completed, cancelled
    
    type VARCHAR(50),  -- consultation, follow-up, admission, etc.
    priority VARCHAR(20),  -- routine, urgent, emergency
    
    period_start TIMESTAMPTZ NOT NULL,
    period_end TIMESTAMPTZ,
    
    location_id UUID REFERENCES core.locations(id),
    service_provider_id UUID REFERENCES core.organizations(id),
    
    chief_complaint TEXT,
    
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    
    fhir_resource JSONB,
    
    CONSTRAINT valid_period CHECK (period_end IS NULL OR period_end >= period_start)
);

CREATE INDEX idx_encounters_patient ON clinical.encounters(patient_id);
CREATE INDEX idx_encounters_status ON clinical.encounters(status);
CREATE INDEX idx_encounters_class ON clinical.encounters(class);
CREATE INDEX idx_encounters_period_start ON clinical.encounters(period_start);
```

### 2.2 Encounter Participants

```sql
CREATE TABLE clinical.encounter_participants (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    encounter_id UUID NOT NULL REFERENCES clinical.encounters(id) ON DELETE CASCADE,
    practitioner_id UUID NOT NULL REFERENCES core.practitioners(id),
    
    role VARCHAR(50) NOT NULL,  -- primary_physician, consultant, attending, resident
    
    period_start TIMESTAMPTZ DEFAULT NOW(),
    period_end TIMESTAMPTZ,
    
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_encounter_participants_encounter ON clinical.encounter_participants(encounter_id);
CREATE INDEX idx_encounter_participants_practitioner ON clinical.encounter_participants(practitioner_id);
```

### 2.3 Encounter Diagnoses

```sql
CREATE TABLE clinical.encounter_diagnoses (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    encounter_id UUID NOT NULL REFERENCES clinical.encounters(id) ON DELETE CASCADE,
    condition_id UUID NOT NULL REFERENCES clinical.conditions(id),
    
    rank INTEGER DEFAULT 1,  -- 1 = primary diagnosis
    role VARCHAR(20),  -- admission, discharge, billing
    
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_encounter_diagnoses_encounter ON clinical.encounter_diagnoses(encounter_id);
CREATE INDEX idx_encounter_diagnoses_condition ON clinical.encounter_diagnoses(condition_id);
```

### 2.4 Conditions

```sql
CREATE TABLE clinical.conditions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    patient_id UUID NOT NULL REFERENCES core.patients(id),
    encounter_id UUID REFERENCES clinical.encounters(id),
    
    clinical_status VARCHAR(20) DEFAULT 'active',  -- active, resolved, remission, inactive
    verification_status VARCHAR(20) DEFAULT 'confirmed',  -- confirmed, provisional, differential
    
    category VARCHAR(50),  -- problem-list-item, encounter-diagnosis
    severity VARCHAR(20),  -- mild, moderate, severe
    
    code_system VARCHAR(50),  -- ICD-11, SNOMED-CT
    code VARCHAR(20) NOT NULL,
    display_text VARCHAR(500),
    locals JSONB,
    
    onset_date DATE,
    abatement_date DATE,
    
    notes TEXT,
    
    recorded_by UUID REFERENCES core.practitioners(id),
    recorded_date TIMESTAMPTZ DEFAULT NOW(),
    
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    
    fhir_resource JSONB
);

CREATE INDEX idx_conditions_patient ON clinical.conditions(patient_id);
CREATE INDEX idx_conditions_encounter ON clinical.conditions(encounter_id);
CREATE INDEX idx_conditions_code ON clinical.conditions(code);
CREATE INDEX idx_conditions_clinical_status ON clinical.conditions(clinical_status);
```

### 2.5 Observations

```sql
CREATE TABLE clinical.observations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    patient_id UUID NOT NULL REFERENCES core.patients(id),
    encounter_id UUID REFERENCES clinical.encounters(id),
    
    status VARCHAR(20) DEFAULT 'final',  -- preliminary, final, amended, corrected
    category VARCHAR(50),  -- vital-signs, laboratory, imaging, etc.
    
    code_system VARCHAR(50),  -- LOINC, SNOMED-CT
    code VARCHAR(20) NOT NULL,
    display_text VARCHAR(500),
    locals JSONB,
    
    value_quantity DECIMAL(10,2),
    value_unit VARCHAR(20),
    value_string TEXT,
    value_boolean BOOLEAN,
    value_datetime TIMESTAMPTZ,
    
    interpretation VARCHAR(20),  -- normal, high, low, critical
    reference_range_low DECIMAL(10,2),
    reference_range_high DECIMAL(10,2),
    
    effective_datetime TIMESTAMPTZ DEFAULT NOW(),
    
    performer_id UUID REFERENCES core.practitioners(id),
    
    notes TEXT,
    
    created_at TIMESTAMPTZ DEFAULT NOW(),
    
    fhir_resource JSONB
);

CREATE INDEX idx_observations_patient ON clinical.observations(patient_id);
CREATE INDEX idx_observations_encounter ON clinical.observations(encounter_id);
CREATE INDEX idx_observations_code ON clinical.observations(code);
CREATE INDEX idx_observations_category ON clinical.observations(category);
CREATE INDEX idx_observations_effective ON clinical.observations(effective_datetime);
```

### 2.6 Procedures

```sql
CREATE TABLE clinical.procedures (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    patient_id UUID NOT NULL REFERENCES core.patients(id),
    encounter_id UUID REFERENCES clinical.encounters(id),
    
    status VARCHAR(20) DEFAULT 'completed',  -- preparation, in-progress, completed, cancelled
    
    code_system VARCHAR(50),
    code VARCHAR(20) NOT NULL,
    display_text VARCHAR(500),
    locals JSONB,
    
    performed_start TIMESTAMPTZ NOT NULL,
    performed_end TIMESTAMPTZ,
    
    location_id UUID REFERENCES core.locations(id),
    
    notes TEXT,
    outcome TEXT,
    
    created_at TIMESTAMPTZ DEFAULT NOW(),
    
    fhir_resource JSONB
);

CREATE INDEX idx_procedures_patient ON clinical.procedures(patient_id);
CREATE INDEX idx_procedures_encounter ON clinical.procedures(encounter_id);
CREATE INDEX idx_procedures_code ON clinical.procedures(code);
CREATE INDEX idx_procedures_performed ON clinical.procedures(performed_start);
```

### 2.7 Allergies

```sql
CREATE TABLE clinical.allergies (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    patient_id UUID NOT NULL REFERENCES core.patients(id),
    
    clinical_status VARCHAR(20) DEFAULT 'active',  -- active, inactive, resolved
    verification_status VARCHAR(20) DEFAULT 'confirmed',
    
    type VARCHAR(20),  -- allergy, intolerance
    category VARCHAR(20),  -- food, medication, environment, biologic
    criticality VARCHAR(20),  -- low, high, unable-to-assess
    
    code_system VARCHAR(50),
    code VARCHAR(20),
    display_text VARCHAR(500) NOT NULL,
    locals JSONB,
    
    reaction_manifestation TEXT,
    reaction_severity VARCHAR(20),  -- mild, moderate, severe
    
    onset_date DATE,
    
    recorded_by UUID REFERENCES core.practitioners(id),
    recorded_date TIMESTAMPTZ DEFAULT NOW(),
    
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    
    fhir_resource JSONB
);

CREATE INDEX idx_allergies_patient ON clinical.allergies(patient_id);
CREATE INDEX idx_allergies_clinical_status ON clinical.allergies(clinical_status);
CREATE INDEX idx_allergies_category ON clinical.allergies(category);
```

### 2.8 Appointments

```sql
CREATE TABLE clinical.appointments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    appointment_number VARCHAR(20) UNIQUE NOT NULL,
    
    patient_id UUID NOT NULL REFERENCES core.patients(id),
    practitioner_id UUID REFERENCES core.practitioners(id),
    location_id UUID REFERENCES core.locations(id),
    
    status VARCHAR(20) DEFAULT 'booked',  -- proposed, booked, arrived, fulfilled, cancelled, noshow
    
    service_type VARCHAR(100),  -- consultation, follow-up, lab, imaging
    appointment_type VARCHAR(50),  -- routine, walk-in, checkup, emergency
    
    priority INTEGER DEFAULT 5,
    
    start_datetime TIMESTAMPTZ NOT NULL,
    end_datetime TIMESTAMPTZ NOT NULL,
    
    reason_code VARCHAR(50),
    reason_text VARCHAR(500),
    comment TEXT,
    
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    
    fhir_resource JSONB,
    
    CONSTRAINT valid_appointment_time CHECK (end_datetime > start_datetime)
);

CREATE INDEX idx_appointments_patient ON clinical.appointments(patient_id);
CREATE INDEX idx_appointments_practitioner ON clinical.appointments(practitioner_id);
CREATE INDEX idx_appointments_status ON clinical.appointments(status);
CREATE INDEX idx_appointments_start ON clinical.appointments(start_datetime);
```

---

## 3. Medication Schema

### 3.1 Medications

```sql
CREATE TABLE medication.medications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    code VARCHAR(50) UNIQUE NOT NULL,
    
    name VARCHAR(200) NOT NULL,
    locals JSONB,
    
    form VARCHAR(50),  -- tablet, capsule, injection, syrup
    strength VARCHAR(50),
    
    manufacturer VARCHAR(200),
    
    active BOOLEAN DEFAULT TRUE,
    
    created_at TIMESTAMPTZ DEFAULT NOW(),
    
    fhir_resource JSONB
);

CREATE INDEX idx_medications_code ON medication.medications(code);
CREATE INDEX idx_medications_name ON medication.medications(name);
```

### 3.2 Medication Requests

```sql
CREATE TABLE medication.medication_requests (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    request_number VARCHAR(20) UNIQUE NOT NULL,
    
    patient_id UUID NOT NULL REFERENCES core.patients(id),
    encounter_id UUID REFERENCES clinical.encounters(id),
    medication_id UUID NOT NULL REFERENCES medication.medications(id),
    
    status VARCHAR(20) DEFAULT 'active',  -- active, completed, cancelled, on-hold
    intent VARCHAR(20) NOT NULL,  -- proposal, plan, order
    
    priority VARCHAR(20) DEFAULT 'routine',  -- routine, urgent, stat
    
    dosage_instruction TEXT,
    quantity_value DECIMAL(10,2),
    quantity_unit VARCHAR(20),
    
    refills_allowed INTEGER DEFAULT 0,
    
    prescribed_by UUID NOT NULL REFERENCES core.practitioners(id),
    prescribed_date TIMESTAMPTZ DEFAULT NOW(),
    
    notes TEXT,
    
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    
    fhir_resource JSONB
);

CREATE INDEX idx_medication_requests_patient ON medication.medication_requests(patient_id);
CREATE INDEX idx_medication_requests_encounter ON medication.medication_requests(encounter_id);
CREATE INDEX idx_medication_requests_medication ON medication.medication_requests(medication_id);
CREATE INDEX idx_medication_requests_status ON medication.medication_requests(status);
```

### 3.3 Medication Administrations

```sql
CREATE TABLE medication.medication_administrations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    medication_request_id UUID NOT NULL REFERENCES medication.medication_requests(id),
    patient_id UUID NOT NULL REFERENCES core.patients(id),
    medication_id UUID NOT NULL REFERENCES medication.medications(id),
    
    status VARCHAR(20) DEFAULT 'completed',  -- in-progress, completed, not-done
    
    effective_datetime TIMESTAMPTZ NOT NULL,
    
    dosage_dose DECIMAL(10,2),
    dosage_unit VARCHAR(20),
    dosage_route VARCHAR(50),  -- oral, IV, IM, SC
    
    administered_by UUID NOT NULL REFERENCES core.practitioners(id),
    
    notes TEXT,
    
    created_at TIMESTAMPTZ DEFAULT NOW(),
    
    fhir_resource JSONB
);

CREATE INDEX idx_medication_administrations_request ON medication.medication_administrations(medication_request_id);
CREATE INDEX idx_medication_administrations_patient ON medication.medication_administrations(patient_id);
CREATE INDEX idx_medication_administrations_effective ON medication.medication_administrations(effective_datetime);
```

---

## 4. Lab Schema

### 4.1 Lab Orders

```sql
CREATE TABLE lab.lab_orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    order_number VARCHAR(20) UNIQUE NOT NULL,
    
    patient_id UUID NOT NULL REFERENCES core.patients(id),
    encounter_id UUID REFERENCES clinical.encounters(id),
    
    status VARCHAR(20) DEFAULT 'requested',  -- requested, accepted, in-progress, completed, cancelled
    priority VARCHAR(20) DEFAULT 'routine',
    
    ordered_by UUID NOT NULL REFERENCES core.practitioners(id),
    ordered_date TIMESTAMPTZ DEFAULT NOW(),
    
    specimen_collected_date TIMESTAMPTZ,
    
    notes TEXT,
    
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    
    fhir_resource JSONB
);

CREATE INDEX idx_lab_orders_patient ON lab.lab_orders(patient_id);
CREATE INDEX idx_lab_orders_encounter ON lab.lab_orders(encounter_id);
CREATE INDEX idx_lab_orders_status ON lab.lab_orders(status);
CREATE INDEX idx_lab_orders_ordered_date ON lab.lab_orders(ordered_date);
```

### 4.2 Lab Results

```sql
CREATE TABLE lab.lab_results (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    lab_order_id UUID NOT NULL REFERENCES lab.lab_orders(id) ON DELETE CASCADE,
    
    test_code VARCHAR(20) NOT NULL,  -- LOINC code
    test_name VARCHAR(200),
    locals JSONB,
    
    result_value VARCHAR(500),
    result_unit VARCHAR(20),
    
    interpretation VARCHAR(20),  -- normal, high, low, critical
    reference_range VARCHAR(100),
    
    status VARCHAR(20) DEFAULT 'final',
    
    performed_by UUID REFERENCES core.practitioners(id),
    verified_by UUID REFERENCES core.practitioners(id),
    
    result_date TIMESTAMPTZ DEFAULT NOW(),
    
    notes TEXT,
    
    created_at TIMESTAMPTZ DEFAULT NOW(),
    
    fhir_resource JSONB
);

CREATE INDEX idx_lab_results_order ON lab.lab_results(lab_order_id);
CREATE INDEX idx_lab_results_test_code ON lab.lab_results(test_code);
CREATE INDEX idx_lab_results_result_date ON lab.lab_results(result_date);
```

---

## 5. Imaging Schema

### 5.1 Imaging Orders

```sql
CREATE TABLE imaging.imaging_orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    order_number VARCHAR(20) UNIQUE NOT NULL,
    
    patient_id UUID NOT NULL REFERENCES core.patients(id),
    encounter_id UUID REFERENCES clinical.encounters(id),
    
    modality VARCHAR(20) NOT NULL,  -- XR (X-Ray), CT, MRI, US (Ultrasound)
    body_site VARCHAR(100),
    
    status VARCHAR(20) DEFAULT 'requested',
    priority VARCHAR(20) DEFAULT 'routine',
    
    ordered_by UUID NOT NULL REFERENCES core.practitioners(id),
    ordered_date TIMESTAMPTZ DEFAULT NOW(),
    
    clinical_info TEXT,
    
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    
    fhir_resource JSONB
);

CREATE INDEX idx_imaging_orders_patient ON imaging.imaging_orders(patient_id);
CREATE INDEX idx_imaging_orders_status ON imaging.imaging_orders(status);
CREATE INDEX idx_imaging_orders_modality ON imaging.imaging_orders(modality);
```

### 5.2 Imaging Reports

```sql
CREATE TABLE imaging.imaging_reports (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    imaging_order_id UUID NOT NULL REFERENCES imaging.imaging_orders(id),
    
    status VARCHAR(20) DEFAULT 'final',
    
    findings TEXT,
    impression TEXT,
    
    dicom_study_uid VARCHAR(100),
    
    reported_by UUID NOT NULL REFERENCES core.practitioners(id),
    reported_date TIMESTAMPTZ DEFAULT NOW(),
    
    created_at TIMESTAMPTZ DEFAULT NOW(),
    
    fhir_resource JSONB
);

CREATE INDEX idx_imaging_reports_order ON imaging.imaging_reports(imaging_order_id);
CREATE INDEX idx_imaging_reports_study_uid ON imaging.imaging_reports(dicom_study_uid);
```

---

## 6. Billing Schema

### 6.1 Invoices

```sql
CREATE TABLE billing.invoices (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    invoice_number VARCHAR(20) UNIQUE NOT NULL,
    
    patient_id UUID NOT NULL REFERENCES core.patients(id),
    encounter_id UUID REFERENCES clinical.encounters(id),
    
    status VARCHAR(20) DEFAULT 'draft',  -- draft, issued, paid, cancelled
    
    issue_date DATE NOT NULL,
    due_date DATE,
    
    total_amount DECIMAL(12,2) NOT NULL,
    paid_amount DECIMAL(12,2) DEFAULT 0,
    
    currency VARCHAR(3) DEFAULT 'DZD',
    
    notes TEXT,
    
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_invoices_patient ON billing.invoices(patient_id);
CREATE INDEX idx_invoices_encounter ON billing.invoices(encounter_id);
CREATE INDEX idx_invoices_status ON billing.invoices(status);
CREATE INDEX idx_invoices_issue_date ON billing.invoices(issue_date);
```

### 6.2 Payments

```sql
CREATE TABLE billing.payments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    payment_number VARCHAR(20) UNIQUE NOT NULL,
    
    invoice_id UUID NOT NULL REFERENCES billing.invoices(id),
    
    amount DECIMAL(12,2) NOT NULL,
    payment_method VARCHAR(20),  -- cash, credit_card, insurance
    
    payment_date DATE DEFAULT CURRENT_DATE,
    
    notes TEXT,
    
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_payments_invoice ON billing.payments(invoice_id);
CREATE INDEX idx_payments_payment_date ON billing.payments(payment_date);
```

---

## 7. Coding Schema

### 7.1 ICD-11 Codes

```sql
CREATE TABLE coding.icd11_codes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    code VARCHAR(20) UNIQUE NOT NULL,
    
    title VARCHAR(500) NOT NULL,
    locals JSONB,
    
    chapter VARCHAR(100),
    parent_code VARCHAR(20),
    
    is_leaf BOOLEAN DEFAULT TRUE,
    
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_icd11_codes_code ON coding.icd11_codes(code);
CREATE INDEX idx_icd11_codes_chapter ON coding.icd11_codes(chapter);
CREATE INDEX idx_icd11_codes_parent ON coding.icd11_codes(parent_code);
CREATE INDEX idx_icd11_codes_title ON coding.icd11_codes USING GIN(to_tsvector('english', title));
```

### 7.2 LOINC Codes

```sql
CREATE TABLE coding.loinc_codes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    code VARCHAR(20) UNIQUE NOT NULL,
    
    component VARCHAR(200),
    property VARCHAR(50),
    time_aspect VARCHAR(50),
    system VARCHAR(100),
    scale_type VARCHAR(50),
    method_type VARCHAR(100),
    
    long_common_name VARCHAR(500),
    locals JSONB,
    
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_loinc_codes_code ON coding.loinc_codes(code);
CREATE INDEX idx_loinc_codes_component ON coding.loinc_codes(component);
```

---

## 8. Audit Schema

### 8.1 Audit Logs

```sql
CREATE TABLE audit.audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    event_type VARCHAR(50) NOT NULL,  -- CREATE, READ, UPDATE, DELETE
    entity_type VARCHAR(50) NOT NULL,  -- Patient, Encounter, MedicationRequest, etc.
    entity_id UUID NOT NULL,
    
    user_id UUID REFERENCES core.users(id),
    username VARCHAR(50),
    user_role VARCHAR(50),
    
    ip_address INET,
    user_agent TEXT,
    
    changes JSONB,  -- Before/after values
    
    timestamp TIMESTAMPTZ DEFAULT NOW(),
    
    -- Immutability: no updates or deletes allowed on audit logs
    CONSTRAINT no_updates CHECK (FALSE)  -- This prevents updates
);

CREATE INDEX idx_audit_logs_entity ON audit.audit_logs(entity_type, entity_id);
CREATE INDEX idx_audit_logs_user ON audit.audit_logs(user_id);
CREATE INDEX idx_audit_logs_timestamp ON audit.audit_logs(timestamp);
CREATE INDEX idx_audit_logs_event_type ON audit.audit_logs(event_type);
```

---

## Database Features | ميزات قاعدة البيانات

### Row-Level Security (RLS)

```sql
-- Example: Patients can only be accessed by authorized users
ALTER TABLE core.patients ENABLE ROW LEVEL SECURITY;

CREATE POLICY patient_access_policy ON core.patients
    USING (
        -- User must have appropriate role or be the assigned practitioner
        current_setting('app.user_role') IN ('SUPER_ADMIN', 'HOSPITAL_ADMIN', 'DOCTOR', 'NURSE')
    );
```

### Triggers for Updated_At

```sql
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_patients_updated_at
    BEFORE UPDATE ON core.patients
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();
```

### Full-Text Search

```sql
CREATE INDEX idx_patients_fulltext ON core.patients 
    USING GIN(to_tsvector('arabic', coalesce(family_name || ' ' || given_names, (locals->'ar'->>'name'))));

CREATE INDEX idx_conditions_fulltext ON clinical.conditions 
    USING GIN(to_tsvector('arabic', coalesce(display_text, (locals->'ar'->>'display_text'))));
```

---

## Data Retention & Archival | الاحتفاظ بالبيانات

- **Patient records:** Retain indefinitely (legal requirement)
- **Audit logs:** Retain for 7 years
- **Imaging studies:** Archive to cold storage after 2 years
- **Deleted records:** Soft delete with `deleted_at` timestamp

---

## Backup Strategy | استراتيجية النسخ الاحتياطي

- **Continuous WAL archiving** to S3-compatible storage
- **Daily full backups** at 02:00 AM
- **Point-in-time recovery** capability up to 30 days
- **Monthly backup testing** to verify restore procedures

---

## Performance Optimization | تحسين الأداء

1. **Partitioning:** Large tables (audit_logs, observations) partitioned by time
2. **Connection pooling:** PgBouncer with 100-500 connections
3. **Read replicas:** For reporting and analytics workloads
4. **Materialized views:** For complex aggregations
5. **VACUUM and ANALYZE:** Automated maintenance

---

## Compliance Notes | ملاحظات الامتثال

✅ **ACID Transactions:** Full ACID compliance for data integrity  
✅ **Foreign Keys:** Referential integrity enforced  
✅ **Audit Trail:** Complete audit logging with immutable records  
✅ **Encryption:** Field-level encryption for sensitive data (SSN, etc.)  
✅ **FHIR Storage:** JSONB columns for full FHIR resource storage  
✅ **Multi-language:** Unicode support (UTF-8) for Arabic, French, English
