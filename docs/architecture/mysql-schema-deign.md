# MySQL Schema Design Document
*FitnessApp — Logical & Physical Database Architecture*

## 1. Overview
The FitnessApp database supports all core application functions, including:

- User identity & authentication
- MFA
- Contact information
- Preferences & units
- Exercise programs
- Workout logging
- Notifications
- Help/FAQ
- Application events & error logging

This document describes the **physical MySQL 8 schema**, including:

- Table definitions
- Column data types
- Primary keys
- Foreign keys
- Default values
- Storage engine decisions
- Naming conventions
- Rationale for key design choices
- This is the implementation blueprint, not the build script.

## 2. Technology Stack
| Component | Choice | Rationale |
| --- | --- | --- |
| **Database Engine** | MySQL 8.0 (InnoDB) | ACID compliance, FK support, row‑level locking |
| **Character Set** | utf8mb4 | Full Unicode support |
| **Collation** | utf8mb4_unicode_ci | Case‑insensitive, stable sorting |
| **Storage Engine** | InnoDB | Required for FK constraints |
| **Timestamps** | ``DATETIME(6)`` | Microsecond precision, MySQL‑native |
| **Booleans** | ``TINYINT(1)`` | MySQL standard |


## 3. Naming Conventions
**Tables**
- snake_case
- singular nouns (e.g., user, exercise_program_item)
- lookup tables end in _type or _category

**Primary Keys**
- Always single‑column
- Always numeric
- Always named table_name_id

**Foreign Keys**
- Named FK_<child>_<parent>
- Always reference the parent table’s PK

**Indexes**
- PKs created automatically
- Additional indexes added based on API workload (future section)

## 4. Schema Summary
This section lists each table with a short description.  

| Table | Purpose |
| --- | --- |
| **user** | Core identity record |
| **user_contact** | Email + phone info |
| **user_type** | Lookup for user roles |
| **user_mfa** | MFA secret + status |
| **user_preference** | User‑specific settings |
| **preference** | Preference definitions |
| **unit_type** | Units of measure |
| **exercise** | Exercise definitions |
| **exercise_program** | Workout program templates |
| **exercise_program_item** | Program structure (weeks/days/sets) |
| **workout_log** | User workout sessions |
| **workout_log_set** | Logged sets |
| **workout_log_interval** | Logged intervals |
| **workout_status** | Lookup for workout state |
| **notification** | User notifications |
| **notification_type** | Notification categories |
| **notification_delivery_type** | Delivery channels |
| **notification_event** | Events that trigger notifications |
| **user_notification_preference** | User overrides |
| **help_category_type** | FAQ categories |
| **help_frequently_asked_question** | FAQ entries |
| **app_event** | Application telemetry |
| **event_type** | Event categories |
| **error_log** | Error tracking |
| **error_severity** | Severity levels |
| **email_type** | Email categories |
| **phone_type** | Phone categories |
| **user_weight** | Weight tracking |


## 5. Table Specifications (Physical Model)
Below is the level of detail expected for each table.
I’ll show **three examples**, then you can confirm if you want the full document expanded for all tables.  

### 5.1 user
**Purpose**: Stores core identity information for each FitnessApp user.

**Columns**
| Column | Type | Null | Default | Description |
| --- | --- | --- | --- | --- |
| user_id | INT | NO | AUTO_INCREMENT | Primary key |
| user_type_id | TINYINT | NO | — | FK to user_type |
| first_name | VARCHAR(100) | NO | — | User’s first name |
| last_name | VARCHAR(100) | NO | — | User’s last name |
| hash_pass | BINARY(256) | NO | — | Password hash |
| last_update | DATETIME(6) | NO | — | Last profile update |


**Constraints**
- **PK_user_id** (user_id)
- **FK_user_user_type** → user_type(user_type_id)

**Notes**
- Password stored as fixed‑length binary for hashing consistency
- No email/phone stored here — normalized into user_contact

### 5.2 workout_log
**Purpose**: Represents a single workout session for a user.

