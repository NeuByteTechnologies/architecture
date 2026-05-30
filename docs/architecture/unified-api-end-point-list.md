# Unified API Endpoint List (All Modules)
(Grouped by functional domain)

## Authentication
| Method | Endpoint | Purpose |
| --- | --- | --- |
| POST | /auth/login | Authenticate user and issue session token |
| POST | /auth/logout | Invalidate current session |
| GET | /auth/session | Validate active session and return user context |


## User
| Method | Endpoint | Purpose |
| --- | --- | --- |
| POST | /user | Create new user |
| GET | /user | Get user profile |
| PUT | /user | Update user profile |


## Dashboard
| Method | Endpoint | Purpose |
| --- | --- | --- |
| GET | /dashboard | Get full dashboard composite summary |


## Metadata
| Method | Endpoint | Purpose |
| --- | --- | --- |
| GET | /metadata | Get all metadata types |
| GET | /metadata/{type} | Get a single metadata type |


## MFA Enrollment
| Method | Endpoint | Purpose |
| --- | --- | --- |
| POST | /mfa/enroll | Begin MFA enrollment |
| POST | /mfa/verify | Verify MFA code |
| POST | /mfa/reenroll | Reset MFA and generate new secret |
| POST | /mfa/disable | Disable MFA |


## Exercise Programs
| Method | Endpoint | Purpose |
| --- | --- | --- |
| GET | /programs | List available programs |
| GET | /programs/{program_id} | Get program detail |
| POST | /programs/{program_id}/start | Start a program |
| POST | /programs/{program_id}/complete | Complete a program |
| GET | /programs/active | Get user’s active program |


##  Help / Support
| Method | Endpoint | Purpose |
| --- | --- | --- |
| GET | /help/categories | Get help categories |
| GET | /help/articles | Get help articles |
| GET | /help/articles/{article_id} | Get help article detail |


##  Notification Center
| Method | Endpoint | Purpose |
| --- | --- | --- |
| GET | /notifications | Get notifications (with filters) |
| GET | /notifications/unread_count | Get unread count |
| POST | /notifications/{notification_id}/read | Mark notification as read |
| POST | /notifications/read_all | Mark all notifications as read |
| DELETE | /notifications/{notification_id} | Delete notification |


## Notification Preferences
| Method | Endpoint | Purpose |
| --- | --- | --- |
| GET | /notification_preferences | Get user notification preferences |
| PUT | /notification_preferences | Update notification preferences |
| GET | /notification_preferences/defaults | Get default notification preferences |


## Preferences
| Method | Endpoint | Purpose |
| --- | --- | --- |
| GET | /preferences | Get all user preferences |
| PUT | /preferences | Update user preferences |
| GET | /preferences/defaults | Get default preferences |


## Weight Tracking
| Method | Endpoint | Purpose |
| --- | --- | --- |
| GET | /preferences | Get all user preferences |
| PUT | /preferences | Update user preferences |
| GET | /preferences/defaults | Get default preferences |


## Workout Logging
Method	Endpoint	Purpose
POST	/workout/start	Start workout
GET	/workout/{workout_log_id}	Get workout log detail
POST	/workout/{workout_log_id}/set	Add logged set
PUT	/workout/set/{set_id}	Edit logged set
DELETE	/workout/set/{set_id}	Delete logged set
POST	/workout/{workout_log_id}/complete	Complete workout
POST	/workout/{workout_log_id}/abandon	Abandon workout
GET	/workout/active	Get active workout