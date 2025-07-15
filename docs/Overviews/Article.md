# Article Model

## Simplified Overview

Think of the **Article Model** as the content library of the application. It's like having a smart health magazine that knows exactly what articles to show each user based on their health goals, age, and interests.

- **Multilingual content**: Every article is written in both Arabic and English with smart language switching
- **Goal-specific targeting**: Articles can be aimed at different health goals (cycle tracking, pregnancy, etc.)
- **Expert-authored content**: Each article can be linked to a doctor for credibility
- **Smart content filtering**: Shows age-appropriate content and respects user preferences
- **Reading tracking**: Remembers what users have read and what they've favorited

Think of it as your personalized health content that adapts to your journey and preferences.

## What is its function?

**(Backend)** The Article model serves as the comprehensive content management system that delivers personalized, multilingual health information with advanced filtering, tracking, and relationship management capabilities.

## Fields

| Field       | Type        | Description                                                                      |
|-------------|-------------|----------------------------------------------------------------------------------|
| `id`        | `bigint`    | Primary key - unique identifier for each article                                 |
| `doctor_id` | `bigint`    | Foreign key linking to the expert doctor who authored/reviewed this content     |
| `viewed`    | `bigint`    | **(Backend)** View counter - tracks how many times article has been accessed    |
| `slug`      | `text`      | **(Backend)** URL-friendly identifier for routing (e.g., "pregnancy-nutrition") |
| `title`     | `text`      | **Translatable** - Article title in Arabic and English                          |
| `body`      | `text`      | **Translatable** - Main article content in Arabic and English                   |
| `meta`      | `json`      | **Translatable** - SEO meta description in Arabic and English                   |
| `references`| `text`      | **Translatable** - Scientific references and sources in both languages          |
| `featured`  | `boolean`   | **(Backend)** Marks if article appears in featured sections (default: false)   |
| `photo`     | `varchar`   | Article header image URL                                                         |
| `goal`      | `enum`      | Target audience: **'follow_cycle'**, **'be_pregnant'**, **'follow_pregnant'**, or **null** (general) |
| `status`    | `tinyint`   | **(Backend)** Visibility status: **1** (shown) or **0** (hidden) - controlled by HasShowedScope |
| `created_at`| `timestamp` | Record creation timestamp                                                        |
| `updated_at`| `timestamp` | Record last update timestamp                                                     |

## Business Logic and Special Methods (Backend)

### Translation System

**(Backend)** Advanced multilingual content management with automatic language detection:

- **`getArticleTitleAttribute()`**: Smart title retrieval based on user language or request header
- **`getArticleBodyAttribute()`**: Dynamic content switching between Arabic and English
- **`getArticleMetaAttribute()`**: SEO-friendly meta descriptions in user's preferred language
- **`getArticleReferencesAttribute()`**: Scientific references adapted to user's language

### Global Scope Management

**(Backend)** Automatic content visibility control through HasShowedScope:

- **HasShowedScope**: Automatically filters articles to show only `status = 1` (visible)
- **Admin override**: Backend controllers can bypass scope using `withoutGlobalScope()`
- **Content moderation**: Provides seamless content approval workflow

### Search Integration

**(Backend)** Full-text search capabilities:

- **Spatie\Searchable**: Implements advanced search functionality across title and content
- **`getSearchResult()`**: Returns formatted search results for search engines
- **JSON search support**: Uses `Nourayman\SearchJson\Searchable` for complex queries

### Route Model Binding

**(Backend)** SEO-friendly URL handling:

- **`getRouteKeyName()`**: Uses slug instead of ID for URLs
- **User-friendly URLs**: Creates readable links like `/articles/pregnancy-nutrition`

### View Tracking

**(Backend)** Automatic engagement analytics:

- **Auto-increment views**: Increases view counter when authenticated users read articles
- **Read tracking**: Automatically adds article to user's read history
- **Analytics support**: Provides data for content performance analysis

## Relationships

- **Doctor** (belongsTo): [Doctor model](./Doctor.md) - Expert who authored/reviewed the article
- **Categories** (morphToMany): [Category model](./Category.md) - Content categorization for organization
- **Tags** (morphToMany): Tag - Keyword tagging for search and filtering
- **Users** (belongsToMany): [User model](./User.md) - Users who have saved this article (via article_user pivot)
- **Favourite Users** (belongsToMany): [User model](./User.md) - Users who favorited this article (via favourite_articles pivot)

---