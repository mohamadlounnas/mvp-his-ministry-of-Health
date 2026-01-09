# System Architecture
# هيكلة النظام

## 1. نظرة عامة على المعمارية | Architecture Overview

```
┌────────────────────────────────────────────────────────────────────────────┐
│                              GHIS Architecture                             │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                            │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                        CLIENT LAYER                                 │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌────────────┐  │   │
│  │  │ Flutter Web │  │ Flutter iOS │  │Flutter Droid│  │Flutter Desk│  │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └────────────┘  │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                    │                                       │
│                                    ▼                                       │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                      API GATEWAY (Nginx/Traefik)                    │   │
│  │                    Rate Limiting · SSL · Load Balancing             │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                    │                                       │
│              ┌─────────────────────┼─────────────────────┐                 │
│              ▼                     ▼                     ▼                 │
│   ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐          │
│   │   Auth Service  │   │   Core Service  │   │  FHIR Service   │          │
│   │    (NestJS)     │   │    (NestJS)     │   │    (NestJS)     │          │
│   │                 │   │                 │   │                 │          │
│   │ • JWT/OAuth2    │   │ • Patient       │   │ • FHIR API      │          │
│   │ • RBAC          │   │ • Encounter     │   │ • Transformers  │          │
│   │ • MFA           │   │ • Appointment   │   │ • Bundles       │          │
│   │ • Session       │   │ • Billing       │   │ • Search        │          │
│   └────────┬────────┘   └────────┬────────┘   └────────┬────────┘          │
│           │                     │                      │                   │
│           └─────────────────────┼──────────────────────┘                   │
│                                 ▼                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                       MESSAGE QUEUE (RabbitMQ)                      │   │
│  │                Events · Tasks · Notifications · Audit               │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                     │                                      │
│            ┌────────────────────────┼────────────────────────┐             │
│            ▼                        ▼                        ▼             │
│  ┌───────────────────┐   ┌─────────────────────┐   ┌───────────────────┐   │
│  │ Notification      │   │   Worker Service    │   │   Report Service  │   │
│  │   Service         │   │                     │   │                   │   │
│  │                   │   │ • Background Jobs   │   │ • PDF Generation  │   │
│  │ • ntfy            │   │ • Data Sync         │   │ • Excel Export    │   │
│  │ • WebSocket       │   │ • Cleanup Tasks     │   │ • Analytics       │   │
│  │ • Email           │   │ • Reminders         │   │ • Dashboards      │   │
│  └───────────────────┘   └─────────────────────┘   └───────────────────┘   │
│                                                                            │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         DATA LAYER                                  │   │
│  │                                                                     │   │
│  │ ┌─────────────┐  ┌─────────────┐  ┌───────────────┐  ┌────────────┐ │   │
│  │ │ PostgreSQL  │  │    Redis    │  │ Elasticsearch │  │   MinIO    │ │   │
│  │ │             │  │             │  │               │  │            │ │   │
│  │ │ • Primary   │  │ • Cache     │  │ • Search      │  │ • Files    │ │   │
│  │ │   Database  │  │ • Sessions  │  │ • Indexing    │  │ • Documents│ │   │
│  │ │ • JSONB     │  │ • Pub/Sub   │  │ • Logs        │  │ • Images   │ │   │
│  │ └─────────────┘  └─────────────┘  └───────────────┘  └────────────┘ │   │
│  │                                                                     │   │
│  │ ┌─────────────────────────────────────────────────────────────────┐ │   │
│  │ │                    Orthanc PACS Server                          │ │   │
│  │ │              DICOM Storage · WADO-RS · Query/Retrieve           │ │   │
│  │ └─────────────────────────────────────────────────────────────────┘ │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. الوحدات التفصيلية | Module Architecture

### 2.1 البنية الطبقية للخدمات (Service Layer Architecture)

```
┌─────────────────────────────────────────────────────────────────┐
│                     NestJS Service Structure                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌──────────────────────────────────────────────────────────┐  │
│   │                    Controller Layer                      │  │
│   │   • REST Endpoints       • Request Validation            │  │
│   │   • Response Formatting  • Error Handling                │  │
│   └────────────────────────────┬─────────────────────────────┘  │
│                                │                                │
│   ┌────────────────────────────▼─────────────────────────────┐  │
│   │                    Service Layer                         │  │
│   │   • Business Logic       • Transaction Management        │  │
│   │   • Validation Rules     • Event Emission                │  │
│   └────────────────────────────┬─────────────────────────────┘  │
│                                │                                │
│   ┌────────────────────────────▼─────────────────────────────┐  │
│   │                   Repository Layer                       │  │
│   │   • Data Access          • Query Building                │  │
│   │   • Entity Mapping       • Caching Strategy              │  │
│   └────────────────────────────┬─────────────────────────────┘  │
│                                │                                │
│   ┌────────────────────────────▼─────────────────────────────┐  │
│   │                    Entity Layer                          │  │
│   │   • TypeORM Entities     • FHIR Resource Mapping         │  │
│   │   • Validation Rules     • Relationships                 │  │
│   └──────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 هيكل الوحدات (Module Structure)

