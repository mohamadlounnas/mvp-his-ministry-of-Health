# Healthcare Standards Compliance
# الامتثال للمعايير الصحية

## 1. المعايير الإلزامية | Mandatory Standards

---

## 1.1 ICD-11 - التصنيف الدولي للأمراض

### 1.1.1 نظرة عامة
| العنصر | القيمة |
|--------|--------|
| الإصدار | ICD-11 (2024 Release) |
| تاريخ السريان | يناير 2022 |
| المصدر | منظمة الصحة العالمية (WHO) |
| اللغات | العربية، الفرنسية، الإنجليزية، +7 أخرى |
| الدول المطبقة | 132 دولة |

### 1.1.2 هيكل ICD-11
```
ICD-11 Structure:
├── 01 - Infectious/parasitic diseases
├── 02 - Neoplasms
├── 03 - Blood diseases
├── 04 - Immune system disorders
├── 05 - Endocrine/nutritional/metabolic
├── 06 - Mental/behavioural/neurodevelopmental
├── 07 - Sleep-wake disorders
├── 08 - Nervous system diseases
├── 09 - Visual system diseases
├── 10 - Ear/mastoid diseases
├── 11 - Circulatory system diseases
├── 12 - Respiratory system diseases
├── 13 - Digestive system diseases
├── 14 - Skin diseases
├── 15 - Musculoskeletal diseases
├── 16 - Genitourinary system diseases
├── 17 - Sexual health conditions
├── 18 - Pregnancy/childbirth/puerperium
├── 19 - Perinatal conditions
├── 20 - Developmental anomalies
├── 21 - Symptoms/signs/abnormal findings
├── 22 - Injury/poisoning/external causes
├── 23 - External causes of morbidity/mortality
├── 24 - Factors influencing health status
├── 25 - Codes for special purposes
├── 26 - Traditional medicine conditions
├── V - Supplementary section (Functioning)
└── X - Extension codes
```

### 1.1.3 التطبيق في GHIS
```typescript
// مثال على هيكل تخزين ICD-11
interface ICD11Code {
  code: string;           // e.g., "BA00" (Essential hypertension)
  title: {
    ar: string;          // ارتفاع ضغط الدم الأساسي
    fr: string;          // Hypertension essentielle
    en: string;          // Essential hypertension
  };
  chapter: string;        // "11" (Circulatory system)
  blockId: string;        // "BA00-BA9Z"
  parent?: string;        // Parent code
  children?: string[];    // Child codes
  extensions?: string[];  // X-codes
  postcoordination?: PostcoordinationRule[];
}

// جدول PostgreSQL
CREATE TABLE icd11_codes (
  id SERIAL PRIMARY KEY,
  code VARCHAR(20) UNIQUE NOT NULL,
  title_ar TEXT NOT NULL,
  title_fr TEXT NOT NULL,
  title_en TEXT NOT NULL,
  chapter VARCHAR(5) NOT NULL,
  block_id VARCHAR(20),
  parent_code VARCHAR(20) REFERENCES icd11_codes(code),
  is_leaf BOOLEAN DEFAULT true,
  is_residual BOOLEAN DEFAULT false,
  extensions JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_icd11_code ON icd11_codes(code);
CREATE INDEX idx_icd11_title_ar ON icd11_codes USING GIN(to_tsvector('arabic', title_ar));
CREATE INDEX idx_icd11_title_fr ON icd11_codes USING GIN(to_tsvector('french', title_fr));
```

### 1.1.4 API لجلب ICD-11
```yaml
Endpoints:
  - GET /api/icd11/search?q={query}&lang={ar|fr|en}
  - GET /api/icd11/code/{code}
  - GET /api/icd11/chapter/{chapter}
  - GET /api/icd11/autocomplete?q={query}
```

---

## 1.2 HL7 FHIR R5 - معيار التشغيل البيني

### 1.2.1 موارد FHIR المطبقة

