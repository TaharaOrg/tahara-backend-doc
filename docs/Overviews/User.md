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

## Fields

These are the fields in the `users` table, as defined in `schema.dbml`.

| Field                 | Type        | Description                                                                     |
| --------------------- | ----------- | ------------------------------------------------------------------------------- |
| `id`                  | `bigint`    | The unique identifier for the user (Primary Key).                               |
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
| `salla_account_id`    | `bigint`    | Foreign key to `salla_accounts`, for linking to a Salla e-commerce account.   |
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

The `User` model is the most connected model in the application.

-   **`address()`**: A `HasMany` relationship with the `Address` model, **to get the user's saved addresses**.
-   **`answers()`**: A `HasMany` relationship with `UserAnswer`, **to retrieve the user's answers to puzzle questions**.
-   **`articles()`**: A `BelongsToMany` relationship with `Article`, **for user interactions with articles**. See also `favourite_articles` and `read_articles`.
-   **`audit_logs()`**: A `HasMany` relationship with `AuditLog`, **to get a history of changes made by the user**.
-   **`carts()`**: A `HasMany` relationship with the `Cart` model, **to retrieve items currently in the user's shopping cart**.
-   **`categories()`**: A `BelongsToMany` relationship with `Category`, **to manage the user's selected categories of interest**.
-   **`chats()`**: A `HasMany` relationship with `Chat`, **to retrieve all chat sessions started by the user**.
-   **`client()`**: A `HasOne` relationship with the `Client` model, **which stores detailed data about the user as a client** (e.g., usage goal, birthdate).
-   **`country()`**: A `BelongsTo` relationship with the `Country` model, **defining the user's country**.
-   **`cycles()`**: A `HasMany` relationship with the `Cycle` model, **to retrieve all menstrual cycles**. This is further refined by:
    -   `expected_cycles()`: **Returns only cycles of type 'expected'**.
    -   `actual_cycles()`: **Returns only cycles of type 'actual'**.
-   **`days()`**: A `HasMany` relationship with the `Day` model. **(Backend)** This model serves as a daily record to which a user's general health symptoms (those not tied to a specific cycle) are attached.
-   **`deletion_requests()`**: A `HasMany` relationship with `DeletionRequest`, **to view account deletion requests made by the user**.
-   **`favourite_articles()`**: A `BelongsToMany` relationship with `Article`, **to get the user's list of favorite articles**.
-   **`fcm_tokens()`**: A `HasMany` relationship with `FcmToken`, **to manage notification tokens for the user's devices**.
-   **`pregnants()` & `pregnant_weeks()`**: `HasMany` relationships **for tracking pregnancy periods and weekly progress**.
-   **`prizes()`**: A `HasMany` relationship with `Prize`, **to list all prizes the user has won**.
-   **`read_articles()`**: A `BelongsToMany` relationship with `Article`, **to track which articles the user has read**.
-   **`roles()`**: A `BelongsToMany` relationship (from a trait) **to manage the roles (e.g., 'admin') assigned to the user**.
-   **`sallaAccount()`**: A `BelongsTo` relationship with `SallaAccount`, **to fetch data from the linked "Salla" account**.
-   **`specialCoupon()`**: A `HasOneThrough` relationship that **underpins the `has_won_special_coupon` attribute**.
-   **`wishesList()`**: A `BelongsToMany` relationship with `SallaProduct`, **to manage the user's product wishlist**.

---
*Note: Links to related models will be added later once their pages are created.* 