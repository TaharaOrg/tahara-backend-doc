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
| `actual_end`         | Date        | **Real period end date** - When user manually ends their period (null by default, only set when user taps "My period ended") |
| `ovulation_date`     | Date        | **Main ovulation day** - Calculated as `start + (repeat_period - 14)` where `repeat_period` comes from [Client model](./Client.md) |
| `start_ovulation_date` | Date      | **Fertile window start** - Calculated as `ovulation_date - 2 days` (5-day fertile window begins) |
| `end_ovulation_date` | Date        | **Fertile window end** - Calculated as `ovulation_date + 2 days` (5-day fertile window ends) |
| `current`            | String      | **Legacy field** - Defaults to "0", appears unused in current codebase **(Backend)** |
| `type`               | Enum        | **Cycle classification**: `actual` (real periods logged by user) or `expected` (predicted future periods) |
| `upnormal`           | Text        | **Abnormality flags** - JSON array storing issues like `exceeds_cycle_days` or `exceeds_cycle_repeats` **(Backend)** |

## Business Logic and Special Methods (Backend)

### Two Types of Cycles: Detailed Breakdown

#### 🩸 **Actual Cycles (`type = 'actual'`)**
**Real menstrual periods that the user has experienced and logged:**

- **User-initiated**: Created when user taps "My period started" in the app
- **Manual end date**: User can manually end the cycle by setting `actual_end`
- **Ovulation calculation**: All ovulation dates calculated immediately upon creation
- **CycleDay generation**: Creates individual day records for symptom tracking
- **Validation rules**: Must be at least 21 days apart, cannot overlap with pregnancy
- **Trigger expected cycles**: When an actual cycle is created, it automatically generates the next expected cycle

**Creation process:**
```php
// User logs period start
$cycle = Cycle::create([
    'start' => $request->start,           // User-selected date
    'end' => $start + day_numbers - 1,    // Calculated from Client settings  
    'user_id' => $user->id,
    'type' => 'actual',                   // Real cycle
]);
```

#### 📅 **Expected Cycles (`type = 'expected'`)**
**Predicted future periods based on user's historical patterns:**

- **Auto-generated**: Created automatically after each actual cycle
- **Pattern-based**: Uses `repeat_period` from Client model for timing
- **Smart cleanup**: Old expected cycles are deleted when new actual cycle is logged
- **User interaction**: Shows "Your cycle may start today" with option to convert to actual
- **No ovulation initially**: Only gets ovulation dates if converted to actual cycle
- **Notification system**: Used for PMS and period start reminders

**Auto-generation logic:**
```php
// After logging actual cycle, system creates next expected cycle
$start = Carbon::parse($actual_cycle->start)->addDays($repeat_period);
$expected_cycle = Cycle::create([
    'start' => $start,                    // Predicted start date
    'end' => $start + day_numbers - 1,    // Predicted end date
    'type' => 'expected',                 // Predicted cycle
    'user_id' => $user->id,
]);
```

#### 🔄 **Cycle Lifecycle Management**
- **Expected → Actual**: When user confirms expected cycle started
- **Cleanup mechanism**: Old expected cycles deleted when patterns change
- **Pregnancy handling**: Both types suspended during pregnancy periods
- **Pattern learning**: System adapts predictions based on actual cycle history

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
- **Smart notifications**: Expected cycles trigger PMS reminders 2-4 days before predicted start
- **Conversion capability**: Expected cycles can be converted to actual when user confirms period started

### Data Integrity and Validation

**(Backend)** Comprehensive validation ensures data accuracy:

- **21-day minimum gap**: Enforced between cycles to prevent unrealistic entries
- **Pregnancy conflict prevention**: Cycles cannot overlap with pregnancy periods
- **Duplicate prevention**: Same start date cannot be used for multiple actual cycles
- **Date logic validation**: End dates must be after start dates with proper constraints

### Historical Analysis Features

**(Backend)** Advanced pattern recognition:

- **Cycle length calculation**: Tracks individual cycle lengths for pattern analysis
- **Abnormality detection**: Flags cycles exceeding normal parameters
- **Trend monitoring**: Identifies changes in user's cycle patterns over time
- **Filter support**: Provides date ranges for historical cycle analysis

### Abnormality Detection

**(Backend)** Monitors cycle patterns and flags irregularities:

- **`exceeds_cycle_days`**: Period longer than normal (> 7 days)
- **`exceeds_cycle_repeats`**: Cycle length longer than normal (> 35 days)
- **Automatic flagging**: System sets these flags during cycle creation/updates

### Query Scopes

- **`withWhereHas`**: Advanced scope for eager loading with constraints

## Relationships

- **User** (belongsTo): [User model](./User.md) - The user who owns this cycle
- **Cycle Days** (hasMany): [CycleDay model](./CycleDay.md) - Individual days within this cycle for detailed tracking

---