#### Administration Resources
```typescript
// Patient Resource
interface FHIRPatient {
  resourceType: "Patient";
  id: string;
  identifier: Identifier[];
  active: boolean;
  name: HumanName[];
  telecom: ContactPoint[];
  gender: "male" | "female" | "other" | "unknown";
  birthDate: string;
  deceasedBoolean?: boolean;
  deceasedDateTime?: string;
  address: Address[];
  maritalStatus?: CodeableConcept;
  multipleBirth?: boolean | number;
  photo?: Attachment[];
  contact?: PatientContact[];
  communication?: PatientCommunication[];
  generalPractitioner?: Reference[];
  managingOrganization?: Reference;
  link?: PatientLink[];
}

// Practitioner Resource
interface FHIRPractitioner {
  resourceType: "Practitioner";
  id: string;
  identifier: Identifier[];
  active: boolean;
  name: HumanName[];
  telecom: ContactPoint[];
  address: Address[];
  gender: "male" | "female" | "other" | "unknown";
  birthDate?: string;
  photo?: Attachment[];
  qualification: PractitionerQualification[];
  communication?: CodeableConcept[];
}

// Organization Resource
interface FHIROrganization {
  resourceType: "Organization";
  id: string;
  identifier: Identifier[];
  active: boolean;
  type: CodeableConcept[];
  name: string;
  alias?: string[];
  telecom: ContactPoint[];
  address: Address[];
  partOf?: Reference;
  contact?: OrganizationContact[];
  endpoint?: Reference[];
}

// Location Resource
interface FHIRLocation {
  resourceType: "Location";
  id: string;
  identifier: Identifier[];
  status: "active" | "suspended" | "inactive";
  operationalStatus?: Coding;
  name: string;
  alias?: string[];
  description?: string;
  mode: "instance" | "kind";
  type: CodeableConcept[];
  telecom: ContactPoint[];
  address: Address;
  physicalType?: CodeableConcept;
  position?: LocationPosition;
  managingOrganization?: Reference;
  partOf?: Reference;
  hoursOfOperation?: LocationHoursOfOperation[];
  availabilityExceptions?: string;
  endpoint?: Reference[];
}
```

#### Clinical Resources
```typescript
// Encounter Resource (الزيارة)
interface FHIREncounter {
  resourceType: "Encounter";
  id: string;
  identifier: Identifier[];
  status: "planned" | "arrived" | "triaged" | "in-progress" | 
          "onleave" | "finished" | "cancelled" | "entered-in-error";
  class: Coding[];  // AMB (ambulatory), EMER (emergency), IMP (inpatient), etc.
  type: CodeableConcept[];
  serviceType?: CodeableConcept;
  priority?: CodeableConcept;
  subject: Reference;  // Patient
  episodeOfCare?: Reference[];
  basedOn?: Reference[];
  participant: EncounterParticipant[];
  appointment?: Reference[];
  period?: Period;
  length?: Duration;
  reason?: EncounterReason[];
  diagnosis?: EncounterDiagnosis[];
  account?: Reference[];
  hospitalization?: EncounterHospitalization;
  location?: EncounterLocation[];
  serviceProvider?: Reference;
  partOf?: Reference;
}

// Condition Resource (التشخيص/المرض)
interface FHIRCondition {
  resourceType: "Condition";
  id: string;
  identifier: Identifier[];
  clinicalStatus: CodeableConcept;
  verificationStatus: CodeableConcept;
  category: CodeableConcept[];
  severity?: CodeableConcept;
  code: CodeableConcept;  // ICD-11 or SNOMED CT
  bodySite?: CodeableConcept[];
  subject: Reference;  // Patient
  encounter?: Reference;
  onsetDateTime?: string;
  onsetAge?: Age;
  onsetPeriod?: Period;
  abatementDateTime?: string;
  recordedDate?: string;
  recorder?: Reference;
  asserter?: Reference;
  stage?: ConditionStage[];
  evidence?: ConditionEvidence[];
  note?: Annotation[];
}

// Observation Resource (الملاحظة/القياس)
interface FHIRObservation {
  resourceType: "Observation";
  id: string;
  identifier: Identifier[];
  basedOn?: Reference[];
  partOf?: Reference[];
  status: "registered" | "preliminary" | "final" | "amended" | "corrected" |
          "cancelled" | "entered-in-error" | "unknown";
  category: CodeableConcept[];  // vital-signs, laboratory, imaging, etc.
  code: CodeableConcept;  // LOINC code
  subject: Reference;
  focus?: Reference[];
  encounter?: Reference;
  effectiveDateTime?: string;
  effectivePeriod?: Period;
  issued?: string;
  performer?: Reference[];
  valueQuantity?: Quantity;
  valueCodeableConcept?: CodeableConcept;
  valueString?: string;
  valueBoolean?: boolean;
  valueInteger?: number;
  valueRange?: Range;
  valueRatio?: Ratio;
  dataAbsentReason?: CodeableConcept;
  interpretation?: CodeableConcept[];
  note?: Annotation[];
  bodySite?: CodeableConcept;
  method?: CodeableConcept;
  specimen?: Reference;
  device?: Reference;
  referenceRange?: ObservationReferenceRange[];
  component?: ObservationComponent[];
}
```

