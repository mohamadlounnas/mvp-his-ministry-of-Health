# Modules Specification
# مواصفات الوحدات

## نظرة عامة على الوحدات | Modules Overview

```
┌────────────────────────────────────────────────────────────────────┐
│                    GHIS Module Structure                            │
├────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ╔═══════════════════════════════════════════════════════════════╗ │
│  ║                   CORE FOUNDATION                              ║ │
│  ║  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────────┐  ║ │
│  ║  │   Auth   │ │  Users   │ │   Org    │ │     Settings     │  ║ │
│  ║  └──────────┘ └──────────┘ └──────────┘ └──────────────────┘  ║ │
│  ╚═══════════════════════════════════════════════════════════════╝ │
│                              │                                      │
│  ╔═══════════════════════════▼═══════════════════════════════════╗ │
│  ║                 ADMINISTRATION MODULES                         ║ │
│  ║  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────────┐  ║ │
│  ║  │ Patient  │ │Practition│ │ Location │ │  Organization    │  ║ │
│  ║  │   ADT    │ │    er    │ │          │ │   (Hospitals)    │  ║ │
│  ║  └──────────┘ └──────────┘ └──────────┘ └──────────────────┘  ║ │
│  ╚═══════════════════════════════════════════════════════════════╝ │
│                              │                                      │
│  ╔═══════════════════════════▼═══════════════════════════════════╗ │
│  ║                   CLINICAL MODULES                             ║ │
│  ║  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────────┐  ║ │
│  ║  │Appointmt │ │ Encounter│ │Condition │ │   Observation    │  ║ │
│  ║  └──────────┘ └──────────┘ └──────────┘ └──────────────────┘  ║ │
│  ║  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────────┐  ║ │
│  ║  │Procedure │ │ CarePlan │ │ Allergy  │ │   Prescription   │  ║ │
│  ║  └──────────┘ └──────────┘ └──────────┘ └──────────────────┘  ║ │
│  ╚═══════════════════════════════════════════════════════════════╝ │
│                              │                                      │
│  ╔═══════════════════════════▼═══════════════════════════════════╗ │
│  ║                  DIAGNOSTIC MODULES                            ║ │
│  ║  ┌────────────────────────┐ ┌────────────────────────────────┐ ║ │
│  ║  │   Laboratory (LIS)     │ │      Radiology (RIS/PACS)      │ ║ │
│  ║  │ • Orders               │ │ • Imaging Orders               │ ║ │
│  ║  │ • Results              │ │ • Studies                      │ ║ │
│  ║  │ • Specimens            │ │ • Reports                      │ ║ │
│  ║  │ • LOINC Integration    │ │ • DICOM/Orthanc                │ ║ │
│  ║  └────────────────────────┘ └────────────────────────────────┘ ║ │
│  ╚═══════════════════════════════════════════════════════════════╝ │
│                              │                                      │
│  ╔═══════════════════════════▼═══════════════════════════════════╗ │
│  ║                  MEDICATION MODULES                            ║ │
│  ║  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────────┐  ║ │
│  ║  │ Pharmacy │ │ Dispense │ │Immunizatn│ │    Medication    │  ║ │
│  ║  │   Stock  │ │          │ │          │ │   Administration │  ║ │
│  ║  └──────────┘ └──────────┘ └──────────┘ └──────────────────┘  ║ │
│  ╚═══════════════════════════════════════════════════════════════╝ │
│                              │                                      │
│  ╔═══════════════════════════▼═══════════════════════════════════╗ │
│  ║                   FINANCIAL MODULES                            ║ │
│  ║  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────────┐  ║ │
│  ║  │ Billing  │ │ Payments │ │Insurance │ │    Pricing       │  ║ │
│  ║  └──────────┘ └──────────┘ └──────────┘ └──────────────────┘  ║ │
│  ╚═══════════════════════════════════════════════════════════════╝ │
│                              │                                      │
│  ╔═══════════════════════════▼═══════════════════════════════════╗ │
│  ║                   SUPPORT MODULES                              ║ │
│  ║  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────────┐  ║ │
│  ║  │Inventory │ │Blood Bank│ │  Audit   │ │    Reporting     │  ║ │
│  ║  └──────────┘ └──────────┘ └──────────┘ └──────────────────┘  ║ │
│  ╚═══════════════════════════════════════════════════════════════╝ │
│                                                                     │
└────────────────────────────────────────────────────────────────────┘
```

---

## 1. وحدة المرضى (Patient Module) - ADT

### 1.1 الوصف
إدارة كاملة لدورة حياة المريض من التسجيل حتى الخروج.

### 1.2 المكونات الفرعية

