---
title: Client Model
description: Detailed overview of the Client model, its fields, and relationships.
---

# Client Model

## Simplified Overview

Think of the **Client Model** as the detailed health profile for each user of the application. While the User model is like your basic ID card, the Client model is like your medical chart that tracks your reproductive health journey.

- **Your health goals**: It stores what you're trying to achieve - tracking your cycle, trying to get pregnant, or monitoring an existing pregnancy.
- **Your personal timeline**: It remembers important dates like when your last cycle started, your birthdate, and pregnancy periods.
- **Your app experience**: It personalizes the app based on your age (like showing age-appropriate content) and caches your dashboard data for faster loading.

Essentially, this is where all your personal health tracking information lives, separate from your basic account information.

---

## What is its function?

**(Backend)** The Client model serves as the core health tracking entity that manages:
- Menstrual cycle data and predictions
- Health goals (cycle tracking vs pregnancy)
- Age-sensitive content control
- Dashboard data caching for performance
- State machine implementation for cycle status

## Fields

| Field                 | Type        | Description                                                                      |
|----------------------|-------------|----------------------------------------------------------------------------------|
| `id`                 | `bigint`    | Primary key                                                                      |
| `user_id`            | `bigint`    | Foreign key to [User model](./User.md) (one-to-one relationship)                |
| `ovulation_date`     | `date`      | **(Backend)** Last calculated ovulation date for cycle predictions              |
| `birthdate`          | `date`      | User's birth date for age calculations and content filtering                    |
| `mode`               | `string`    | **(Backend)** Age mode: "+18" or "-18" for content filtering                   |
| `goal`               | `enum`      | Health tracking goal: `follow_cycle`, `be_pregnant`, `follow_pregnant`         |
| `day_numbers`        | `int`       | **Length of menstrual bleeding period in days (not full cycle)**. Default: 5   |
| `repeat_period`      | `int`       | **Length of complete menstrual cycle in days**. Default: 28                    |
| `latest_cycle_start` | `date`      | Date when the user's most recent cycle started                                 |
| `status`             | `string`    | **(Backend)** Current cycle status managed by state machine                    |
| `show_notification`  | `boolean`   | Whether to show notifications to the user. Default: true                       |
| `pregnant_start`     | `date`      | Start date of pregnancy (for pregnancy tracking goal)                          |
| `pregnant_end`       | `date`      | Expected end date of pregnancy                                                  |
| `pregnant_id`        | `bigint`    | **(Backend)** Reference to pregnancy tracking record                           |
| `IDK`                | `text`      | **(Backend)** Flag indicating if user doesn't know their cycle details         |
| `home_data`          | `text`      | **(Backend)** Cached JSON data for dashboard to improve performance            |
| `expire_home_data_at`| `datetime`  | **(Backend)** Expiration timestamp for cached home data                        |
| `turn_into_18_pop_up`| `string`    | **(Backend)** Controls 18+ content popup display. Default: "0"                 |
| `created_at`         | `timestamp` | Record creation timestamp                                                        |
| `updated_at`         | `timestamp` | Record last update timestamp                                                     |

## Business Logic and Special Methods (Backend)

### State Machine Integration

**(Backend)** The Client model implements a sophisticated state machine using `CycleStatusStateMachine` to track and manage different phases of the menstrual cycle and reproductive health journey.

**Available States:**
- `no_event` - Default state, no specific cycle phase
- `period` - Active menstruation period  
- `ovulation` - Fertile/ovulation window
- `delay_period` - Late period detection
- `pregnancy` - Confirmed pregnancy state

### State Transitions

The state machine automatically transitions between states based on:
- **Date calculations**: Using `last_period_date`, `day_numbers`, and `repeat_period`
- **User actions**: Manual period logging, pregnancy confirmation
- **Time-based triggers**: Automatic detection of late periods or ovulation windows

```php
// Example: Check current cycle state
$client->currentState(); // Returns current state name

// Example: Transition to period state
$client->startPeriod();

// Example: Handle late period
if ($client->isDaysPastDue() > 3) {
    $client->transitionTo('delay_period');
}
```

### Computed Attributes

- **`isCompletedCycleSetup`**: Returns `true` if user has completed basic cycle setup (has `last_period_date` and both `day_numbers` and `repeat_period` are not null)

### Caching Mechanism

**(Backend)** The model implements a sophisticated caching system for dashboard performance:

- **`home_data`** (JSON): Cached dashboard data including cycle predictions, health insights, and personalized content
- **`expire_home_data_at`** (timestamp): Cache expiration time, automatically refreshed when data becomes stale
- **Automatic invalidation**: Cache is cleared when cycle data changes (new period, updated settings, etc.)
- **Performance optimization**: Reduces complex calculations on every dashboard load

## Relationships

- **User** (belongsTo): [User model](./User.md) - The base user account that owns this health profile
- **Cycles** (through User): [Cycle model](./Cycle.md) - Individual menstrual cycles tracked for this client

--- 