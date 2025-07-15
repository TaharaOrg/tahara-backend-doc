# Category Model

## Simplified Overview

Think of the **Category Model** as the organization system for all content in the app. It's like having smart folders that not only organize articles and campaigns, but also understand user ages and preferences to show appropriate content.

- **Content organization**: Groups articles and campaigns into meaningful topics like "Nutrition", "Exercise", "Mental Health"
- **Age-appropriate filtering**: Some categories are marked as sensitive and only shown to users 18+
- **User personalization**: Users can select their interests to get personalized content recommendations
- **Dual-purpose design**: Works for both articles and campaigns with type differentiation
- **Multilingual support**: Category names appear in both Arabic and English based on user preference

Think of it as your smart content curator that knows what you're interested in and what's appropriate for your age.

## What is its function?

**(Backend)** The Category model serves as the intelligent content classification system that manages polymorphic relationships, age-sensitive content filtering, and personalized content delivery across multiple content types.

## Fields

| Field               | Type        | Description                                                                      |
|---------------------|-------------|----------------------------------------------------------------------------------|
| `id`                | `bigint`    | Primary key - unique identifier for each category                                |
| `type`              | `enum`      | Content type: **'article'** or **'campaign'** - distinguishes category purpose |
| `name`              | `text`      | **Translatable** - Category name in Arabic and English                          |
| `photo`             | `varchar`   | Category icon/image URL for visual representation                                |
| `sensetive_content` | `varchar`   | **(Backend)** Age restriction flag: **'1'** (18+) or **'0'** (all ages)       |
| `created_at`        | `timestamp` | Record creation timestamp                                                        |
| `updated_at`        | `timestamp` | Record last update timestamp                                                     |

## Business Logic and Special Methods (Backend)

### Translation System

**(Backend)** Dynamic language switching based on user preferences:

- **`getCategoryNameAttribute()`**: Smart name retrieval that checks:
  - Authenticated user's language preference (`Auth::user()->lang`)
  - Request header language (`request()->header('lang')`)
  - Automatic fallback to Arabic if no preference set

### Age-Sensitive Content Control

**(Backend)** Sophisticated age-based content filtering system:

- **Sensitive content detection**: `sensetive_content = '1'` marks 18+ content
- **Automatic age filtering**: System automatically excludes sensitive categories for users under 18
- **Goal-based filtering**: Content shown based on user's health goals and age combination

```php
// Example: Age-based filtering in action
if ($age < 18) {
    $interests = Category::where('sensetive_content', 0)->get();
} else {
    $interests = Category::all();
}
```

### Polymorphic Content Management

**(Backend)** Advanced relationship handling for multiple content types:

- **Type-specific queries**: Automatically filters categories by type (article vs campaign)
- **Polymorphic relationships**: Single category can organize both articles and campaigns
- **Admin separation**: Different admin interfaces for article vs campaign categories

### User Interest Matching

**(Backend)** Personalized content recommendation engine:

- **Interest-based filtering**: Matches user's selected categories with available content
- **Fallback system**: Shows age-appropriate content if user hasn't set interests
- **Dynamic content**: Updates recommendations as user preferences change

## Relationships

- **Articles** (morphedByMany): [Article model](./Article.md) - Articles belonging to this category (via categoryables pivot)
- **Campaigns** (morphedByMany): Campaign - Campaigns belonging to this category (via categoryables pivot)  
- **Users** (belongsToMany): [User model](./User.md) - Users who have selected this category as an interest

---