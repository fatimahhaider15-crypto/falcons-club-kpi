# Falcons Sports Club: Sign-up Funnel KPIs

This repository is the **source of truth** for how we measure the Falcons Sports Club sign-up funnel. It defines the key metrics (KPIs) and the data structure behind them. All future analysis and dashboards must follow these definitions.

- **Client:** Falcons Sports Club
- **Role:** BI Analyst
- **Scope:** Definitions and structure only (no code yet)
- **Related file:** [`data_dictionary.md`](./data_dictionary.md) describes the `users` and `events` tables

---

## Funnel Overview

A person becomes an active paying member by passing through 5 stages:

| Stage | Name     | What it means                                      | Event that marks it        |
|-------|----------|----------------------------------------------------|----------------------------|
| 1     | Visit    | Person lands on the Falcons Club website           | `page_view`                |
| 2     | Signup   | Person creates an account                          | `signup_completed`         |
| 3     | Verify   | Person confirms their email or phone               | `email_verified`           |
| 4     | Payment  | Person pays for a membership plan                  | `payment_completed`        |
| 5     | Activate | Person uses the club facility for the first time   | `first_checkin`            |

```
Visit  ->  Signup  ->  Verify  ->  Payment  ->  Activate
```

---

## KPI Definitions

All KPIs are calculated for a chosen **reporting period** (e.g. a week or a month). Count each person **once** per stage, even if they repeat an action.

### KPI 1: Visitor-to-Signup Rate

- **Formula:** `(Number of users who completed signup / Number of unique visitors) x 100`
- **Funnel step:** Visit -> Signup
- **Why it matters:** It shows whether the website and offers are convincing people to create an account. A low rate points to problems with marketing, the landing page, or the sign-up form. It is the first and widest leak in the funnel.

### KPI 2: Signup-to-Verification Rate

- **Formula:** `(Number of users who verified their email/phone / Number of users who completed signup) x 100`
- **Funnel step:** Signup -> Verify
- **Why it matters:** It shows how many sign-ups are real and committed. A low rate can mean verification emails are not arriving, the process is confusing, or many sign-ups use fake contact details. Unverified users cannot be followed up, so this protects the quality of the lead list.

### KPI 3: Verified-to-Paid Rate

- **Formula:** `(Number of users who completed a payment / Number of verified users) x 100`
- **Funnel step:** Verify -> Payment
- **Why it matters:** This is the step where interest turns into revenue. A low rate can point to pricing issues, a complicated payment page, or missing payment methods. It is the most important KPI for business income.

### KPI 4: Payment-to-Activation Rate

- **Formula:** `(Number of paid members who made their first check-in / Number of paid members) x 100`
- **Funnel step:** Payment -> Activate
- **Why it matters:** A member who pays but never visits is likely to cancel soon. This rate shows whether new members actually start using the club, and it is an early warning for churn and refund requests.

### KPI 5: Overall Visitor-to-Paid Conversion Rate

- **Formula:** `(Number of users who completed a payment / Number of unique visitors) x 100`
- **Funnel step:** Visit -> Payment (the full funnel)
- **Why it matters:** This single number is the health score of the whole sign-up funnel. It is the "Conversion Rate" the club's management should track over time. When it moves, KPIs 1 to 3 show which stage caused the change.

---

## KPI Summary

| # | KPI                                | Numerator            | Denominator          |
|---|------------------------------------|----------------------|----------------------|
| 1 | Visitor-to-Signup Rate             | Signups              | Unique visitors      |
| 2 | Signup-to-Verification Rate        | Verified users       | Signups              |
| 3 | Verified-to-Paid Rate              | Paid users           | Verified users       |
| 4 | Payment-to-Activation Rate         | Activated members    | Paid members         |
| 5 | Overall Visitor-to-Paid Conversion | Paid users           | Unique visitors      |

**Drop-off rate (derived):** for any stage, `Drop-off Rate = 100% - Stage Conversion Rate`. For example, if the Signup-to-Verification Rate is 70%, the drop-off at that stage is 30%.

---

## Counting Rules

1. **Unique counts:** a person or visitor is counted once per stage per period.
2. **Visitors** are identified by `session_id` (they may not have an account yet). Users are identified by `user_id`.
3. **Cohort view:** a user is attributed to the period in which they signed up.
4. **Test and internal accounts** must be excluded from all KPIs.
5. **Time zone:** all timestamps are stored and reported in UTC.

---

## Repository Contents

| File                 | Purpose                                                      |
|----------------------|--------------------------------------------------------------|
| `README.md`          | KPI definitions and business context (this file)             |
| `data_dictionary.md` | Column-level description of the `users` and `events` tables  |