```
Patient Module
├── Registration (تسجيل المريض)
│   ├── New patient registration
│   ├── Demographics capture
│   ├── ID verification
│   ├── Photo capture
│   └── Emergency contacts
│
├── Identification (التعريف)
│   ├── MRN generation (رقم الملف الطبي)
│   ├── National ID linkage
│   ├── Patient matching/deduplication
│   └── Multiple identifiers support
│
├── Search (البحث)
│   ├── By MRN
│   ├── By National ID
│   ├── By name (phonetic search)
│   ├── By phone number
│   └── Advanced filters
│
├── Medical History (السجل الطبي)
│   ├── Chronic conditions
│   ├── Allergies
│   ├── Surgeries history
│   ├── Family history
│   └── Social history
│
├── ADT Events (القبول/الخروج/النقل)
│   ├── Admission
│   ├── Discharge
│   ├── Transfer
│   └── Pre-admission
│
└── Patient Portal (بوابة المريض)
    ├── View appointments
    ├── View results
    ├── Request records
    └── Update contact info
```

### 1.3 كيانات البيانات (Entities)

```typescript
// Patient Entity
@Entity('patients')
export class Patient {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  mrn: string; // Medical Record Number

  @Column({ nullable: true, unique: true })
  nationalId: string;

  @Column({ type: 'jsonb' })
  name: {
    ar: { family: string; given: string[] };
    fr: { family: string; given: string[] };
  };

  @Column({ type: 'date' })
  birthDate: Date;

  @Column({ type: 'enum', enum: ['male', 'female'] })
  gender: 'male' | 'female';

  @Column({ type: 'jsonb', nullable: true })
  telecom: ContactPoint[];

  @Column({ type: 'jsonb', nullable: true })
  address: Address[];

  @Column({ nullable: true })
  bloodType: string;

  @Column({ type: 'jsonb', nullable: true })
  emergencyContacts: EmergencyContact[];

  @Column({ type: 'jsonb', nullable: true })
  photo: string; // MinIO object key

  @Column({ default: true })
  active: boolean;

  @Column({ type: 'boolean', default: false })
  deceased: boolean;

  @Column({ type: 'timestamp', nullable: true })
  deceasedDateTime: Date;

  @ManyToOne(() => Organization)
  managingOrganization: Organization;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  // Relations
  @OneToMany(() => Encounter, encounter => encounter.patient)
  encounters: Encounter[];

  @OneToMany(() => Appointment, appointment => appointment.patient)
  appointments: Appointment[];

  @OneToMany(() => Condition, condition => condition.patient)
  conditions: Condition[];
}

// Supporting types
interface ContactPoint {
  system: 'phone' | 'email' | 'fax';
  value: string;
  use: 'home' | 'work' | 'mobile';
  rank?: number;
}

interface Address {
  use: 'home' | 'work' | 'temp';
  line: string[];
  city: string;
  state: string; // Wilaya
  postalCode: string;
  country: string;
}

interface EmergencyContact {
  name: string;
  relationship: string;
  phone: string;
}
```

### 1.4 واجهات API

```yaml
Patient APIs:
  Registration:
    POST /api/patients:
      description: Create new patient
      body: CreatePatientDto
      response: Patient
      roles: [RECEPTIONIST, ADMIN]

    POST /api/patients/search-duplicate:
      description: Check for duplicate before registration
      body: { nationalId?, name?, birthDate? }
      response: { duplicates: Patient[], confidence: number }

  Search:
    GET /api/patients:
      description: Search patients
      params:
        - q: string (general search)
        - mrn: string
        - nationalId: string
        - phone: string
        - name: string
        - birthDate: date
        - page: number
        - limit: number
      response: PaginatedResponse<Patient>

    GET /api/patients/{id}:
      description: Get patient by ID
      response: Patient with relations

  Update:
    PATCH /api/patients/{id}:
      description: Update patient
      body: UpdatePatientDto
      response: Patient

    PUT /api/patients/{id}/photo:
      description: Upload patient photo
      body: FormData (file)
      response: { photoUrl: string }

  Medical History:
    GET /api/patients/{id}/conditions:
      description: Get patient conditions
      response: Condition[]

    GET /api/patients/{id}/allergies:
      description: Get patient allergies
      response: AllergyIntolerance[]

    GET /api/patients/{id}/timeline:
      description: Get patient timeline (all events)
      response: TimelineEvent[]

  ADT:
    POST /api/patients/{id}/admit:
      description: Admit patient (inpatient)
      body: AdmissionDto
      response: Encounter

    POST /api/patients/{id}/discharge:
      description: Discharge patient
      body: DischargeDto
      response: Encounter

    POST /api/patients/{id}/transfer:
      description: Transfer patient
      body: TransferDto
      response: Encounter
```

---

## 2. وحدة المواعيد (Appointment Module)

### 2.1 الوصف
جدولة وإدارة مواعيد المرضى مع الأطباء والخدمات.

### 2.2 المكونات الفرعية