```
src/
├── main.ts                          # Application entry point
├── app.module.ts                    # Root module
│
├── common/                          # Shared utilities
│   ├── decorators/                  # Custom decorators
│   ├── filters/                     # Exception filters
│   ├── guards/                      # Auth guards
│   ├── interceptors/                # Request interceptors
│   ├── pipes/                       # Validation pipes
│   ├── dto/                         # Shared DTOs
│   ├── interfaces/                  # Shared interfaces
│   └── utils/                       # Utility functions
│
├── config/                          # Configuration
│   ├── database.config.ts
│   ├── redis.config.ts
│   ├── minio.config.ts
│   └── app.config.ts
│
├── modules/
│   ├── auth/                        # Authentication module
│   │   ├── auth.module.ts
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   ├── strategies/
│   │   │   ├── jwt.strategy.ts
│   │   │   └── local.strategy.ts
│   │   ├── guards/
│   │   │   ├── jwt-auth.guard.ts
│   │   │   └── roles.guard.ts
│   │   └── dto/
│   │       ├── login.dto.ts
│   │       └── register.dto.ts
│   │
│   ├── patient/                     # Patient module
│   │   ├── patient.module.ts
│   │   ├── patient.controller.ts
│   │   ├── patient.service.ts
│   │   ├── patient.repository.ts
│   │   ├── entities/
│   │   │   ├── patient.entity.ts
│   │   │   ├── patient-contact.entity.ts
│   │   │   └── patient-identifier.entity.ts
│   │   ├── dto/
│   │   │   ├── create-patient.dto.ts
│   │   │   ├── update-patient.dto.ts
│   │   │   └── search-patient.dto.ts
│   │   └── fhir/
│   │       ├── patient.transformer.ts
│   │       └── patient.validator.ts
│   │
│   ├── encounter/                   # Encounter/Visit module
│   │   ├── encounter.module.ts
│   │   ├── encounter.controller.ts
│   │   ├── encounter.service.ts
│   │   ├── entities/
│   │   │   ├── encounter.entity.ts
│   │   │   ├── encounter-diagnosis.entity.ts
│   │   │   └── encounter-participant.entity.ts
│   │   └── dto/
│   │
│   ├── appointment/                 # Appointment module
│   │   ├── appointment.module.ts
│   │   ├── appointment.controller.ts
│   │   ├── appointment.service.ts
│   │   ├── appointment.scheduler.ts
│   │   └── entities/
│   │       └── appointment.entity.ts
│   │
│   ├── clinical/                    # Clinical module
│   │   ├── clinical.module.ts
│   │   ├── condition/               # Conditions/Diagnoses
│   │   ├── observation/             # Observations/Vital Signs
│   │   ├── procedure/               # Procedures
│   │   ├── allergy/                 # Allergies
│   │   └── care-plan/               # Care Plans
│   │
│   ├── medication/                  # Medication module
│   │   ├── medication.module.ts
│   │   ├── medication.controller.ts
│   │   ├── medication.service.ts
│   │   ├── prescription/
│   │   │   ├── prescription.controller.ts
│   │   │   ├── prescription.service.ts
│   │   │   └── entities/
│   │   └── dispense/
│   │
│   ├── laboratory/                  # Laboratory module (LIS)
│   │   ├── laboratory.module.ts
│   │   ├── lab-order/
│   │   ├── lab-result/
│   │   ├── specimen/
│   │   └── loinc/                   # LOINC integration
│   │
│   ├── radiology/                   # Radiology module (RIS)
│   │   ├── radiology.module.ts
│   │   ├── imaging-order/
│   │   ├── imaging-study/
│   │   └── pacs/                    # PACS/Orthanc integration
│   │
│   ├── billing/                     # Billing module
│   │   ├── billing.module.ts
│   │   ├── invoice/
│   │   ├── payment/
│   │   ├── insurance/
│   │   └── pricing/
│   │
│   ├── inventory/                   # Inventory module
│   │   ├── inventory.module.ts
│   │   ├── stock/
│   │   ├── purchase-order/
│   │   └── supplier/
│   │
│   ├── organization/                # Organization module
│   │   ├── organization.module.ts
│   │   ├── hospital/
│   │   ├── department/
│   │   └── location/
│   │
│   ├── practitioner/                # Practitioner module
│   │   ├── practitioner.module.ts
│   │   ├── entities/
│   │   └── schedule/
│   │
│   ├── coding/                      # Medical coding module
│   │   ├── coding.module.ts
│   │   ├── icd11/                   # ICD-11 integration
│   │   │   ├── icd11.service.ts
│   │   │   ├── icd11.controller.ts
│   │   │   └── entities/
│   │   ├── snomed/                  # SNOMED CT (future)
│   │   └── loinc/                   # LOINC codes
│   │
│   ├── notification/                # Notification module
│   │   ├── notification.module.ts
│   │   ├── notification.service.ts
│   │   ├── channels/
│   │   │   ├── ntfy.channel.ts
│   │   │   ├── websocket.channel.ts
│   │   │   └── email.channel.ts
│   │   └── templates/
│   │
│   ├── audit/                       # Audit module
│   │   ├── audit.module.ts
│   │   ├── audit.service.ts
│   │   ├── audit.interceptor.ts
│   │   └── entities/
│   │       └── audit-log.entity.ts
│   │
│   ├── report/                      # Report module
│   │   ├── report.module.ts
│   │   ├── generators/
│   │   └── templates/
│   │
│   └── fhir/                        # FHIR API module
│       ├── fhir.module.ts
│       ├── fhir.controller.ts
│       ├── resources/
│       ├── operations/
│       └── search/
│
├── database/
│   ├── migrations/
│   ├── seeds/
│   └── factories/
│
└── jobs/                            # Background jobs
    ├── reminder.job.ts
    ├── cleanup.job.ts
    └── sync.job.ts
```

