# GHIS - نظام معلومات المستشفيات الحكومية الجزائرية
# Government Hospital Information System

## 1. رؤية المشروع | Project Vision

### 1.1 الهدف الاستراتيجي
بناء نظام معلومات صحي وطني شامل يربط جميع المستشفيات الجزائرية بمنصة موحدة تتوافق مع المعايير الدولية للرعاية الصحية.

### 1.2 Strategic Goal
Build a comprehensive National Health Information System connecting all Algerian hospitals through a unified platform compliant with international healthcare standards.

---

## 2. نطاق المشروع | Project Scope

### 2.1 ما يشمله النظام (In Scope)

| الوحدة | Module | الوصف |
|--------|--------|-------|
| **ADT** | Admission, Discharge, Transfer | إدارة دخول وخروج ونقل المرضى |
| **EMR** | Electronic Medical Records | السجلات الطبية الإلكترونية |
| **OPD** | Outpatient Department | إدارة العيادات الخارجية |
| **IPD** | Inpatient Department | إدارة الأقسام الداخلية |
| **ER** | Emergency Room | قسم الطوارئ |
| **OR** | Operating Room | غرف العمليات |
| **LIS** | Laboratory Information System | نظام معلومات المختبر |
| **RIS/PACS** | Radiology/Imaging | الأشعة والتصوير الطبي |
| **PHA** | Pharmacy | إدارة الصيدلية |
| **BLD** | Blood Bank | بنك الدم |
| **BIL** | Billing & Finance | الفوترة والمالية |
| **INV** | Inventory Management | إدارة المخزون |
| **HR** | Human Resources | الموارد البشرية |
| **REP** | Reporting & Analytics | التقارير والتحليلات |

### 2.2 ما لا يشمله (Out of Scope - MVP)
- نظام الإسعاف (EMS) - مرحلة لاحقة
- التطبيب عن بُعد (Telemedicine) - مرحلة لاحقة
- الذكاء الاصطناعي للتشخيص - مرحلة لاحقة

---

## 3. المعايير المرجعية | Reference Standards

### 3.1 التصنيفات الطبية (Medical Classifications)

#### ICD-11 (إلزامي - Mandatory)
> **التصنيف الدولي للأمراض - الإصدار الحادي عشر**
> - معتمد من منظمة الصحة العالمية منذ يناير 2022
> - إلزامي لجميع الدول الأعضاء (132 دولة)
> - متوفر بالعربية والفرنسية والإنجليزية
> - يدعم APIs الرقمية

```
استخدامات ICD-11:
├── تشفير التشخيصات (Diagnosis Coding)
├── تشفير الإجراءات (Procedure Coding)  
├── تصنيف الوفيات (Mortality Classification)
├── إحصاءات الأمراض (Morbidity Statistics)
└── الفوترة والتأمين (Billing & Insurance)
```

#### SNOMED CT (موصى به - Recommended)
> **المصطلحات السريرية المنهجية**
> - أكثر من 350,000 مفهوم طبي
> - يدعم الترميز المركب (Post-coordination)
> - متوفر بالعربية والإنجليزية والإسبانية والفرنسية

```
استخدامات SNOMED CT:
├── توثيق سريري مفصل
├── دعم القرار السريري
├── البحث الطبي
└── تبادل البيانات
```

### 3.2 معايير التبادل (Interoperability Standards)

#### HL7 FHIR R5 (إلزامي - Mandatory)
> **موارد التشغيل البيني السريع للرعاية الصحية**

```
وحدات FHIR المطبقة:
├── Foundation
│   ├── Resource
│   ├── DomainResource
│   └── Bundle
├── Administration  
│   ├── Patient
│   ├── Practitioner
│   ├── Organization
│   ├── Location
│   └── HealthcareService
├── Clinical
│   ├── Condition
│   ├── Procedure
│   ├── CarePlan
│   ├── ClinicalImpression
│   └── AllergyIntolerance
├── Diagnostics
│   ├── Observation
│   ├── DiagnosticReport
│   ├── Specimen
│   └── ImagingStudy
├── Medications
│   ├── Medication
│   ├── MedicationRequest
│   ├── MedicationDispense
│   └── Immunization
├── Workflow
│   ├── Appointment
│   ├── Schedule
│   ├── Task
│   └── Encounter
└── Financial
    ├── Claim
    ├── Coverage
    └── Invoice
```

#### DICOM (للتصوير - For Imaging)
> **معيار الاتصالات والتصوير الرقمي في الطب**
> - إلزامي لجميع أجهزة التصوير
> - التكامل مع PACS