```
Appointment Module
├── Scheduling (الجدولة)
│   ├── Calendar view
│   ├── Slot management
│   ├── Resource booking
│   └── Recurring appointments
│
├── Waitlist (قائمة الانتظار)
│   ├── Walk-in patients
│   ├── Priority queue
│   └── Estimated wait time
│
├── Reminders (التذكيرات)
│   ├── SMS notifications
│   ├── Push notifications
│   ├── Confirmation requests
│   └── No-show tracking
│
└── Check-in (تسجيل الوصول)
    ├── Self check-in (kiosk)
    ├── Staff check-in
    └── Arrival notifications
```

### 2.3 كيانات البيانات

```typescript
@Entity('appointments')
export class Appointment {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ type: 'enum', enum: AppointmentStatus })
  status: AppointmentStatus;
  // proposed | pending | booked | arrived | fulfilled | cancelled | noshow

  @Column({ type: 'enum', enum: AppointmentClass })
  appointmentType: AppointmentClass;
  // checkup | emergency | followup | routine | walkin

  @Column({ type: 'jsonb', nullable: true })
  serviceType: CodeableConcept; // Service being booked

  @Column({ type: 'text', nullable: true })
  reason: string;

  @Column({ type: 'int', nullable: true })
  priority: number; // 1-5

  @ManyToOne(() => Patient)
  patient: Patient;

  @ManyToOne(() => Practitioner)
  practitioner: Practitioner;

  @ManyToOne(() => Location, { nullable: true })
  location: Location;

  @Column({ type: 'timestamptz' })
  start: Date;

  @Column({ type: 'timestamptz' })
  end: Date;

  @Column({ type: 'int' })
  minutesDuration: number;

  @Column({ type: 'timestamptz', nullable: true })
  arrivedAt: Date;

  @Column({ type: 'text', nullable: true })
  cancellationReason: string;

  @Column({ type: 'text', nullable: true })
  patientInstruction: string;

  @ManyToOne(() => User)
  createdBy: User;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}

@Entity('schedules')
export class Schedule {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  active: boolean;

  @ManyToOne(() => Practitioner, { nullable: true })
  practitioner: Practitioner;

  @ManyToOne(() => Location, { nullable: true })
  location: Location;

  @Column({ type: 'jsonb' })
  planningHorizon: { start: Date; end: Date };

  @Column({ type: 'text', nullable: true })
  comment: string;

  @OneToMany(() => Slot, slot => slot.schedule)
  slots: Slot[];
}

@Entity('slots')
export class Slot {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @ManyToOne(() => Schedule)
  schedule: Schedule;

  @Column({ type: 'enum', enum: SlotStatus })
  status: SlotStatus;
  // busy | free | busy-unavailable | busy-tentative

  @Column({ type: 'timestamptz' })
  start: Date;

  @Column({ type: 'timestamptz' })
  end: Date;

  @Column({ type: 'boolean', default: false })
  overbooked: boolean;

  @Column({ type: 'text', nullable: true })
  comment: string;
}

enum AppointmentStatus {
  PROPOSED = 'proposed',
  PENDING = 'pending',
  BOOKED = 'booked',
  ARRIVED = 'arrived',
  FULFILLED = 'fulfilled',
  CANCELLED = 'cancelled',
  NOSHOW = 'noshow',
  ENTERED_IN_ERROR = 'entered-in-error',
  CHECKED_IN = 'checked-in',
  WAITLIST = 'waitlist'
}
```

### 2.4 واجهات API

```yaml
Appointment APIs:
  Scheduling:
    GET /api/appointments/slots:
      description: Get available slots
      params:
        - practitionerId: uuid
        - locationId: uuid
        - startDate: date
        - endDate: date
        - serviceType: string
      response: Slot[]

    POST /api/appointments:
      description: Book appointment
      body: CreateAppointmentDto
      response: Appointment

    PATCH /api/appointments/{id}:
      description: Update appointment
      body: UpdateAppointmentDto
      response: Appointment

    DELETE /api/appointments/{id}:
      description: Cancel appointment
      body: { reason: string }
      response: Appointment

  Check-in:
    POST /api/appointments/{id}/check-in:
      description: Patient check-in
      response: Appointment

    POST /api/appointments/{id}/no-show:
      description: Mark as no-show
      response: Appointment

  Calendar:
    GET /api/appointments/calendar:
      description: Get calendar view
      params:
        - view: day | week | month
        - date: date
        - practitionerId: uuid
        - locationId: uuid
      response: CalendarView

  Waitlist:
    GET /api/appointments/waitlist:
      description: Get current waitlist
      params:
        - locationId: uuid
        - practitionerId: uuid
      response: WaitlistItem[]

    POST /api/appointments/waitlist:
      description: Add to waitlist
      body: WaitlistDto
      response: WaitlistItem
```

---

## 3. وحدة الزيارات (Encounter Module)

### 3.1 الوصف
توثيق وإدارة زيارات المرضى بجميع أنواعها.

### 3.2 أنواع الزيارات

