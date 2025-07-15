# System Workflows

This section documents the key business processes and workflows implemented in the Tahara platform.

## User Lifecycle Workflows

### User Registration Workflow
1. **Mobile Registration**
   - User provides mobile number and country code
   - System sends OTP via WhatsApp/SMS using Cequens service
   - User verifies OTP and completes registration
   - Role assignment: 'user' role automatically assigned

2. **Social Login Workflow**
   - Apple ID or Google authentication
   - Automatic account creation if new user
   - Email verification for Google users
   - Profile linking with existing data if applicable

### Profile Setup Workflow
1. **Initial Setup**
   - Age verification and birthdate entry
   - Goal selection (follow_cycle, be_pregnant, follow_pregnant)
   - Cycle parameters setup (day_numbers, repeat_period)
   - Latest cycle start date entry

2. **Age-Based Content Access**
   - Under 18: Restricted content access (sensetive_content = 0)
   - 18+: Full platform access
   - Automatic upgrade on 18th birthday via scheduled command

## Health Tracking Workflows

### Cycle Management Workflow
1. **Cycle Creation**
   - User starts new cycle with actual start date
   - System validates minimum 21-day gap between cycles
   - Automatic cycle day creation for each day of the cycle
   - Ovulation date calculation (cycle start + repeat_period - 14 days)

2. **Cycle Updates**
   - User can end cycle early with actual end date
   - System recalculates cycle days based on actual duration
   - Expected cycles generation for future predictions
   - Abnormal cycle flagging for durations exceeding normal ranges

3. **Symptom Tracking**
   - Daily symptom logging linked to cycle days
   - Age-appropriate symptom filtering
   - Goal-based symptom content (follow_cycle, be_pregnant, follow_pregnant)

### Pregnancy Workflow
1. **Pregnancy Confirmation**
   - User confirms pregnancy from last cycle
   - System validates last actual cycle eligibility
   - 42-week pregnancy period creation
   - Weekly pregnancy milestone generation

2. **Pregnancy Monitoring**
   - Weekly pregnancy data tracking
   - Baby development information display
   - Pregnancy-specific content and tips delivery
   - Goal change from 'be_pregnant' to 'follow_pregnant'

## Content Management Workflows

### Article Workflow
1. **Content Creation**
   - Doctor creates article with medical review
   - Goal-based categorization (follow_cycle, be_pregnant, follow_pregnant)
   - Age-appropriate content flagging
   - Translation management (Arabic/English)

2. **Content Distribution**
   - User interest-based filtering
   - Age-appropriate content serving
   - Goal-specific content recommendations
   - Reading progress tracking

### Educational Content Workflow
1. **Content Curation**
   - Doctor-reviewed medical articles
   - Video content integration
   - Health guide creation
   - Reference management

2. **AI Assistant Integration**
   - Content indexing for AI responses
   - User query processing
   - Personalized content recommendations
   - Medical advice boundary enforcement

## E-commerce Workflows

### Salla Integration Workflow
1. **Product Synchronization**
   - Real-time product updates via webhooks
   - Inventory management
   - Price and availability updates
   - Category management

2. **Order Processing**
   - User account linking with Salla
   - Cart management
   - Secure checkout process
   - Order status tracking
   - Discreet delivery coordination

### Customer Account Workflow
1. **Account Creation**
   - OTP verification for email
   - Salla account registration or linking
   - Profile synchronization
   - Address management

2. **Shopping Experience**
   - Privacy-focused product browsing
   - Wishlist management
   - Order history tracking
   - Customer support integration

## Notification Workflows

### Automated Notifications
1. **Cycle Notifications**
   - Period reminder notifications
   - Ovulation window alerts
   - Late cycle notifications
   - PMS symptom reminders

2. **Pregnancy Notifications**
   - Weekly pregnancy milestones
   - Appointment reminders
   - Health tip delivery
   - Baby development updates

3. **Educational Notifications**
   - Daily health tips
   - Article recommendations
   - Campaign announcements
   - Puzzle game notifications

## Administrative Workflows

### User Management Workflow
1. **Admin Operations**
   - User creation and role assignment
   - Profile management
   - Deletion request processing
   - Activity monitoring

2. **Content Moderation**
   - Article approval workflow
   - Doctor verification process
   - Content quality assurance
   - Age-appropriate content validation

### System Maintenance Workflows
1. **Scheduled Tasks**
   - Daily notification processing
   - Birthday upgrade automation
   - Account deletion after 30 days
   - Token refresh for external services

2. **Data Management**
   - Audit log maintenance
   - System performance monitoring
   - Backup procedures
   - Data privacy compliance

## Queue Processing Workflows

### Background Job Processing
1. **High Priority Queue**
   - Real-time notifications
   - Payment processing
   - Critical user updates

2. **Standard Queue**
   - Email notifications
   - Content synchronization
   - Report generation

3. **Low Priority Queue**
   - Analytics processing
   - Bulk data operations
   - System maintenance tasks

## Security Workflows

### Authentication & Authorization
1. **Access Control**
   - Role-based permission checking
   - API rate limiting
   - Token validation
   - Session management

2. **Data Protection**
   - Encryption at rest and transit
   - PII handling procedures
   - Audit trail maintenance
   - Privacy compliance validation

## Integration Workflows

### Third-Party Service Integration
1. **API Management**
   - Token refresh automation
   - Error handling and retry logic
   - Rate limit management
   - Webhook processing

2. **Data Synchronization**
   - Real-time updates from external systems
   - Conflict resolution procedures
   - Data consistency validation
   - Backup and recovery processes 