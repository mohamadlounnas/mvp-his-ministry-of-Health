# استراتيجية تعدد اللغات
# Internationalization (i18n) Strategy

## 1. المبدأ الأساسي (Core Principle)

> **العربية هي اللغة الأصلية والافتراضية للنظام.**
> جميع اللغات الأخرى تُحمّل ديناميكياً من ملفات JSON.

```
┌─────────────────────────────────────────────────────┐
│                    GHIS System                      │
├─────────────────────────────────────────────────────┤
│  Default Language: العربية (ar) - Hardcoded         │
│  ───────────────────────────────────────────────────│
│  Dynamic Languages:                                 │
│    ├── /locales/fr.json  (Français)                 │
│    ├── /locales/en.json  (English)                  │
│    └── /locales/*.json   (Any future language)      │
└─────────────────────────────────────────────────────┘
```

---

## 2. هيكل ملفات الترجمة (Translation File Structure)

### 2.1 مجلد الترجمات
```
/locales/
  ├── manifest.json      # قائمة اللغات المتاحة
  ├── fr.json            # الفرنسية
  ├── en.json            # الإنجليزية
  └── ber.json           # الأمازيغية
```

### 2.2 ملف manifest.json
```json
{
  "defaultLanguage": "ar",
  "availableLanguages": [
    {
      "code": "ar",
      "name": "العربية",
      "nativeName": "العربية",
      "direction": "rtl",
      "isDefault": true,
      "isBuiltIn": true
    },
    {
      "code": "fr",
      "name": "French",
      "nativeName": "Français",
      "direction": "ltr",
      "isDefault": false,
      "isBuiltIn": false,
      "file": "fr.json"
    },
    {
      "code": "en",
      "name": "English",
      "nativeName": "English",
      "direction": "ltr",
      "isDefault": false,
      "isBuiltIn": false,
      "file": "en.json"
    }
  ]
}
```

### 2.3 هيكل ملف الترجمة (fr.json example)
```json
{
  "_meta": {
    "language": "fr",
    "version": "1.0.0",
    "lastUpdated": "2026-01-09",
    "completeness": 0.95
  },
  "common": {
    "save": "Enregistrer",
    "cancel": "Annuler",
    "delete": "Supprimer",
    "edit": "Modifier",
    "search": "Rechercher",
    "loading": "Chargement...",
    "error": "Erreur",
    "success": "Succès"
  },
  "patient": {
    "title": "Patient",
    "firstName": "Prénom",
    "lastName": "Nom de famille",
    "dateOfBirth": "Date de naissance",
    "nationalId": "Numéro d'identité nationale",
    "bloodType": "Groupe sanguin",
    "allergies": "Allergies"
  },
  "appointment": {
    "title": "Rendez-vous",
    "schedule": "Planifier un rendez-vous",
    "date": "Date",
    "time": "Heure",
    "doctor": "Médecin",
    "department": "Service"
  },
  "medical": {
    "diagnosis": "Diagnostic",
    "prescription": "Ordonnance",
    "labResults": "Résultats de laboratoire",
    "vitals": "Signes vitaux"
  }
}
```

---

## 3. التطبيق التقني (Technical Implementation)

### 3.1 Backend (NestJS)

```typescript
// src/i18n/i18n.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import * as fs from 'fs/promises';
import * as path from 'path';

interface LanguageManifest {
  defaultLanguage: string;
  availableLanguages: LanguageConfig[];
}

interface LanguageConfig {
  code: string;
  name: string;
  nativeName: string;
  direction: 'ltr' | 'rtl';
  isDefault: boolean;
  isBuiltIn: boolean;
  file?: string;
}

@Injectable()
export class I18nService implements OnModuleInit {
  private translations: Map<string, Record<string, any>> = new Map();
  private manifest: LanguageManifest;
  private readonly localesPath = path.join(process.cwd(), 'locales');

  async onModuleInit() {
    await this.loadManifest();
    await this.loadAllTranslations();
  }

  private async loadManifest() {
    const manifestPath = path.join(this.localesPath, 'manifest.json');
    const content = await fs.readFile(manifestPath, 'utf-8');
    this.manifest = JSON.parse(content);
  }

  private async loadAllTranslations() {
    for (const lang of this.manifest.availableLanguages) {
      if (!lang.isBuiltIn && lang.file) {
        const filePath = path.join(this.localesPath, lang.file);
        const content = await fs.readFile(filePath, 'utf-8');
        this.translations.set(lang.code, JSON.parse(content));
      }
    }
  }

  /**
   * الحصول على ترجمة بمفتاح معين
   * @param key - مفتاح الترجمة (مثال: "patient.firstName")
   * @param lang - رمز اللغة (مثال: "fr")
   * @param fallback - النص الافتراضي (العربي)
   */
  translate(key: string, lang: string, fallback: string): string {
    // إذا كانت اللغة هي العربية (الافتراضية)، نعيد النص كما هو
    if (lang === 'ar' || !lang) {
      return fallback;
    }

    const translation = this.translations.get(lang);
    if (!translation) {
      return fallback;
    }

    // الوصول للمفتاح المتداخل (nested key)
    const value = key.split('.').reduce((obj, k) => obj?.[k], translation);
    return (value as string) || fallback;
  }

  getAvailableLanguages(): LanguageConfig[] {
    return this.manifest.availableLanguages;
  }

  async reloadTranslations() {
    this.translations.clear();
    await this.loadAllTranslations();
  }
}
```