---

## 3. هيكلة قاعدة البيانات | Database Architecture

### 3.1 Schemas

```sql
-- Core schemas
CREATE SCHEMA core;       -- Core entities (patients, practitioners, organizations)
CREATE SCHEMA clinical;   -- Clinical data (encounters, conditions, observations)
CREATE SCHEMA medication; -- Medication-related data
CREATE SCHEMA lab;        -- Laboratory data
CREATE SCHEMA imaging;    -- Radiology/imaging data
CREATE SCHEMA billing;    -- Financial data
CREATE SCHEMA inventory;  -- Inventory/stock data
CREATE SCHEMA coding;     -- Medical coding (ICD-11, LOINC, etc.)
CREATE SCHEMA audit;      -- Audit logs
CREATE SCHEMA config;     -- System configuration
```

### 3.2 المخطط العلائقي الرئيسي (Core ERD)

```
                                    ┌─────────────────┐
                                    │  organizations  │
                                    ├─────────────────┤
                                    │ id              │
                                    │ name            │
                                    │ type            │
                                    │ parent_id       │◄───┐
                                    │ address         │    │
                                    │ active          │    │
                                    └────────┬────────┘    │
                                             │             │
                                             │ parent      │
                    ┌────────────────────────┴─────────────┘
                    │
                    ▼
┌─────────────────────┐         ┌─────────────────────┐
│     locations       │         │   practitioners     │
├─────────────────────┤         ├─────────────────────┤
│ id                  │         │ id                  │
│ name                │         │ identifier          │
│ type                │         │ name                │
│ organization_id     │◄────────│ locals              │
│ parent_id           │         │ gender              │
│ status              │         │ specialty           │
│ physical_type       │         │ organization_id     │
│ address             │         │ active              │
└─────────────────────┘         └──────────┬──────────┘
           │                               │
           │                               │
           ▼                               │
┌─────────────────────┐                    │
│    appointments     │                    │
├─────────────────────┤                    │
│ id                  │                    │
│ patient_id          │◄───────────────────┤
│ practitioner_id     │◄───────────────────┘
│ location_id         │◄──────┘
│ start_time          │
│ end_time            │
│ status              │
│ reason              │
└─────────┬───────────┘
          │
          │
          ▼
┌─────────────────────┐         ┌─────────────────────┐
│      patients       │         │     encounters      │
├─────────────────────┤         ├─────────────────────┤
│ id                  │◄────────│ patient_id          │
│ mrn                 │         │ id                  │
│ national_id         │         │ class               │
│ name                │         │ type                │
│ locals              │         │ status              │
│ birth_date          │         │ priority            │
│ gender              │         │ period_start        │
│ phone               │         │ period_end          │
│ address             │         │ practitioner_id     │◄──
│ blood_type          │         │ location_id         │
│ emergency_contact   │         │ appointment_id      │
│ active              │         │ service_type        │
└─────────┬───────────┘         └──────────┬──────────┘
          │                                │
          │                                │
          │         ┌──────────────────────┴──────────────┐
          │         │                                     │
          │         ▼                                     ▼
          │  ┌─────────────────────┐            ┌─────────────────────┐
          │  │    conditions       │            │   observations      │
          │  ├─────────────────────┤            ├─────────────────────┤
          │  │ id                  │            │ id                  │
          │  │ patient_id          │◄─────┐     │ patient_id          │◄─
          │  │ encounter_id        │      │     │ encounter_id        │
          │  │ icd11_code          │      │     │ loinc_code          │
          │  │ clinical_status     │      │     │ category            │
          │  │ verification_status │      │     │ value_quantity      │
          │  │ severity            │      │     │ value_string        │
          │  │ onset_date          │      │     │ effective_date      │
          │  │ recorded_date       │      │     │ interpretation      │
          │  │ recorder_id         │      │     │ performer_id        │
          │  └─────────────────────┘      │     └─────────────────────┘
          │                               │
          │         ┌─────────────────────┤
          │         │                     │
          ▼         ▼                     │
┌─────────────────────────┐               │
│  medication_requests    │               │
├─────────────────────────┤               │
│ id                      │               │
│ patient_id              │◄──────────────┤
│ encounter_id            │               │
│ medication_id           │◄───┐          │
│ status                  │    │          │
│ intent                  │    │          │
│ dosage_instruction      │    │          │
│ quantity                │    │          │
│ authored_on             │    │          │
│ requester_id            │    │          │
└─────────────────────────┘    │          │
                               │          │
                    ┌──────────┘          │
                    │                     │
                    ▼                     │
          ┌─────────────────────┐         │
          │    medications      │         │
          ├─────────────────────┤         │
          │ id                  │         │
          │ code                │         │
          │ name                │         │
          │ locals              │         │
          │ form                │         │
          │ strength            │         │
          │ manufacturer        │         │
          └─────────────────────┘         │
                                          │
          ┌───────────────────────────────┘
          │
          ▼
┌─────────────────────────┐
│       invoices          │
├─────────────────────────┤
│ id                      │
│ patient_id              │◄─────────
│ encounter_id            │
│ invoice_number          │
│ status                  │
│ total_amount            │
│ paid_amount             │
│ issued_date             │
│ due_date                │
└─────────────────────────┘
```