```
Encounter Classes (FHIR):
├── AMB  - Ambulatory (عيادة خارجية)
├── EMER - Emergency (طوارئ)
├── IMP  - Inpatient (إقامة داخلية)
├── ACUTE - Inpatient Acute
├── NONAC - Inpatient Non-Acute
├── OBSENC - Observation
├── PRENC - Pre-admission
├── SS   - Short Stay
└── HH   - Home Health
```

### 3.3 كيانات البيانات

```typescript
@Entity('encounters')
export class Encounter {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  encounterNumber: string;

  @Column({ type: 'enum', enum: EncounterStatus })
  status: EncounterStatus;
  // planned | arrived | triaged | in-progress | onleave | finished | cancelled

  @Column({ type: 'enum', enum: EncounterClass })
  class: EncounterClass;

  @Column({ type: 'jsonb', nullable: true })
  type: CodeableConcept[];

  @Column({ type: 'jsonb', nullable: true })
  serviceType: CodeableConcept;

  @Column({ type: 'enum', enum: Priority, nullable: true })
  priority: Priority;

  @ManyToOne(() => Patient)
  @JoinColumn({ name: 'patient_id' })
  patient: Patient;

  @ManyToOne(() => Appointment, { nullable: true })
  appointment: Appointment;

  @Column({ type: 'timestamptz' })
  periodStart: Date;

  @Column({ type: 'timestamptz', nullable: true })
  periodEnd: Date;

  @Column({ type: 'int', nullable: true })
  lengthMinutes: number;

  @Column({ type: 'jsonb', nullable: true })
  reason: CodeableConcept[];

  @ManyToOne(() => Organization)
  serviceProvider: Organization;

  @ManyToOne(() => Location, { nullable: true })
  location: Location;

  // For inpatient
  @Column({ type: 'jsonb', nullable: true })
  hospitalization: {
    preAdmissionIdentifier?: string;
    origin?: string;
    admitSource?: CodeableConcept;
    reAdmission?: CodeableConcept;
    dietPreference?: CodeableConcept[];
    specialCourtesy?: CodeableConcept[];
    specialArrangement?: CodeableConcept[];
    destination?: string;
    dischargeDisposition?: CodeableConcept;
  };

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  // Relations
  @OneToMany(() => EncounterParticipant, p => p.encounter)
  participants: EncounterParticipant[];

  @OneToMany(() => EncounterDiagnosis, d => d.encounter)
  diagnoses: EncounterDiagnosis[];

  @OneToMany(() => Observation, o => o.encounter)
  observations: Observation[];

  @OneToMany(() => MedicationRequest, m => m.encounter)
  prescriptions: MedicationRequest[];
}

@Entity('encounter_participants')
export class EncounterParticipant {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @ManyToOne(() => Encounter)
  encounter: Encounter;

  @Column({ type: 'jsonb' })
  type: CodeableConcept[];

  @Column({ type: 'timestamptz', nullable: true })
  periodStart: Date;

  @Column({ type: 'timestamptz', nullable: true })
  periodEnd: Date;

  @ManyToOne(() => Practitioner)
  individual: Practitioner;
}

@Entity('encounter_diagnoses')
export class EncounterDiagnosis {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @ManyToOne(() => Encounter)
  encounter: Encounter;

  @ManyToOne(() => Condition)
  condition: Condition;

  @Column({ type: 'enum', enum: DiagnosisUse })
  use: DiagnosisUse;
  // AD (admission) | DD (discharge) | CC (chief-complaint) | CM (comorbidity)
  // pre-op | post-op | billing

  @Column({ type: 'int' })
  rank: number;
}
```

### 3.4 سير العمل (Workflow)

```
┌─────────────────────────────────────────────────────────────────┐
│                    Encounter Workflow                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐  │
│  │ PLANNED  │───▶│ ARRIVED  │───▶│ TRIAGED  │───▶│IN-PROGRESS│  │
│  └──────────┘    └──────────┘    └──────────┘    └─────┬─────┘  │
│       │                                                │        │
│       │              ┌─────────────────────────────────┘        │
│       │              │                                          │
│       │              ▼                                          │
│       │         ┌──────────┐    ┌──────────┐                   │
│       │         │ ON-LEAVE │───▶│IN-PROGRESS│                   │
│       │         └──────────┘    └─────┬─────┘                   │
│       │                               │                         │
│       │                               ▼                         │
│       │                         ┌──────────┐                   │
│       │                         │ FINISHED │                   │
│       │                         └──────────┘                   │
│       │                                                         │
│       └────────────────────────┐                               │
│                                ▼                               │
│                          ┌──────────┐                          │
│                          │CANCELLED │                          │
│                          └──────────┘                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. وحدة المختبر (Laboratory Module - LIS)

### 4.1 الوصف
نظام معلومات المختبر الشامل لإدارة الطلبات والعينات والنتائج.

### 4.2 المكونات الفرعية

```
Laboratory Module
├── Order Management (إدارة الطلبات)
│   ├── Order creation
│   ├── Order panels (مجموعات الفحوصات)
│   ├── Priority handling
│   └── Order status tracking
│
├── Specimen Management (إدارة العينات)
│   ├── Collection
│   ├── Labeling (barcode)
│   ├── Transport
│   ├── Reception
│   └── Storage
│
├── Result Entry (إدخال النتائج)
│   ├── Manual entry
│   ├── Instrument interface
│   ├── Reference ranges
│   ├── Critical values alerts
│   └── Validation workflow
│
├── Reporting (التقارير)
│   ├── Patient reports
│   ├── Cumulative reports
│   └── Statistical reports
│
└── Quality Control (مراقبة الجودة)
    ├── QC samples
    ├── Calibration tracking
    └── Proficiency testing
