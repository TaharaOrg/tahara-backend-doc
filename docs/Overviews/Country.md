# Country Model

## Simplified Overview

Think of the **Country Model** as the geographical foundation that powers location-based features throughout the app. It's like having a smart world map that not only knows country names in multiple languages, but also understands tax rules, currency systems, and phone number formats.

- **Localization support**: Country names appear in both Arabic and English based on user preference
- **Tax calculation**: Knows which countries require tax calculations for e-commerce features
- **Communication formatting**: Stores mobile codes for proper international phone number handling
- **Currency awareness**: Links countries to their currencies for financial features
- **User profiling**: Helps personalize app experience based on geographical location

Think of it as your international assistant that handles all the complex geographical and regulatory details.

## What is its function?

**(Backend)** The Country model serves as the comprehensive geographical reference system that manages international localization, tax compliance, currency handling, and mobile communication formatting across the global user base.

## Fields

| Field         | Type           | Description                                                                                                    |
|---------------|----------------|--------------------------------------------------------------------------------------------------------------------------------|
| `id`          | `bigint`       | Primary key - unique identifier for each country                                                                               |
| `name`        | `varchar(255)` | **Default country name** - Primary name, typically in Arabic.                                                                  |
| `name_en`     | `varchar(255)` | **(Backend)** English country name used for localization. Nullable.                                                            |
| `code`        | `varchar(2)`   | **(Backend)** ISO 3166-1 alpha-2 country code (e.g., 'SA', 'US', 'EG').                                                        |
| `mobile_code` | `varchar(255)` | **(Backend)** International dialing code (e.g., '+966', '+1', '+20').                                                          |
| `currency`    | `varchar(255)` | **(Backend)** National currency name (e.g., 'Saudi Riyal', 'US Dollar').                                                       |
| `is_taxable`  | `tinyint`      | **(Backend)** Tax requirement flag (`0` = no, `1` = yes). Defaults to `0`.                                                     |
| `created_at`  | `timestamp`    | Record creation timestamp.                                                                                                     |
| `updated_at`  | `timestamp`    | Record last update timestamp.                                                                                                  |
| `deleted_at`  | `timestamp`    | **(Backend)** Soft deletion timestamp for data integrity.                                                                      |

## Business Logic and Special Methods (Backend)

### Translation System

**(Backend)** Smart country name localization with user preference detection:

- **`translateName()`**: Advanced computed attribute that provides localized country names
- **Multi-source language detection**: Checks authenticated user language, then request headers
- **Automatic fallback**: Gracefully handles missing language preferences

```php
// Example: Smart language detection
if (Auth::user()) {
    if (Auth::user()->lang == 'ar') {
        return $this->getTranslation('name', 'ar');
    } else {
        return $this->getTranslation('name', 'en');
    }
} else {
    if (request()->header('lang') == 'ar') {
        return $this->getTranslation('name', 'ar');
    } else {
        return $this->getTranslation('name', 'en');
    }
}
```

### Tax Compliance System

**(Backend)** E-commerce tax calculation framework:

- **Tax jurisdiction identification**: `is_taxable` flag determines tax calculation requirements
- **Saudi Arabia special case**: Only Saudi Arabia is marked as taxable (VAT compliance)
- **StoreService integration**: Used by StoreService to determine if customer country requires tax

### Mobile Communication Support

**(Backend)** International phone number handling:

- **Standardized mobile codes**: All stored with '+' prefix for consistency
- **Validation support**: Used in user registration and profile updates
- **International compatibility**: Supports global user base with proper formatting

### Data Integrity Features

**(Backend)** Robust data management:

- **Soft deletes**: Uses `SoftDeletes` trait for data preservation
- **Unique constraints**: Country code must be unique for data integrity
- **Comprehensive seeding**: Includes 100+ countries with full data

## Relationships

- **Users** (hasMany): [User model](./User.md) - Users who belong to this country for localization and tax purposes

---