# دليل الأسلوب - Effective Dart Style
# `Effective Dart` (Style Guide)

## الهدف | Purpose
تقديم ملخص عملي لقواعد أسلوب Dart المبينة في https://dart.dev/effective-dart/style، مع أمثلة قابلة للتطبيق في مشروع GHIS (Flutter + Dart).

---

## 1. التنسيق وFormatter 🔧
- استخدم `dart format` (dartfmt) دائماً قبل رفع الكود.
- إعداد CI ليطبق `dart format --fix` على الكود وتبلغ عن الانحرافات.

**نقطة سريعة:** اتبع قواعد الفاصلة، المسافات، والفواصل البرمجية لسهولة المراجعة.

---

## 2. التسمية (Naming) 🏷️
- Types / Classes / Enums: **UpperCamelCase** (مثال: `PatientRepository`).
- Methods / variables / fields / params: **lowerCamelCase** (`patientName`, `calculateAge`).
- Constants: **lowerCamelCase** for const values in libraries; UPPER_SNAKE_CASE only for environment-style constants.
- Files: use **snake_case** (`patient_service.dart`).

**مثال (Good vs Bad):**
```dart
// Good
class PatientRecord {}
final int patientAge;

// Bad
class patientrecord {}
int patient_age;
```

---

## 3. use final/const and immutability ✅
- استخدم `final` إذا كانت القيمة تُعيَّن مرة واحدة.
- استخدم `const` للثوابت المعروفة وقت الترجمة.
- Prefer immutable data structures for models where possible.

```dart
final List<String> tags = const ['urgent', 'followup'];
const defaultTimeout = Duration(seconds: 30);
```

---

## 4. Null Safety و Types 🛡️
- اعمل مع null-safety بصرامة: اجعل الحقول nullable فقط عندما يكون ذلك ضرورياً (`String? middleName`).
- استغل type inference لكن حدد الأنواع علناً على الحقول العامة في الـ API و DTOs.

---

## 5. Documentation & Comments 🧾
- استخدم `///` للتوثيق العام (public APIs). صف ماذا يفعل الدالة وليس كيف.
- ضع مثالاً قصيراً داخل التعليق إن أمكن.

```dart
/// Returns the patient's full display name using preferred language.
String displayName(Patient p) => p.name;
```

---

## 6. Formatting و Line length
- التزم بعرض سطر مناسب (80-100 chars) في الوثائق، ودع الـ formatter يضبط باقي القواعد.

---

## 7. Error handling & async patterns ⚠️
- تجنّب `catch (_) {}` الخالي؛ سجّل الأخطاء أو أعِد رميها.
- استخدم `try/catch` حول العمليات I/O ودوّن أنواع الخطأ المتوقعة.
- `async/await` أفضل من سوية الـ callbacks للوضوح.

```dart
try {
  final data = await apiClient.fetchPatient(id);
} on ApiException catch (e) {
  log(e.message);
  rethrow;
}
```

---

## 8. Widgets & Flutter-specific 📱
- Widgets immutable and `const` constructors when possible:
```dart
class PatientCard extends StatelessWidget {
  const PatientCard({super.key, required this.patient});
  final Patient patient;
  // ...
}
```
- Use `lowerCamelCase` for routes and keys.

---

## 9. Avoid anti-patterns
- لا تستخدم `new` و `var` بدون داعٍ في حالات المفهوم الخاطئ؛ استخدم `final`.
- احذر من side-effects في getters.

---

## 10. Lints, Analysis & Tooling 🔎
- Recommended: enable analysis options and use lints packages (e.g., `package:lints/recommended` or community `effective_dart` sets).
- Use `dart analyze` in CI and fail PRs on analyzer errors (warnings as a policy).

Sample `analysis_options.yaml` snippet:
```yaml
include: package:lints/recommended
analyzer:
  errors:
    avoid_print: error
linter:
  rules:
    - prefer_const_constructors
    - use_rethrow_when_possible
    - prefer_final_locals
```

---

## 11. Examples — Good vs Bad
**Bad:**
```dart
var a = [];
void doSomething(){
 print('hi');
}
```
**Good:**
```dart
final List<String> items = [];
void doSomething() {
  // Use logging library instead of print
  logger.info('hi');
}
```

---

## 12. Quick checklist for PR reviewers ✅
- [ ] `dart format` passed
- [ ] `dart analyze` clean
- [ ] public members documented with `///`
- [ ] const/ final used where relevant
- [ ] tests added/updated for behavior

---

## المصادر
- https://dart.dev/effective-dart/style
- https://dart.dev/tools/dart-format
- https://dart.dev/tools/analyzer

---

**ملاحظة:** أستطيع توسيع هذا المستند ليشمل قواعد تفصيلية للـ DTOs، naming conventions لفولدرات Flutter، والـ analysis_options.yaml كامل للمشروع إن أردت.