```

### 4.3 كيانات البيانات

```typescript
@Entity('lab_orders')
export class LabOrder {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  orderNumber: string;

  @Column({ type: 'enum', enum: LabOrderStatus })
  status: LabOrderStatus;
  // draft | active | on-hold | revoked | completed | entered-in-error

  @Column({ type: 'enum', enum: OrderPriority })
  priority: OrderPriority;
  // routine | urgent | asap | stat

  @ManyToOne(() => Patient)
  patient: Patient;

  @ManyToOne(() => Encounter, { nullable: true })
  encounter: Encounter;

  @ManyToOne(() => Practitioner)
  requester: Practitioner;

  @Column({ type: 'timestamptz' })
  authoredOn: Date;

  @Column({ type: 'text', nullable: true })
  clinicalNote: string;

  @OneToMany(() => LabOrderItem, item => item.order)
  items: LabOrderItem[];

  @OneToMany(() => Specimen, specimen => specimen.order)
  specimens: Specimen[];
}

@Entity('lab_order_items')
export class LabOrderItem {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @ManyToOne(() => LabOrder)
  order: LabOrder;

  @Column()
  loincCode: string;

  @Column()
  testName: string;

  @Column({ type: 'enum', enum: LabOrderItemStatus })
  status: LabOrderItemStatus;

  @Column({ type: 'jsonb', nullable: true })
  reason: CodeableConcept;

  @OneToMany(() => LabResult, result => result.orderItem)
  results: LabResult[];
}

@Entity('specimens')
export class Specimen {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  accessionNumber: string;

  @Column()
  barcode: string;

  @Column({ type: 'jsonb' })
  type: CodeableConcept;
  // blood | urine | saliva | tissue | etc.

  @ManyToOne(() => LabOrder)
  order: LabOrder;

  @ManyToOne(() => Patient)
  patient: Patient;

  @Column({ type: 'timestamptz' })
  collectedDateTime: Date;

  @Column({ type: 'timestamptz', nullable: true })
  receivedDateTime: Date;

  @ManyToOne(() => Practitioner)
  collector: Practitioner;

  @Column({ type: 'enum', enum: SpecimenStatus })
  status: SpecimenStatus;
  // available | unavailable | unsatisfactory | entered-in-error

  @Column({ type: 'jsonb', nullable: true })
  condition: CodeableConcept[]; // hemolyzed | lipemic | clotted

  @Column({ type: 'text', nullable: true })
  note: string;
}

@Entity('lab_results')
export class LabResult {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @ManyToOne(() => LabOrderItem)
  orderItem: LabOrderItem;

  @ManyToOne(() => Specimen)
  specimen: Specimen;

  @Column()
  loincCode: string;

  @Column({ type: 'enum', enum: ObservationStatus })
  status: ObservationStatus;
  // registered | preliminary | final | amended | corrected | cancelled

  @Column({ type: 'decimal', precision: 20, scale: 6, nullable: true })
  valueQuantity: number;

  @Column({ nullable: true })
  valueUnit: string;

  @Column({ type: 'text', nullable: true })
  valueString: string;

  @Column({ type: 'decimal', precision: 20, scale: 6, nullable: true })
  referenceRangeLow: number;

  @Column({ type: 'decimal', precision: 20, scale: 6, nullable: true })
  referenceRangeHigh: number;

  @Column({ type: 'text', nullable: true })
  referenceRangeText: string;

  @Column({ type: 'enum', enum: Interpretation, nullable: true })
  interpretation: Interpretation;
  // N (normal) | A (abnormal) | L (low) | H (high) | LL | HH | 

  @Column({ type: 'boolean', default: false })
  isCritical: boolean;

  @ManyToOne(() => User)
  performer: User;

  @Column({ type: 'timestamptz' })
  effectiveDateTime: Date;

  @Column({ type: 'timestamptz' })
  issued: Date;

  @Column({ type: 'text', nullable: true })
  comment: string;

  // Validation
  @ManyToOne(() => User, { nullable: true })
  validatedBy: User;

