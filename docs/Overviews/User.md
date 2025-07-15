---
title: User Model
description: Detailed overview of the User model, its fields, and relationships.
---

# User Model

## Simplified Overview

Think of the **User Model** as the official digital ID card for every person who uses the application. Just like your driver's license identifies you in the real world, the User Model identifies you within the app.

-   **Who are you?**: It holds your basic information, like your name and email address.
-   **How do we know it's you?**: It's what the system uses to recognize you and let you in when you log in with your password or through services like Google and Apple.
-   **What's connected to you?**: It acts as the central hub that links you to all of your personal information and activities within the app, such as your saved addresses, your shopping cart, your preferences, and your history.

Essentially, without this "digital ID card," the application wouldn't be able to recognize you or manage your personal data securely.

---

## What is its function?

The `User` model is fundamental to several vital operations in the system:

-   **Authentication**: It's the model used by Laravel's authentication system to verify a user's identity upon login (via email, phone number, or Apple ID).
-   **Authorization**: Through its relationship with Roles and Permissions (via Spatie's `HasRoles` trait), it determines what a user can and cannot access. For example, it checks if the user has the 'admin' role to access the Filament admin panel.
-   **User Data Storage**: It stores personal information like name, email, avatar, and preferred language.
-   **Notifications**: It utilizes the `Notifiable` trait, allowing the system to send notifications to users.
-   **Activity Logging**: It uses the `LogsActivity` trait to track and log significant activities performed by the user.
-   **Data Relationships**: It acts as a primary link to most other data in the system, such as cycles, favorite articles, shopping carts, and more.

## Fields

These are the fields in the `users` table, as defined in `schema.dbml`.

| Field                 | Type        | Description                                                                     |
| --------------------- | ----------- | ------------------------------------------------------------------------------- |
| `id`                  | `bigint`    | The unique identifier for the user (Primary Key).                               |
| `verificationCode`    | `varchar`   | **(Backend)** A temporary code sent to the user's email for verification. It's checked against user input to confirm email ownership. |
| `name`                | `varchar`   | The user's full name.                                                           |
| `email`               | `varchar`   | The user's email address. **Logic (Backend)**: An attribute mutator ensures that an empty string is saved as `NULL` in the database for data consistency. Must be unique. |
| `email_verified_at`   | `timestamp` | **(Backend)** The timestamp when a user's email is confirmed. A `NULL` value indicates an unverified user. This is used by Laravel's `verified` middleware to protect routes and application features, preventing access until the user verifies their email. |
| `apple_id`            | `varchar`   | **(Backend)** The unique user identifier from Apple's authentication service. It's used to find existing users and create new ones during the "Sign in with Apple" flow. It also serves as a key differentiator for segmenting users in admin panels. |
| `mobile`              | `varchar`   | The user's phone number, used for login. Must be unique.                        |
| `phone_verified_at`   | `timestamp` | The date and time when the phone number was verified.                           |
| `password`            | `varchar`   | The user's password, stored in a hashed format.                                 |
| `lang`                | `varchar`   | The user's preferred language in the application (default is 'ar').             |
| `timezone_utc`        | `varchar`   | The user's timezone (default is 'Asia/Riyadh').                                 |
| `avatar`              | `varchar`   | A link to the user's profile picture.                                           |
| `remember_token`      | `varchar`   | A token used to remember the user's session on the web ("Remember Me" feature). |
| `country_id`          | `bigint`    | Foreign key to the `countries` table, identifying the user's country.           |
| `country_code`        | `varchar`   | **(Backend)** The mobile country code (e.g., "+966") required whenever a mobile number is provided. |
| `salla_account_id`    | `bigint`    | Foreign key to `salla_accounts`, for linking to a Salla e-commerce account.   |
| `deleted_salla_account_id` | `bigint` | **(Backend)** An archival field that stores the ID of a Salla account that was previously linked to the user. This is populated automatically by an observer when the `salla_account_id` is changed. |
| `deleted_at`          | `timestamp` | The timestamp for soft-deleted records, allowing for account restoration.       |
| `created_at`/`updated_at` | `timestamp` | Timestamps for when the user record was created and last updated.                 |

## Business Logic and Special Methods (Backend)

The `User` model contains important business logic beyond simple data storage.

### Computed Attributes (Backend)

-   **`is_beta_user`**: This is a read-only attribute that returns `true` if the user's email is in a predefined list of beta testers. This logic is handled by the `StoreService` and its results are cached. This is used to grant access to beta features.
-   **`has_won_special_coupon`**: This is a read-only attribute that returns `true` if the user has already been awarded a special coupon. It is used in the `PrizeService` to prevent users from winning this type of prize more than once.

### Custom Class Methods (Backend)

-   **`canAccessFilament()`**: A critical authorization method that **returns `true` only if the user has the 'admin' role**. This is used by the Filament admin panel to control access.
-   **`getActivitylogOptions()`**: **Configures the activity logger** for the User model. It ensures that any changes to a user's fillable attributes are automatically recorded in the 'user' log for auditing.
-   **`log_change(...)`**: A helper method to **create an `AuditLog` record**. It's used to track important changes made by a user, such as modifications to their health symptoms.
-   **`getUsersSignUpBy...()`**: A set of static methods (`getUsersSignUpByMobile`, `getUsersSignUpByGoogle`, `getUsersSignUpByApple`) used to **get quick statistics on user registration channels**, likely for dashboards or reports.

## Relationships

- **Country** (belongsTo): [Country model](./Country.md) - User's country for localization and regional features
- **Client** (hasOne): [Client](./Client.md) - Extended health profile for menstrual cycle tracking
- **Cycles** (hasMany): [Cycle model](./Cycle.md) - Individual menstrual cycle records and predictions
- **Cycle Days** (through Cycles): [CycleDay model](./CycleDay.md) - Individual days within cycles for detailed tracking
- **Articles** (belongsToMany): [Article model](./Article.md) - Articles user can access or has interacted with
- **Favourite Articles** (belongsToMany): [Article model](./Article.md) - Articles marked as favorites by the user
- **Read Articles** (belongsToMany): [Article model](./Article.md) - Articles the user has read (tracking)
- **Pregnant Weeks** (hasMany): Pregnancy milestone tracking records
- **FCM Tokens** (hasMany): Device tokens for push notifications **(Backend)**
- **Categories** (belongsToMany): [Category model](./Category.md) - User's interest categories for content filtering
- **Deletion Requests** (hasMany): Account deletion requests **(Backend)**
- **Audit Logs** (hasMany): Activity logs for change tracking **(Backend)**
- **Pregnants** (hasMany): Pregnancy tracking records
- **Days** (hasMany): Daily symptom tracking records **(Backend)**
- **Ads/Puzzles** (belongsToMany): Puzzle game engagement tracking **(Backend)**
- **Salla Account** (belongsTo): E-commerce account linkage for shopping features
- **Prizes** (hasMany): Contest winnings and rewards **(Backend)**
- **Carts** (hasMany): Shopping cart items for e-commerce **(Backend)**
- **Chats** (hasMany): AI chat conversations **(Backend)**
- **Special Coupon** (hasOneThrough): Won special coupon through prizes **(Backend)**
- **Wishes List** (belongsToMany): Saved e-commerce products for later purchase
- **Address** (hasMany): Shipping addresses for e-commerce

---