---

## 4. مسارات البيانات | Data Flow

### 4.1 مسار تسجيل مريض جديد

```
┌─────────┐    ┌─────────────┐    ┌──────────────┐    ┌────────────┐
│ Flutter │───▶│ API Gateway │───▶│ Auth Service │───▶│ JWT Valid? │
│  App    │    │   (Nginx)   │    │              │    │            │
└─────────┘    └─────────────┘    └──────────────┘    └─────┬──────┘
                                                            │
                    ┌───────────────────────────────────────┘
                    │                         NO ───▶ 401 Unauthorized
                    ▼ YES
          ┌─────────────────┐    ┌─────────────────┐
          │ Patient Service │───▶│  Validation     │
          │   Controller    │    │  (DTO/Pipes)    │
          └────────┬────────┘    └────────┬────────┘
                   │                      │
                   │                      ▼ INVALID ───▶ 400 Bad Request
                   │                      │
                   ▼ VALID                │
          ┌─────────────────┐    ◄────────┘
          │ Patient Service │
          │  (Business)     │
          └────────┬────────┘
                   │
          ┌────────┴────────┐
          │                 │
          ▼                 ▼
┌─────────────────┐  ┌─────────────────┐
│ Check Duplicate │  │ Generate MRN    │
│ (National ID)   │  │                 │
└────────┬────────┘  └────────┬────────┘
         │                    │
         ▼                    │
    EXISTS? ─── YES ──▶ 409 Conflict
         │
         │ NO
         ▼
┌─────────────────┐    ┌─────────────────┐
│   Repository    │───▶│   PostgreSQL    │
│   (TypeORM)     │    │   INSERT        │
└────────┬────────┘    └────────┬────────┘
         │                      │
         │                      ▼
         │             ┌─────────────────┐
         │             │   Audit Log     │
         │             │   (Event)       │
         │             └─────────────────┘
         ▼
┌─────────────────┐    ┌─────────────────┐
│ Cache Update    │    │   RabbitMQ      │
│ (Redis)         │    │ patient.created │
└─────────────────┘    └────────┬────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │ Elasticsearch   │
                       │ Index Update    │
                       └─────────────────┘
```

