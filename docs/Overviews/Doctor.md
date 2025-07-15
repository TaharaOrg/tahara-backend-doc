# Doctor Model

## Simplified Overview

Think of the **Doctor Model** as the expert profile system that adds credibility and medical authority to the app's content. It's like having a team of medical professionals whose expertise backs up the health information provided.

- **Medical credentials**: Stores detailed professional information including specializations, experience, and positions
- **Content authorship**: Links doctors to articles and campaigns they've authored or reviewed
- **Multilingual profiles**: All doctor information is available in both Arabic and English
- **Professional quotes**: Features inspiring or educational quotes from medical experts
- **Visual representation**: Includes professional photos for user trust and recognition

Think of it as your medical advisory board that provides expert-backed health information.

## What is its function?

**(Backend)** The Doctor model serves as the expert credential system that manages multilingual professional profiles and establishes medical authority for content through authorship relationships.

## Fields

| Field        | Type        | Description                                                                      |
|--------------|-------------|----------------------------------------------------------------------------------|
| `id`         | `bigint`    | Primary key - unique identifier for each doctor                                  |
| `name`       | `varchar`   | **Translatable** - Doctor's full name in Arabic and English                     |
| `speciality` | `text`      | **Translatable** - Medical specialization (e.g., "Gynecology", "Nutrition")    |
| `experience` | `text`      | **Translatable** - Professional experience and background details               |
| `position`   | `text`      | **Translatable** - Current job title and institutional affiliation              |
| `qoute`      | `text`      | **Translatable** - Professional quote or medical advice from the doctor         |
| `photo`      | `varchar`   | Professional headshot URL for credibility and user recognition                  |
| `created_at` | `timestamp` | Record creation timestamp                                                        |
| `updated_at` | `timestamp` | Record last update timestamp                                                     |

## Business Logic and Special Methods (Backend)

### Translation System

**(Backend)** Comprehensive multilingual profile management with automatic language detection:

- **`getDoctorNameAttribute()`**: Returns localized doctor name based on request language header
- **`getDoctorSpecialityAttribute()`**: Provides specialty in user's preferred language
- **`getDoctorExperienceAttribute()`**: Shows experience details in appropriate language
- **`getDoctorPositionAttribute()`**: Displays current position in user's language
- **`getDoctorQouteAttribute()`**: Returns inspirational quotes in user's preferred language

### Professional Credibility System

**(Backend)** Features that establish medical authority:

- **Multi-field expertise**: Comprehensive professional information storage
- **Content attribution**: Links doctors to their authored content for credibility
- **Visual trust**: Professional photos enhance user confidence in medical advice

### Language-Smart Content

**(Backend)** All computed attributes automatically detect language preference from:

```php
// Example: Language detection logic
if (request()->header('lang') == 'ar') {
    return $this->getTranslation('name', 'ar');
} else {
    return $this->getTranslation('name', 'en');
}
```

## Relationships

- **Articles** (hasMany): [Article model](./Article.md) - Medical articles authored or reviewed by this doctor
- **Campaigns** (hasMany): Campaign - Health campaigns led or endorsed by this doctor

---