### 3.2 Frontend (Flutter)

```dart
// lib/core/i18n/translation_service.dart
import 'dart:convert';
import 'package:flutter/services.dart';
import 'package:flutter/material.dart';

class TranslationService extends ChangeNotifier {
  String _currentLanguage = 'ar';
  Map<String, dynamic> _translations = {};
  List<LanguageConfig> _availableLanguages = [];

  String get currentLanguage => _currentLanguage;
  bool get isRTL => _currentLanguage == 'ar';

  Future<void> init() async {
    final manifestJson = await rootBundle.loadString('assets/locales/manifest.json');
    final manifest = json.decode(manifestJson);
    
    _availableLanguages = (manifest['availableLanguages'] as List)
        .map((e) => LanguageConfig.fromJson(e))
        .toList();
  }

  Future<void> setLanguage(String langCode) async {
    if (langCode == 'ar') {
      _translations = {};
      _currentLanguage = 'ar';
    } else {
      final langFile = await rootBundle.loadString('assets/locales/$langCode.json');
      _translations = json.decode(langFile);
      _currentLanguage = langCode;
    }
    notifyListeners();
  }

  /// الترجمة مع fallback للعربية
  String t(String key, String arabicDefault) {
    if (_currentLanguage == 'ar') return arabicDefault;
    
    final keys = key.split('.');
    dynamic value = _translations;
    for (final k in keys) {
      value = value?[k];
      if (value == null) return arabicDefault;
    }
    return value.toString();
  }
}

class LanguageConfig {
  final String code;
  final String name;
  final String nativeName;
  final String direction;
  final bool isDefault;

  LanguageConfig({
    required this.code,
    required this.name,
    required this.nativeName,
    required this.direction,
    required this.isDefault,
  });

  factory LanguageConfig.fromJson(Map<String, dynamic> json) {
    return LanguageConfig(
      code: json['code'],
      name: json['name'],
      nativeName: json['nativeName'],
      direction: json['direction'],
      isDefault: json['isDefault'] ?? false,
    );
  }
}
```

### 3.3 استخدام الترجمة في الواجهة

```dart
// في أي Widget
class PatientFormWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final t = context.watch<TranslationService>();
    
    return Column(
      children: [
        Text(t.t('patient.title', 'المريض')),
        TextFormField(
          decoration: InputDecoration(
            labelText: t.t('patient.firstName', 'الاسم الأول'),
          ),
        ),
        TextFormField(
          decoration: InputDecoration(
            labelText: t.t('patient.lastName', 'اسم العائلة'),
          ),
        ),
      ],
    );
  }
}
```

---

## 4. إضافة لغة جديدة (Adding New Language)

### الخطوات:
1. **إنشاء ملف JSON جديد** في `/locales/` (مثال: `es.json` للإسبانية).
2. **تحديث `manifest.json`** بإضافة اللغة الجديدة.
3. **لا يلزم أي تغيير في الكود!**

```json
// إضافة الإسبانية في manifest.json
{
  "code": "es",
  "name": "Spanish",
  "nativeName": "Español",
  "direction": "ltr",
  "isDefault": false,
  "isBuiltIn": false,
  "file": "es.json"
}
```

---

## 5. مزايا هذا النهج (Advantages)

| الميزة | الشرح |
|--------|-------|
| **العربية أولاً** | النصوص العربية مدمجة في الكود، مما يضمن عدم وجود "مفاتيح مفقودة" |
| **التوسع السهل** | إضافة لغة جديدة = إضافة ملف JSON فقط |
| **الأداء** | اللغة العربية لا تحتاج لأي تحميل إضافي |
| **الصيانة** | المترجمون يعملون على ملفات JSON بسيطة بدون لمس الكود |
| **Fallback آمن** | إذا كانت الترجمة ناقصة، يظهر النص العربي |

---

## 6. دعم RTL و LTR

```dart
// في MaterialApp
MaterialApp(
  locale: Locale(translationService.currentLanguage),
  supportedLocales: translationService.availableLanguages
      .map((l) => Locale(l.code))
      .toList(),
  builder: (context, child) {
    return Directionality(
      textDirection: translationService.isRTL 
          ? TextDirection.rtl 
          : TextDirection.ltr,
      child: child!,
    );
  },
)
```