#### Medications Resources
```typescript
// MedicationRequest Resource (الوصفة الطبية)
interface FHIRMedicationRequest {
  resourceType: "MedicationRequest";
  id: string;
  identifier: Identifier[];
  status: "active" | "on-hold" | "cancelled" | "completed" | 
          "entered-in-error" | "stopped" | "draft" | "unknown";
  statusReason?: CodeableConcept;
  intent: "proposal" | "plan" | "order" | "original-order" | 
          "reflex-order" | "filler-order" | "instance-order" | "option";
  category: CodeableConcept[];
  priority?: "routine" | "urgent" | "asap" | "stat";
  doNotPerform?: boolean;
  medication: CodeableReference;  // Reference to Medication
  subject: Reference;  // Patient
  encounter?: Reference;
  supportingInformation?: Reference[];
  authoredOn?: string;
  requester?: Reference;  // Practitioner
  performer?: Reference[];
  performerType?: CodeableConcept;
  recorder?: Reference;
  reason?: CodeableReference[];
  courseOfTherapyType?: CodeableConcept;
  insurance?: Reference[];
  note?: Annotation[];
  dosageInstruction: Dosage[];
  dispenseRequest?: MedicationRequestDispenseRequest;
  substitution?: MedicationRequestSubstitution;
  priorPrescription?: Reference;
  detectedIssue?: Reference[];
  eventHistory?: Reference[];
}
```

#### Financial Resources
```typescript
// Claim Resource (المطالبة/الفاتورة)
interface FHIRClaim {
  resourceType: "Claim";
  id: string;
  identifier: Identifier[];
  status: "active" | "cancelled" | "draft" | "entered-in-error";
  type: CodeableConcept;  // institutional, oral, pharmacy, professional, vision
  subType?: CodeableConcept;
  use: "claim" | "preauthorization" | "predetermination";
  patient: Reference;
  billablePeriod?: Period;
  created: string;
  enterer?: Reference;
  insurer?: Reference;
  provider: Reference;
  priority: CodeableConcept;
  fundsReserve?: CodeableConcept;
  related?: ClaimRelated[];
  prescription?: Reference;
  originalPrescription?: Reference;
  payee?: ClaimPayee;
  referral?: Reference;
  facility?: Reference;
  careTeam?: ClaimCareTeam[];
  supportingInfo?: ClaimSupportingInfo[];
  diagnosis?: ClaimDiagnosis[];
  procedure?: ClaimProcedure[];
  insurance: ClaimInsurance[];
  accident?: ClaimAccident;
  item: ClaimItem[];
  total?: Money;
}
```

### 1.2.2 FHIR API Endpoints
```yaml
Base URL: /api/fhir

Patient:
  - GET    /Patient                    # Search patients
  - GET    /Patient/{id}               # Read patient
  - POST   /Patient                    # Create patient
  - PUT    /Patient/{id}               # Update patient
  - DELETE /Patient/{id}               # Delete patient
  - GET    /Patient/{id}/_history      # Patient history

Encounter:
  - GET    /Encounter                  # Search encounters
  - GET    /Encounter/{id}             # Read encounter
  - POST   /Encounter                  # Create encounter
  - PUT    /Encounter/{id}             # Update encounter
  - GET    /Encounter?patient={id}     # Patient encounters

Observation:
  - GET    /Observation                # Search observations
  - GET    /Observation/{id}           # Read observation
  - POST   /Observation                # Create observation
  - GET    /Observation?patient={id}&category=vital-signs  # Vital signs

Condition:
  - GET    /Condition                  # Search conditions
  - GET    /Condition/{id}             # Read condition
  - POST   /Condition                  # Create condition
  - GET    /Condition?patient={id}     # Patient conditions

MedicationRequest:
  - GET    /MedicationRequest          # Search prescriptions
  - GET    /MedicationRequest/{id}     # Read prescription
  - POST   /MedicationRequest          # Create prescription
  - GET    /MedicationRequest?patient={id}&status=active  # Active prescriptions

Appointment:
  - GET    /Appointment                # Search appointments
  - GET    /Appointment/{id}           # Read appointment
  - POST   /Appointment                # Create appointment
  - PUT    /Appointment/{id}           # Update appointment
  - GET    /Appointment?patient={id}&date=ge{today}  # Future appointments

Bundle Operations:
  - POST   /                           # Transaction bundle
  - POST   /                           # Batch bundle
```

---

## 1.3 DICOM - معيار التصوير الطبي

### 1.3.1 نظرة عامة
```
DICOM (Digital Imaging and Communications in Medicine)
├── Image Storage Format
├── Network Protocol
├── Query/Retrieve (C-FIND, C-MOVE, C-GET)
└── PACS Integration
```

### 1.3.2 التكامل مع Orthanc PACS
```yaml
Orthanc Configuration:
  Server: Orthanc (self-hosted)
  Port: 8042 (REST API)
  DICOM Port: 4242
  Storage: MinIO/Local
  
GHIS Integration:
  - REST API for image metadata
  - DICOM Web (WADO-RS) for image retrieval
  - Integration with RIS module
```

