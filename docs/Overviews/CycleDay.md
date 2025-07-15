# CycleDay Model

## Simplified Overview

The **CycleDay Model** represents individual days within a menstrual cycle, providing detailed day-by-day tracking and data collection. It's like having a daily diary entry for each day of a menstrual cycle.

- **Daily tracking**: Each day of a cycle gets its own record for detailed symptom and data logging
- **Symptom collection**: Stores daily symptoms, mood, and health indicators for that specific cycle day
- **Educational content**: Connects relevant tips and advice to specific cycle days
- **Date mapping**: Maps actual calendar dates to cycle day numbers for accurate tracking

Think of it as the daily detail page in your cycle tracking journal, where each day holds specific information about symptoms, feelings, and health data.

## What is its function?

**(Backend)** The CycleDay model serves as the detailed daily tracking entity within each menstrual cycle, enabling precise symptom logging, educational content delivery, and granular cycle analysis for comprehensive health insights.

## Fields

| Field | Type | Description |
|-------|------|-------------|
| `cycle_id` | Integer | Foreign key linking cycle day to its parent [Cycle model](./Cycle.md) |
| `day_number` | Integer | **Sequential day number** within the cycle (1, 2, 3...) for cycle progression tracking |
| `date` | Date | **Actual calendar date** for this cycle day, enabling real-world date mapping and scheduling |

## Business Logic and Special Methods (Backend)

### Daily Cycle Tracking System

**(Backend)** The CycleDay model implements granular daily tracking within menstrual cycles:

- **Sequential day numbering**: Each CycleDay has a `day_number` starting from 1 within its parent cycle
- **Calendar date mapping**: Real calendar `date` field enables scheduling and real-world tracking
- **Automated generation**: CycleDays are automatically created when parent Cycle is established
- **Cascade deletion**: When parent cycle is deleted, all related cycle days are automatically removed

### Symptom Data Collection System

**(Backend)** CycleDay serves as a polymorphic container for daily symptom tracking:

```php
// Polymorphic relationship for symptoms
public function day_symptoms()
{
    return $this->hasMany(SymptomAttributeValue::class, 'dayable_id');
}
```

- **Polymorphic target**: Acts as `dayable` entity for symptom data through `dayable_id` and `dayable_type`
- **Daily symptom logging**: Users can record multiple symptoms per cycle day
- **Symptom categorization**: Supports various symptom types (mood, physical, etc.) per day
- **Historical tracking**: Enables pattern analysis across multiple cycles

### Educational Content Association

**(Backend)** Dynamic tip and content delivery system:

```php
// Many-to-many polymorphic relationship with tips
public function tips()
{
    return $this->morphToMany(Tip::class, 'tipable');
}
```

- **Contextual tips**: Educational content associated with specific cycle days
- **Goal-based filtering**: Tips filtered by user goals (follow_cycle, be_pregnant, etc.)
- **Dynamic content**: Tips change based on cycle day number and user context
- **Multilingual support**: Tips support Arabic and English content

### Cycle Validation and Analytics

**(Backend)** Advanced validation and pattern recognition:

- **Normal duration validation**: System validates against `config('cycles.normal_cycle_days')` (default: 7 days)
- **Abnormality detection**: When cycle days exceed normal parameters, parent cycle gets `upnormal` flag
- **Pattern analysis**: Daily data contributes to cycle length calculations and predictions
- **PDF reporting**: CycleDay data used in comprehensive cycle analysis reports

### Performance and Data Management

**(Backend)** Optimized for high-volume daily tracking:

- **Composite indexing**: Index on `cycle_id, date` optimizes common queries
- **Bulk operations**: Efficient creation and deletion of cycle day ranges
- **Eager loading**: Often loaded with parent cycle and symptom data to reduce database queries
- **Memory optimization**: Supports large datasets for long-term cycle tracking

### Data Integrity and Constraints

**(Backend)** Comprehensive data validation:

- **Foreign key constraints**: Ensures referential integrity with parent Cycle
- **Date validation**: Calendar dates must align with cycle progression
- **Unique constraints**: Prevents duplicate day numbers within same cycle
- **Cascade operations**: Maintains data consistency during cycle modifications

## Relationships

- **Cycle** (belongsTo): [Cycle model](./Cycle.md) - Parent cycle containing this day
- **Day Symptoms** (hasMany): SymptomAttributeValue - Daily symptom records for this cycle day
- **Tips** (morphToMany): Tip - Educational content associated with this cycle day

--- 