  @Column({ type: 'timestamptz', nullable: true })
  validatedAt: Date;
}
```

### 4.4 سير العمل

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Laboratory Workflow                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────────────┐  │
│  │  Order   │───▶│ Specimen │───▶│ Specimen │───▶│    Processing    │  │
│  │ Created  │    │Collection│    │ Received │    │                  │  │
│  └──────────┘    └──────────┘    └──────────┘    └────────┬─────────┘  │
│                                                           │             │
│                       ┌───────────────────────────────────┘             │
│                       ▼                                                 │
│  ┌───────────────────────────────────────────────────────────────────┐ │
│  │                    Result Entry                                    │ │
│  │  ┌─────────┐    ┌────────────┐    ┌────────────┐    ┌──────────┐ │ │
│  │  │ Manual  │    │ Instrument │    │ Reference  │    │ Critical │ │ │
│  │  │ Entry   │    │ Interface  │    │ Range Check│    │  Alert   │ │ │
│  │  └────┬────┘    └─────┬──────┘    └─────┬──────┘    └────┬─────┘ │ │
│  │       └───────────────┴─────────────────┴────────────────┘       │ │
│  └─────────────────────────────────────┬─────────────────────────────┘ │
│                                        ▼                               │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────────────────┐ │
│  │ Preliminary  │───▶│  Technical   │───▶│    Medical Validation    │ │
│  │   Results    │    │  Validation  │    │     (Pathologist)        │ │
│  └──────────────┘    └──────────────┘    └───────────┬──────────────┘ │
│                                                      │                 │
│                          ┌───────────────────────────┘                 │
│                          ▼                                             │
│  ┌──────────────────────────────────────────────────────────────────┐ │
│  │                    Final Results                                  │ │
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────┐   │ │
│  │  │   Report    │    │   Notify    │    │   Available in EMR  │   │ │
│  │  │  Generation │    │  Clinician  │    │                     │   │ │
│  │  └─────────────┘    └─────────────┘    └─────────────────────┘   │ │
│  └──────────────────────────────────────────────────────────────────┘ │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 5. وحدة الصيدلية (Pharmacy Module)

### 5.1 الوصف
إدارة الأدوية والوصفات والصرف والمخزون الصيدلاني.

### 5.2 المكونات الفرعية

```
Pharmacy Module
├── Prescription Management (إدارة الوصفات)
│   ├── E-Prescribing
│   ├── Drug-Drug Interaction Check
│   ├── Allergy Check
│   ├── Dosage Validation
│   └── Refill Management
│
├── Dispensing (الصرف)
│   ├── Queue Management
│   ├── Label Printing
│   ├── Patient Counseling
│   └── Partial Dispense
│
├── Inventory (المخزون)
│   ├── Stock Levels
│   ├── Expiry Tracking
│   ├── Reorder Points
│   └── Lot/Batch Tracking
│
├── Formulary (دليل الأدوية)
│   ├── Drug Database
│   ├── Generic Substitution
│   └── Pricing
│
└── Medication Administration (إدارة العلاج)
    ├── MAR (Medication Administration Record)
    ├── Bedside Verification
    └── Administration Documentation
```

### 5.3 كيانات البيانات

```typescript
@Entity('medications')
export class Medication {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  code: string; // Drug code

  @Column({ type: 'jsonb' })
  name: {
    ar: string;
    fr: string;
    en: string;
    tradeName?: string;
    genericName: string;
  };

  @Column({ type: 'jsonb' })
  form: CodeableConcept;
  // tablet | capsule | syrup | injection | cream | ointment | drops

  @Column({ type: 'jsonb', nullable: true })
  amount: Quantity;

  @Column({ type: 'jsonb', nullable: true })
  ingredient: MedicationIngredient[];

  @Column({ nullable: true })
  manufacturer: string;

  @Column({ type: 'jsonb', nullable: true })
  batch: {
    lotNumber: string;
    expirationDate: Date;
  };

  @Column({ default: true })
  active: boolean;
}

@Entity('medication_requests')
export class MedicationRequest {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  prescriptionNumber: string;

  @Column({ type: 'enum', enum: MedicationRequestStatus })
  status: MedicationRequestStatus;
  // active | on-hold | cancelled | completed | entered-in-error | stopped | draft

  @Column({ type: 'enum', enum: MedicationRequestIntent })
  intent: MedicationRequestIntent;
  // proposal | plan | order | original-order | reflex-order | filler-order

  @ManyToOne(() => Patient)
  patient: Patient;

  @ManyToOne(() => Encounter, { nullable: true })
  encounter: Encounter;

  @ManyToOne(() => Medication)
  medication: Medication;

  @ManyToOne(() => Practitioner)
  requester: Practitioner;

  @Column({ type: 'timestamptz' })
  authoredOn: Date;

  @Column({ type: 'jsonb' })
  dosageInstruction: Dosage[];