### 4.2 مسار حجز موعد

```
┌──────────────────────────────────────────────────────────────────┐
│                      Appointment Booking Flow                    │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Check Availability                                           │
│     ┌─────────┐                      ┌──────────────────┐        │
│     │ Client  │ ── GET /slots ────▶  │ Schedule Service │        │
│     └─────────┘                      └────────┬─────────┘        │
│                                               │                  │
│                      ┌────────────────────────┘                  │
│                      ▼                                           │
│            ┌─────────────────┐                                   │
│            │ Query Available │                                   │
│            │ Slots for Date  │                                   │
│            └────────┬────────┘                                   │
│                     │                                            │
│                     ▼                                            │
│            ┌─────────────────┐                                   │
│            │ Return Slots    │                                   │
│            │ with Status     │                                   │
│            └─────────────────┘                                   │
│                                                                  │
│  2. Book Appointment                                             │
│     ┌─────────┐                      ┌─────────────────┐         │
│     │ Client  │ ── POST /appt ────▶  │ Appointment Svc │         │
│     └─────────┘                      └────────┬────────┘         │
│                                               │                  │
│            ┌──────────────────────────────────┘                  │
│            ▼                                                     │
│   ┌─────────────────┐                                            │
│   │ Transaction     │                                            │
│   │ ┌─────────────┐ │    ┌─────────────┐                         │
│   │ │ Lock Slot   │─┼───▶│ Redis Lock  │                         │
│   │ └─────────────┘ │    └─────────────┘                         │
│   │ ┌─────────────┐ │                                            │
│   │ │ Validate    │ │                                            │
│   │ │ Patient     │ │                                            │
│   │ └─────────────┘ │                                            │
│   │ ┌─────────────┐ │    ┌─────────────┐                         │
│   │ │ Create Appt │─┼───▶│ PostgreSQL  │                         │
│   │ └─────────────┘ │    └─────────────┘                         │
│   │ ┌─────────────┐ │    ┌─────────────┐                         │
│   │ │ Update Slot │─┼───▶│ Mark Booked │                         │
│   │ └─────────────┘ │    └─────────────┘                         │
│   └─────────────────┘                                            │
│            │                                                     │
│            ▼                                                     │
│   ┌─────────────────┐    ┌─────────────────┐                     │
│   │ Emit Event      │───▶│ RabbitMQ        │                     │
│   │ appointment     │    │ appointment     │                     │
│   │ .booked         │    │ .booked         │                     │
│   └─────────────────┘    └────────┬────────┘                     │
│                                   │                              │
│            ┌──────────────────────┼──────────────────────┐       │
│            ▼                      ▼                      ▼       │
│   ┌─────────────────┐    ┌─────────────────┐    ┌────────────┐   │
│   │ Send            │    │ Schedule        │    │ Audit      │   │
│   │ Confirmation    │    │ Reminder Job    │    │ Log        │   │
│   │ (ntfy/SMS)      │    │ (24h before)    │    │            │   │
│   └─────────────────┘    └─────────────────┘    └────────────┘   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 5. الأمان | Security Architecture

### 5.1 طبقات الأمان

```
┌─────────────────────────────────────────────────────────────────┐
│                      Security Layers                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ Layer 1: Network Security                                 │  │
│  │ • Firewall Rules                                          │  │
│  │ • VPN for admin access                                    │  │
│  │ • DDoS Protection                                         │  │
│  │ • IP Whitelisting (for external APIs)                     │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              │                                  │
│                              ▼                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ Layer 2: Transport Security                               │  │
│  │ • TLS 1.3 (all communications)                            │  │
│  │ • Certificate pinning (mobile apps)                       │  │
│  │ • HSTS headers                                            │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              │                                  │
│                              ▼                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ Layer 3: Authentication                                   │  │
│  │ • JWT with RS256 signing                                  │  │
│  │ • Refresh token rotation                                  │  │
│  │ • MFA (TOTP) for sensitive roles                          │  │
│  │ • Session management (Redis)                              │  │
│  │ • Brute-force protection                                  │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              │                                  │
│                              ▼                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ Layer 4: Authorization (RBAC)                             │  │
│  │ • Role-based access control                               │  │
│  │ • Resource-level permissions                              │  │
│  │ • Organization-scoped access                              │  │
│  │ • Patient consent management                              │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              │                                  │
│                              ▼                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ Layer 5: Data Security                                    │  │
│  │ • Encryption at rest (AES-256)                            │  │
│  │ • Field-level encryption (sensitive data)                 │  │
│  │ • Data masking (logs, exports)                            │  │
│  │ • Secure deletion                                         │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              │                                  │
│                              ▼                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ Layer 6: Audit & Monitoring                               │  │
│  │ • Complete audit trail                                    │  │
│  │ • Anomaly detection                                       │  │
│  │ • Real-time alerts                                        │  │
│  │ • Immutable logs                                          │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 نموذج الأدوار والصلاحيات (RBAC)

