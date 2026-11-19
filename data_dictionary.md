# Data Dictionary: Falcons Sports Club

This document describes every column of the two tables used to calculate the KPIs defined in [`README.md`](./README.md).

- **`users`**: one row per registered person (their current status in the funnel)
- **`events`**: one row per action a visitor or user performs (the full activity log)

Conventions: all timestamps are in **UTC**; column names use `snake_case`; `NULL` means the value does not exist yet.

---

## Table 1: `users`

**Grain:** one row per user account. **Primary key:** `user_id`.

| Column            | Type           | Nullable | Description                                                                 | Example Value               |
|-------------------|----------------|----------|-----------------------------------------------------------------------------|-----------------------------|
| `user_id`         | INT            | No       | Unique ID of the user. Primary key.                                         | `1001`                      |
| `full_name`       | VARCHAR(100)   | No       | User's full name as entered at signup.                                      | `Ali Khan`                  |
| `email`           | VARCHAR(255)   | No       | User's email address. Must be unique.                                       | `ali.khan@example.com`      |
| `phone_number`    | VARCHAR(20)    | Yes      | Mobile number including country code.                                       | `+923001234567`             |
| `signup_date`     | TIMESTAMP      | No       | Date and time the account was created (funnel stage 2).                     | `2026-11-02 09:15:30`       |
| `is_verified`     | BOOLEAN        | No       | Whether the user confirmed email or phone. Default `FALSE`.                 | `TRUE`                      |
| `verified_at`     | TIMESTAMP      | Yes      | Date and time of verification (stage 3). `NULL` if not verified.            | `2026-11-02 09:20:11`       |
| `membership_plan` | VARCHAR(30)    | Yes      | Plan the user paid for. `NULL` if no payment yet. Values: `monthly`, `quarterly`, `yearly`. | `monthly` |
| `payment_date`    | TIMESTAMP      | Yes      | Date and time of the first successful payment (stage 4). `NULL` if unpaid.  | `2026-11-03 14:05:00`       |
| `payment_amount`  | DECIMAL(10,2)  | Yes      | Amount paid in PKR for the first payment.                                   | `5000.00`                   |
| `activated_at`    | TIMESTAMP      | Yes      | Date and time of the first facility check-in (stage 5). `NULL` if never.    | `2026-11-05 18:30:00`       |
| `traffic_source`  | VARCHAR(50)    | Yes      | Marketing channel that brought the user. Values: `organic`, `facebook`, `instagram`, `google_ads`, `referral`. | `instagram` |
| `is_test_account` | BOOLEAN        | No       | `TRUE` for internal or test accounts. Excluded from all KPIs. Default `FALSE`. | `FALSE`                  |

**Funnel stage columns:** `signup_date` (stage 2), `verified_at` (stage 3), `payment_date` (stage 4), `activated_at` (stage 5). A user is at a given stage when the related column is not `NULL`.

### Sample rows

| user_id | full_name  | email                  | signup_date         | is_verified | verified_at         | membership_plan | payment_date        | payment_amount | activated_at        | traffic_source |
|---------|------------|------------------------|---------------------|-------------|---------------------|-----------------|---------------------|----------------|---------------------|----------------|
| 1001    | Ali Khan   | ali.khan@example.com   | 2026-11-02 09:15:30 | TRUE        | 2026-11-02 09:20:11 | monthly         | 2026-11-03 14:05:00 | 5000.00        | 2026-11-05 18:30:00 | instagram      |
| 1002    | Sara Ahmed | sara.a@example.com     | 2026-11-02 11:40:02 | TRUE        | 2026-11-02 11:45:50 | NULL            | NULL                | NULL           | NULL                | google_ads     |
| 1003    | Bilal Raza | bilal.r@example.com    | 2026-11-03 16:22:45 | FALSE       | NULL                | NULL            | NULL                | NULL           | NULL                | organic        |

---

## Table 2: `events`

**Grain:** one row per action. **Primary key:** `event_id`. **Foreign key:** `user_id` references `users.user_id`.