  @Column({ type: 'jsonb', nullable: true })
  dispenseRequest: {
    initialFill?: { quantity: Quantity; duration: Duration };
    dispenseInterval?: Duration;
    validityPeriod?: Period;
    numberOfRepeatsAllowed?: number;
    quantity?: Quantity;
    expectedSupplyDuration?: Duration;
  };

  @Column({ type: 'jsonb', nullable: true })
  substitution: {
    allowedBoolean: boolean;
    allowedCodeableConcept?: CodeableConcept;
    reason?: CodeableConcept;
  };

  @Column({ type: 'text', nullable: true })
  note: string;

  @OneToMany(() => MedicationDispense, dispense => dispense.prescription)
  dispenses: MedicationDispense[];
}

@Entity('medication_dispenses')
export class MedicationDispense {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  dispenseNumber: string;

  @Column({ type: 'enum', enum: MedicationDispenseStatus })
  status: MedicationDispenseStatus;
  // preparation | in-progress | cancelled | on-hold | completed | entered-in-error | stopped | declined

  @ManyToOne(() => MedicationRequest)
  prescription: MedicationRequest;

  @ManyToOne(() => Patient)
  patient: Patient;

  @ManyToOne(() => Medication)
  medication: Medication;

  @ManyToOne(() => User)
  performer: User;

  @ManyToOne(() => Location)
  location: Location;

  @Column({ type: 'jsonb' })
  quantity: Quantity;

  @Column({ type: 'int', nullable: true })
  daysSupply: number;

  @Column({ type: 'timestamptz' })
  whenPrepared: Date;

  @Column({ type: 'timestamptz', nullable: true })
  whenHandedOver: Date;

  @Column({ type: 'text', nullable: true })
  note: string;

  @Column({ type: 'jsonb', nullable: true })
  substitution: {
    wasSubstituted: boolean;
    type?: CodeableConcept;
    reason?: CodeableConcept[];
  };
}

interface Dosage {
  sequence?: number;
  text: string;
  timing?: Timing;
  asNeeded?: boolean;
  site?: CodeableConcept;
  route?: CodeableConcept;  // oral | IV | IM | SC | topical
  method?: CodeableConcept;
  doseAndRate?: {
    type?: CodeableConcept;
    doseQuantity?: Quantity;
    doseRange?: Range;
    rateQuantity?: Quantity;
    rateRange?: Range;
  }[];
  maxDosePerPeriod?: Ratio;
  maxDosePerAdministration?: Quantity;
}
```

---

## 6. وحدة الفوترة (Billing Module)

### 6.1 الوصف
إدارة الفواتير والمدفوعات والتأمين.

### 6.2 المكونات الفرعية

```
Billing Module
├── Charge Capture (تسجيل الرسوم)
│   ├── Service charges
│   ├── Medication charges
│   ├── Lab/Imaging charges
│   ├── Room charges
│   └── Procedure charges
│
├── Pricing (التسعير)
│   ├── Price lists
│   ├── Discounts
│   ├── Packages
│   └── Service bundles
│
├── Insurance (التأمين)
│   ├── Eligibility verification
│   ├── Pre-authorization
│   ├── Claims submission
│   └── EOB processing
│
├── Invoicing (الفوترة)
│   ├── Invoice generation
│   ├── Statement generation
│   └── Payment allocation
│
└── Payments (المدفوعات)
    ├── Cash
    ├── Card
    ├── Insurance
    └── Payment plans
```

### 6.3 كيانات البيانات

```typescript
@Entity('invoices')
export class Invoice {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  invoiceNumber: string;

  @Column({ type: 'enum', enum: InvoiceStatus })
  status: InvoiceStatus;
  // draft | issued | balanced | cancelled | entered-in-error

  @ManyToOne(() => Patient)
  patient: Patient;

  @ManyToOne(() => Encounter, { nullable: true })
  encounter: Encounter;

  @Column({ type: 'date' })
  date: Date;

  @Column({ type: 'date', nullable: true })
  dueDate: Date;

  @ManyToOne(() => Organization)
  issuer: Organization;

  @OneToMany(() => InvoiceLineItem, item => item.invoice)
  lineItems: InvoiceLineItem[];

  @Column({ type: 'decimal', precision: 15, scale: 2 })
  subtotal: number;

  @Column({ type: 'decimal', precision: 15, scale: 2, default: 0 })
  discount: number;

  @Column({ type: 'decimal', precision: 15, scale: 2, default: 0 })
  tax: number;

  @Column({ type: 'decimal', precision: 15, scale: 2 })
  totalGross: number;

  @Column({ type: 'decimal', precision: 15, scale: 2 })
  totalNet: number;

  @Column({ type: 'decimal', precision: 15, scale: 2, default: 0 })
  paidAmount: number;

  @Column({ type: 'decimal', precision: 15, scale: 2 })
  balanceDue: number;