```yaml
Roles:
  SUPER_ADMIN:
    description: System administrator
    permissions: ["*"]
    
  HOSPITAL_ADMIN:
    description: Hospital administrator
    scope: organization
    permissions:
      - users:manage
      - departments:manage
      - reports:view
      - settings:manage
      
  DOCTOR:
    description: Physician
    scope: assigned_patients
    permissions:
      - patients:view
      - patients:update
      - encounters:create
      - encounters:update
      - prescriptions:create
      - prescriptions:view
      - lab_orders:create
      - lab_results:view
      - imaging_orders:create
      - imaging_results:view
      
  NURSE:
    description: Nursing staff
    scope: department
    permissions:
      - patients:view
      - encounters:view
      - observations:create
      - observations:view
      - medications:administer
      
  LAB_TECHNICIAN:
    description: Laboratory technician
    scope: laboratory
    permissions:
      - lab_orders:view
      - lab_results:create
      - lab_results:update
      - specimens:manage
      
  RECEPTIONIST:
    description: Front desk staff
    scope: location
    permissions:
      - patients:create
      - patients:view
      - appointments:manage
      - billing:create
      - billing:view
      
  PHARMACIST:
    description: Pharmacy staff
    scope: pharmacy
    permissions:
      - prescriptions:view
      - dispensations:create
      - inventory:view
      - inventory:update
      
  BILLING_CLERK:
    description: Billing staff
    scope: billing
    permissions:
      - billing:manage
      - payments:manage
      - insurance:view
```

---

## 6. النشر والبنية التحتية | Deployment

### 6.1 بيئات العمل

```yaml
Environments:
  Development:
    purpose: Local development
    database: Local PostgreSQL
    services: Docker Compose
    
  Staging:
    purpose: Testing & QA
    database: Replicated PostgreSQL
    services: Kubernetes (single node)
    
  Production:
    purpose: Live system
    database: HA PostgreSQL cluster
    services: Kubernetes (HA cluster)
    backup: Daily + continuous WAL
```

### 6.2 Docker Compose (Development)

```yaml
version: '3.9'

services:
  api:
    build: ./backend
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://ghis:ghis@postgres:5432/ghis
      - REDIS_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis
      
  postgres:
    image: postgres:16
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=ghis
      - POSTGRES_USER=ghis
      - POSTGRES_PASSWORD=ghis
      
  redis:
    image: redis:7-alpine
    volumes:
      - redisdata:/data
      
  minio:
    image: minio/minio
    command: server /data --console-address ":9001"
    ports:
      - "9000:9000"
      - "9001:9001"
    volumes:
      - miniodata:/data
      
  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"
      - "15672:15672"
      
  elasticsearch:
    image: elasticsearch:8.11.1
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    volumes:
      - esdata:/usr/share/elasticsearch/data
      
  orthanc:
    image: jodogne/orthanc-plugins
    ports:
      - "8042:8042"
      - "4242:4242"
    volumes:
      - orthancdata:/var/lib/orthanc/db

volumes:
  pgdata:
  redisdata:
  miniodata:
  esdata:
  orthancdata:
```

---

> **ملاحظة**: هذه الهيكلة قابلة للتعديل بناءً على متطلبات الأداء والتوسع.