#### LOINC (للمختبر - For Laboratory)
> **أكواد تحديد الملاحظات المنطقية**
> - ترميز موحد للفحوصات المخبرية
> - أكثر من 90,000 كود

---

## 4. القيود والمتطلبات | Constraints & Requirements

### 4.1 السيادة على البيانات (Data Sovereignty)
```
⚠️ متطلب حرج - Critical Requirement

✓ جميع البيانات تُخزن داخل الجزائر فقط
✓ لا استخدام لخدمات سحابية خارجية
✓ نظام إشعارات مستضاف ذاتياً (Self-hosted Push)
✓ تخزين ملفات مستضاف ذاتياً (Self-hosted Storage)
```

### 4.2 اللغات المدعومة (Supported Languages)
| اللغة | الكود | الاتجاه | الأولوية |
|-------|-------|---------|---------|
| العربية | ar | RTL | Primary |
| الفرنسية | fr | LTR | Primary |
| الإنجليزية | en | LTR | Secondary |
| الإسبانية | es | LTR | Future |

### 4.3 التكنولوجيا المختارة (Technology Stack)
```yaml
Frontend:
  framework: Flutter
  platforms: [Web, Android, iOS, Desktop]
  state_management: Riverpod/Bloc
  
Backend:
  framework: NestJS
  language: TypeScript
  runtime: Node.js 20+
  
Database:
  primary: PostgreSQL 16+
  cache: Redis
  search: Elasticsearch/OpenSearch
  
Infrastructure:
  push_notifications: ntfy (self-hosted)
  file_storage: MinIO (S3-compatible)
  media_server: Orthanc (DICOM/PACS)
  message_queue: RabbitMQ
```

---

## 5. أهداف MVP | MVP Goals

### 5.1 المدة الزمنية: ~8-10 أسابيع

### 5.2 نطاق MVP
```
MVP Phase 1 (الأولوية القصوى):
├── إدارة المرضى (Patient Management)
│   ├── تسجيل المرضى الجدد
│   ├── البحث عن المرضى
│   └── عرض السجل الطبي الأساسي
├── المواعيد (Appointments)
│   ├── جدولة المواعيد
│   ├── قائمة انتظار اليوم
│   └── إشعارات التذكير
├── الزيارات (Encounters)
│   ├── تسجيل الزيارة
│   ├── الأعراض والتشخيص (ICD-11)
│   └── الوصفات الطبية
└── الفوترة الأساسية (Basic Billing)
    ├── إنشاء الفاتورة
    └── تسجيل الدفع
```

### 5.3 معايير النجاح
- [ ] تسجيل 100 مريض بدون أخطاء
- [ ] حجز 50 موعد وإكمالها
- [ ] إنشاء 50 فاتورة
- [ ] وقت استجابة < 2 ثانية
- [ ] دعم RTL بشكل كامل
- [ ] عمل النظام لمدة 72 ساعة متواصلة

---

## 6. الفريق | Team

### 6.1 حجم الفريق
- إجمالي: ~25 شخص
- قادة فرق: 5
- مطورون مبتدئون: ~20

### 6.2 هيكل الفرق
```
Team Alpha (Core Platform)     - 5 أشخاص
├── Database & Backend Core
└── Authentication & Security

Team Beta (Patient Module)     - 5 أشخاص
├── Patient Registration
└── Medical Records

Team Gamma (Clinical Module)   - 5 أشخاص
├── Appointments
├── Encounters
└── Prescriptions

Team Delta (Financial Module)  - 5 أشخاص
├── Billing
└── Inventory

Team Epsilon (Frontend)        - 5 أشخاص
├── Flutter Web
├── Flutter Mobile
└── UI/UX
```

---

## 7. المراجع | References

### 7.1 أنظمة مرجعية مفتوحة المصدر
| النظام | الوصف | الرابط |
|--------|-------|--------|
| OpenMRS | نظام سجلات طبية مفتوح | openmrs.org |
| GNU Health | نظام HIS شامل - GNU | gnuhealth.org |
| Bahmni | منصة EMR/HIS | bahmni.org |

### 7.2 وثائق المعايير
- ICD-11: https://icd.who.int/
- HL7 FHIR: https://hl7.org/fhir/
- SNOMED CT: https://www.snomed.org/
- LOINC: https://loinc.org/
- DICOM: https://www.dicomstandard.org/

---

> **ملاحظة**: هذه الوثيقة قيد التطوير وستُحدث باستمرار.
> 
> **Note**: This document is under development and will be continuously updated.