  @Column({ default: 'DZD' })
  currency: string;

  @Column({ type: 'text', nullable: true })
  note: string;

  @OneToMany(() => Payment, payment => payment.invoice)
  payments: Payment[];

  @CreateDateColumn()
  createdAt: Date;

  @ManyToOne(() => User)
  createdBy: User;
}

@Entity('invoice_line_items')
export class InvoiceLineItem {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @ManyToOne(() => Invoice)
  invoice: Invoice;

  @Column({ type: 'int' })
  sequence: number;

  @Column({ type: 'enum', enum: ChargeType })
  chargeType: ChargeType;
  // service | medication | procedure | room | supply | other

  @Column({ type: 'jsonb' })
  code: CodeableConcept;

  @Column()
  description: string;

  @Column({ type: 'decimal', precision: 10, scale: 2 })
  quantity: number;

  @Column({ type: 'decimal', precision: 15, scale: 2 })
  unitPrice: number;

  @Column({ type: 'decimal', precision: 5, scale: 2, default: 0 })
  discountPercent: number;

  @Column({ type: 'decimal', precision: 15, scale: 2 })
  netPrice: number;

  @ManyToOne(() => Encounter, { nullable: true })
  encounter: Encounter;

  @Column({ type: 'uuid', nullable: true })
  referenceId: string; // Reference to service, medication, etc.

  @Column({ type: 'text', nullable: true })
  referenceType: string;
}

@Entity('payments')
export class Payment {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  receiptNumber: string;

  @ManyToOne(() => Invoice)
  invoice: Invoice;

  @ManyToOne(() => Patient)
  patient: Patient;

  @Column({ type: 'decimal', precision: 15, scale: 2 })
  amount: number;

  @Column({ default: 'DZD' })
  currency: string;

  @Column({ type: 'enum', enum: PaymentMethod })
  method: PaymentMethod;
  // cash | card | bank-transfer | check | insurance

  @Column({ type: 'enum', enum: PaymentStatus })
  status: PaymentStatus;
  // pending | completed | cancelled | refunded

  @Column({ type: 'timestamptz' })
  paymentDate: Date;

  @Column({ type: 'text', nullable: true })
  reference: string; // Card transaction ID, check number, etc.

  @ManyToOne(() => User)
  receivedBy: User;

  @Column({ type: 'text', nullable: true })
  note: string;

  @CreateDateColumn()
  createdAt: Date;
}

@Entity('insurance_coverages')
export class InsuranceCoverage {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ type: 'enum', enum: CoverageStatus })
  status: CoverageStatus;

  @ManyToOne(() => Patient)
  beneficiary: Patient;

  @Column()
  insurerName: string;

  @Column({ nullable: true })
  insurerCode: string;

  @Column()
  memberId: string;

  @Column({ nullable: true })
  groupId: string;

  @Column({ type: 'jsonb', nullable: true })
  type: CodeableConcept;

  @Column({ type: 'date' })
  periodStart: Date;

  @Column({ type: 'date', nullable: true })
  periodEnd: Date;

  @Column({ type: 'int', default: 1 })
  order: number; // Primary = 1, Secondary = 2, etc.

  @Column({ type: 'jsonb', nullable: true })
  class: {
    type: CodeableConcept;
    value: string;
    name?: string;
  }[];

  @Column({ type: 'jsonb', nullable: true })
  costToBeneficiary: {
    type: CodeableConcept;
    value: Quantity | Money;
  }[];
}
```

---

## 7. أولوية التنفيذ | Implementation Priority

### MVP (المرحلة الأولى - 8-10 أسابيع)
| الوحدة | الأولوية | الحالة |
|--------|----------|--------|
| Auth & Users | P0 | 🔴 Not Started |
| Patient Registration | P0 | 🔴 Not Started |
| Patient Search | P0 | 🔴 Not Started |
| Appointment Booking | P0 | 🔴 Not Started |
| Encounter (OPD) | P0 | 🔴 Not Started |
| Basic Billing | P0 | 🔴 Not Started |
| Basic Reporting | P1 | 🔴 Not Started |

### Phase 2 (المرحلة الثانية)
| الوحدة | الأولوية | الحالة |
|--------|----------|--------|
| Laboratory (LIS) | P1 | 🔴 Not Started |
| Pharmacy | P1 | 🔴 Not Started |
| Inpatient (IPD) | P1 | 🔴 Not Started |
| Insurance | P2 | 🔴 Not Started |

### Phase 3 (المرحلة الثالثة)
| الوحدة | الأولوية | الحالة |
|--------|----------|--------|
| Radiology (RIS) | P2 | 🔴 Not Started |
| Emergency | P2 | 🔴 Not Started |
| Operating Room | P2 | 🔴 Not Started |
| Blood Bank | P3 | 🔴 Not Started |

---

> **Legend**: 🔴 Not Started | 🟡 In Progress | 🟢 Completed