| Column            | Type          | Nullable | Description                                                                                  | Example Value                   |
|-------------------|---------------|----------|----------------------------------------------------------------------------------------------|---------------------------------|
| `event_id`        | BIGINT        | No       | Unique ID of the event. Primary key.                                                         | `900045`                        |
| `session_id`      | VARCHAR(64)   | No       | ID of the browsing session. Used to count unique visitors who have no account yet.           | `sess_8f3a21c9`                 |
| `user_id`         | INT           | Yes      | Links the event to a user. `NULL` for anonymous visitors before signup.                      | `1001`                          |
| `event_type`      | VARCHAR(50)   | No       | The action performed. Allowed values listed below.                                           | `signup_completed`              |
| `event_timestamp` | TIMESTAMP     | No       | Date and time the event happened.                                                            | `2026-11-02 09:15:30`           |
| `page_url`        | VARCHAR(500)  | Yes      | Page where the event happened (web events only).                                             | `/signup`                       |
| `device_type`     | VARCHAR(20)   | Yes      | Device used. Values: `mobile`, `desktop`, `tablet`.                                          | `mobile`                        |
| `traffic_source`  | VARCHAR(50)   | Yes      | Channel of the session. Same values as in `users`.                                           | `instagram`                     |
| `event_value`     | DECIMAL(10,2) | Yes      | Money value of the event. Used only for `payment_completed`.                                 | `5000.00`                       |

### Allowed values for `event_type`

| event_type          | Funnel stage | Meaning                                      |
|---------------------|--------------|----------------------------------------------|
| `page_view`         | 1. Visit     | A visitor opened a page of the website       |
| `signup_started`    | 2. Signup    | The visitor opened the sign-up form          |
| `signup_completed`  | 2. Signup    | The account was created                      |
| `email_verified`    | 3. Verify    | The user confirmed email or phone            |
| `payment_started`   | 4. Payment   | The user opened the payment page             |
| `payment_completed` | 4. Payment   | Payment was successful                       |
| `first_checkin`     | 5. Activate  | The member used the facility for the first time |

### Sample rows

| event_id | session_id    | user_id | event_type        | event_timestamp     | page_url   | device_type | traffic_source | event_value |
|----------|---------------|---------|-------------------|---------------------|------------|-------------|----------------|-------------|
| 900001   | sess_8f3a21c9 | NULL    | page_view         | 2026-11-02 09:10:05 | /home      | mobile      | instagram      | NULL        |
| 900002   | sess_8f3a21c9 | NULL    | signup_started    | 2026-11-02 09:13:40 | /signup    | mobile      | instagram      | NULL        |
| 900003   | sess_8f3a21c9 | 1001    | signup_completed  | 2026-11-02 09:15:30 | /signup    | mobile      | instagram      | NULL        |
| 900004   | sess_8f3a21c9 | 1001    | email_verified    | 2026-11-02 09:20:11 | /verify    | mobile      | instagram      | NULL        |
| 900010   | sess_b21d77e0 | 1001    | payment_completed | 2026-11-03 14:05:00 | /checkout  | desktop     | NULL           | 5000.00     |
| 900021   | sess_c90e14aa | 1001    | first_checkin     | 2026-11-05 18:30:00 | NULL       | NULL        | NULL           | NULL        |

---

## How the Tables Support the KPIs

| KPI                                | Numerator source                                  | Denominator source                                 |
|------------------------------------|---------------------------------------------------|----------------------------------------------------|
| 1. Visitor-to-Signup Rate          | `users.signup_date` is not `NULL`                 | Distinct `events.session_id` with `page_view`      |
| 2. Signup-to-Verification Rate     | `users.verified_at` is not `NULL`                 | `users.signup_date` is not `NULL`                  |
| 3. Verified-to-Paid Rate           | `users.payment_date` is not `NULL`                | `users.verified_at` is not `NULL`                  |
| 4. Payment-to-Activation Rate      | `users.activated_at` is not `NULL`                | `users.payment_date` is not `NULL`                 |
| 5. Overall Visitor-to-Paid Rate    | `users.payment_date` is not `NULL`                | Distinct `events.session_id` with `page_view`      |

Always filter out rows where `users.is_test_account = TRUE`.

## Relationship Between Tables

```
users (1) ----< events (many)
users.user_id = events.user_id
```

One user can have many events. Events from anonymous visitors have `user_id = NULL`.