**Columns**
| Column | Type | Null | Default | Description |
| --- | --- | --- | --- | --- |
| workout_log_id | BIGINT | NO | AUTO_INCREMENT | Primary key |
| user_id | INT | NO | — | FK to user |
| exercise_program_id | INT | YES | — | FK to exercise_program |
| workout_start_time | DATETIME(6) | NO | — | Start timestamp |
| workout_end_time | DATETIME(6) | YES | — | End timestamp |
| workout_status_id | TINYINT | NO | — | FK to workout_status |
| workout_note | VARCHAR(2000) | YES | — | Optional notes |


**Constraints**
- **PK_workout_log** (workout_log_id)
- **FK_workout_log_user** → user(user_id)
- **FK_exercise_program** → exercise_program(exercise_program_id)
- **FK_workout_status** → workout_status(workout_status_id)

**Notes**
- Designed to support both structured and free‑form workouts
- End time may be null for in‑progress sessions

### 5.3 notification
**Purpose**: Stores notifications delivered to users.

**Columns**
| Column | Type | Null | Default | Description |
| --- | --- | --- | --- | --- |
| notification_id | BIGINT | NO | AUTO_INCREMENT | Primary key |
| user_id | INT | NO | — | FK to user |
| notification_type_id | TINYINT | NO | — | FK to notification_type |
| notification_delivery_type_id | TINYINT | NO | — | FK to notification_delivery_type |
| notification_title | VARCHAR(200) | NO | — | Title |
| notification_message | VARCHAR(2000) | NO | — | Body text |
| created_at | DATETIME(6) | NO | CURRENT_TIMESTAMP(6) | Creation time |
| delivered_at | DATETIME(6) | YES | — | Delivery timestamp |
| read_at | DATETIME(6) | YES | — | Read timestamp |
| is_read | TINYINT(1) | NO | 0 | Read flag |
| is_archived | TINYINT(1) | NO | 0 | Archive flag |


**Constraints**
- **PK_notification_id** (notification_id)
- **FK_notification_user** → user(user_id)
- **FK_notification_type_id** → notification_type(notification_type_id)
- **FK_notification_delivery_type** → notification_delivery_type(notification_delivery_type_id)

**Notes**
- Designed for both push and in‑app notifications
- Flags allow efficient filtering for unread messages

## 6. Relationship Diagram (Textual Summary)
This section describes the ERD in words.  

**User Domain**
- user → user_type (many‑to‑one)
- user → user_contact (one‑to‑many)
- user → user_mfa (one‑to‑one)
- user → user_preference (one‑to‑many)
- user → user_weight (one‑to‑many)
- user → workout_log (one‑to‑many)
- user → notification (one‑to‑many)
- user → error_log (one‑to‑many)

**Workout Domain**
- exercise_program → exercise_program_item (one‑to‑many)
- exercise → exercise_program_item (one‑to‑many)
- workout_log → workout_log_set (one‑to‑many)
- workout_log → workout_log_interval (one‑to‑many)

**Notification Domain**
- notification_type → notification_event (one‑to‑many)
- notification_type → notification (one‑to‑many)
- notification_delivery_type → notification (one‑to‑many)
- user → user_notification_preference (one‑to‑many)

**Help/FAQ Domain**
- help_category_type → help_frequently_asked_question (one‑to‑many)

**System Logging**
- event_type → app_event (one‑to‑many)
- error_severity → error_log (one‑to‑many)

## 7. Design Rationale
This section explains **why** the schema is designed this way.

### 7.1 Single‑column numeric PKs
- Fast joins
- Small index footprint
- MySQL‑friendly clustering

### 7.2 Normalized contact & preference data
- Supports multiple emails/phones
- Supports extensible preference system

### 7.3 Separation of workout_log, workout_log_set, workout_log_interval
- Supports strength, cardio, and interval training
- Flexible for future workout types

### 7.4 Notification system decoupled from events
- Allows multiple delivery channels
- Allows user overrides
- Supports future automation

### 7.5 Logging tables isolated
- Prevents noise in transactional tables
- Supports analytics and monitoring

##  8. Future Indexing Strategy
(To be completed after API query definitions.)  

Will include:  

- Query workload
- Recommended indexes
- Covering indexes
- Composite index ordering
- Justification

## 9. Appendix: Full MySQL DDL