### 1.3.3 Modalities المدعومة
| الرمز | النوع | الوصف |
|-------|-------|-------|
| CR | Computed Radiography | الأشعة السينية الرقمية |
| CT | Computed Tomography | الأشعة المقطعية |
| MR | Magnetic Resonance | الرنين المغناطيسي |
| US | Ultrasound | الموجات فوق الصوتية |
| XA | X-Ray Angiography | تصوير الأوعية |
| MG | Mammography | تصوير الثدي |
| NM | Nuclear Medicine | الطب النووي |
| PT | PET | التصوير البوزيتروني |

---

## 1.4 LOINC - ترميز المختبر

### 1.4.1 هيكل LOINC
```
LOINC Code Structure:
├── Component (ما يُقاس)
├── Property (الخاصية)
├── Time (نقطة/فترة زمنية)
├── System (العينة/النظام)
├── Scale (المقياس)
└── Method (الطريقة - اختياري)
```

### 1.4.2 أمثلة على أكواد LOINC الشائعة
```
Vital Signs:
├── 8867-4   Heart rate
├── 8310-5   Body temperature
├── 8480-6   Systolic blood pressure
├── 8462-4   Diastolic blood pressure
├── 9279-1   Respiratory rate
├── 59408-5  Oxygen saturation
└── 29463-7  Body weight

Common Lab Tests:
├── 2339-0   Glucose [Mass/volume] in Blood
├── 2345-7   Glucose [Mass/volume] in Serum/Plasma
├── 718-7    Hemoglobin [Mass/volume] in Blood
├── 4544-3   Hematocrit [Volume Fraction] of Blood
├── 6690-2   Leukocytes [#/volume] in Blood
├── 26515-7  Platelets [#/volume] in Blood
├── 2160-0   Creatinine [Mass/volume] in Serum/Plasma
└── 1742-6   Alanine aminotransferase [Enzymatic activity/volume]
```

### 1.4.3 تطبيق LOINC في GHIS
```sql
CREATE TABLE loinc_codes (
  code VARCHAR(20) PRIMARY KEY,
  long_name TEXT NOT NULL,
  short_name VARCHAR(100),
  component TEXT,
  property VARCHAR(50),
  time_aspect VARCHAR(50),
  system VARCHAR(100),
  scale_type VARCHAR(50),
  method_type VARCHAR(100),
  class VARCHAR(50),
  status VARCHAR(20),
  version VARCHAR(10),
  -- Translations
  name_ar TEXT,
  name_fr TEXT
);

-- Lab results reference LOINC
CREATE TABLE lab_results (
  id UUID PRIMARY KEY,
  order_id UUID REFERENCES lab_orders(id),
  loinc_code VARCHAR(20) REFERENCES loinc_codes(code),
  value_quantity DECIMAL,
  value_unit VARCHAR(50),
  value_string TEXT,
  reference_range_low DECIMAL,
  reference_range_high DECIMAL,
  interpretation VARCHAR(20),  -- normal, abnormal, critical
  performed_at TIMESTAMPTZ,
  performed_by UUID REFERENCES practitioners(id)
);
```

---

## 2. جدول الامتثال | Compliance Matrix

| المعيار | الإلزامية | حالة التنفيذ | الوحدات المتأثرة |
|---------|-----------|--------------|-----------------|
| ICD-11 | إلزامي | مخطط | EMR, Billing, Reports |
| HL7 FHIR R5 | إلزامي | مخطط | All modules |
| SNOMED CT | موصى به | مستقبلي | EMR, Clinical Decision |
| LOINC | إلزامي للمختبر | مخطط | LIS |
| DICOM | إلزامي للأشعة | مخطط | RIS/PACS |
| HL7 v2 | للتوافق القديم | اختياري | Legacy integration |

---

## 3. خطة التنفيذ | Implementation Plan

### المرحلة 1 (MVP)
- [ ] تحميل وفهرسة ICD-11 (العربية + الفرنسية)
- [ ] تنفيذ FHIR Resources الأساسية (Patient, Encounter, Condition)
- [ ] واجهة برمجة البحث في ICD-11

### المرحلة 2
- [ ] تحميل LOINC للمختبر
- [ ] FHIR Observation للنتائج المخبرية
- [ ] تكامل Orthanc للأشعة

### المرحلة 3
- [ ] SNOMED CT للترميز السريري المتقدم
- [ ] FHIR Bundles للعمليات المجمعة
- [ ] تصدير بيانات متوافق مع WHO

---

> **ملاحظة**: يتم تحديث هذه الوثيقة باستمرار مع تقدم التنفيذ.
