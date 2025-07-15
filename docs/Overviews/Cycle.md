# Cycle Model

## Simplified Overview

Think of the **Cycle Model** as the detailed record keeper for each menstrual cycle. It's like having a diary that tracks every menstrual cycle a user experiences - both the ones that actually happened and the ones the app predicts will happen.

- **Individual cycle tracking**: Each cycle gets its own record with start/end dates and cycle details
- **Two types of cycles**: "actual" cycles (real periods the user logged) and "expected" cycles (predicted future periods)
- **Smart calculations**: Automatically calculates ovulation dates, fertile windows, and cycle patterns
- **Detailed daily breakdown**: Each cycle is connected to individual cycle days for symptom tracking

Think of it as your personal menstrual cycle historian that remembers every detail and helps predict future patterns.

## What is its function?

**(Backend)** The Cycle model serves as the core tracking entity for individual menstrual cycles, managing both historical data and future predictions to provide comprehensive cycle tracking and health insights.

## Fields

| Field                 | Type        | Description                                                                      |
|----------------------|-------------|----------------------------------------------------------------------------------|
| `user_id`            | Integer     | Foreign key linking cycle to its owner from [User model](./User.md)            |
| `start`              | Date        | **Period start date** - When menstrual bleeding begins (user-entered for actual cycles, calculated for expected) |
| `end`                | Date        | **Predicted period end date** - Calculated as `start + day_numbers - 1` from [Client model](./Client.md) |
| `actual_end`         | Date        | **Real period end date** - When user actually stopped bleeding (null until user manually ends cycle) |
| `ovulation_date`     | Date        | **Main ovulation day** - Calculated as `start + (repeat_period - 14)` where `repeat_period` comes from [Client model](./Client.md) |
| `start_ovulation_date` | Date      | **Fertile window start** - Calculated as `ovulation_date - 2 days` (5-day fertile window begins) |
| `end_ovulation_date` | Date        | **Fertile window end** - Calculated as `ovulation_date + 2 days` (5-day fertile window ends) |
| `current`            | String      | **Legacy field** - Defaults to "0", appears unused in current codebase **(Backend)** |
| `type`               | Enum        | **Cycle classification**: `actual` (real periods logged by user) or `expected` (predicted future periods) |
| `upnormal`           | Text        | **Abnormality flags** - JSON array storing issues like `exceeds_cycle_days` or `exceeds_cycle_repeats` **(Backend)** |

## Business Logic and Special Methods (Backend)

### Key Calculations Explained

**Ovulation Calculation Logic:**
```php
$ovulation_date = Carbon::parse($cycle->start)
    ->addDays($user->client->repeat_period - 14)
    ->format('Y-m-d');
```
- `repeat_period` comes from Client model (default: 28 days = full cycle length)
- Formula: **Period Start + (Cycle Length - 14 days)** 
- Example: Start Jan 1 + (28 - 14) = Jan 15 ovulation

**Fertile Window Calculation:**
```php
'start_ovulation_date' => $ovulation_date - 2 days,  // Jan 13
'end_ovulation_date' => $ovulation_date + 2 days     // Jan 17  
```
- Creates 5-day fertile window centered on ovulation day
- Used for pregnancy planning and cycle tracking

### Cycle Management System

**(Backend)** The Cycle model implements sophisticated cycle management that handles both actual period tracking and predictive cycle generation:

- **Automatic ovulation calculation**: `ovulation_date = start + (repeat_period - 14)`
- **Fertile window generation**: 5-day window centered around ovulation date
- **Cycle validation**: Ensures minimum 21-day gap between cycles
- **Abnormality detection**: Flags cycles that exceed normal parameters

### Cycle Day Generation

**(Backend)** Automatically creates individual CycleDay records for each day of the cycle:

```php
// Example: Creating cycle days for a 5-day period
for ($i = 1; $i <= $day_numbers + 1; $i++) {
    CycleDay::create([
        'cycle_id' => $cycle->id,
        'day_number' => $i,
        'date' => $i == 1 ? $cycle->start : Carbon::parse($cycle->start)->addDays($i - 1)
    ]);
}
```

### Expected Cycle Prediction

**(Backend)** The system automatically generates future expected cycles based on user patterns:

- **Auto-generation**: Creates next expected cycle when actual cycle is logged
- **Pattern-based prediction**: Uses `repeat_period` from user's client settings
- **Cleanup mechanism**: Removes old expected cycles when new actual cycle is created

### Abnormality Detection

**(Backend)** Monitors cycle patterns and flags irregularities:

- **`exceeds_cycle_days`**: Period longer than normal (> 7 days)
- **`exceeds_cycle_repeats`**: Cycle length longer than normal (> 35 days)
- **Automatic flagging**: System sets these flags during cycle creation/updates

### Query Scopes

- **`withWhereHas`**: Advanced scope for eager loading with constraints

## Relationships

- **User** (belongsTo): [User model](./User.md) - The user who owns this cycle
- **Cycle Days** (hasMany): CycleDay - Individual days within this cycle for detailed tracking

---