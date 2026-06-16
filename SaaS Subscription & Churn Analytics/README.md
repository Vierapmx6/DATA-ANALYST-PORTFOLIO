# 📊 RavenStack Churn Analytics

> **Data Analyst Portfolio — Project #2**
> Analyzing why 22% of SaaS customers churn, using SQL, Excel, and Power BI.

![Tools](https://img.shields.io/badge/Tools-SQL%20%7C%20Excel%20%7C%20Power%20BI-blue)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Dataset](https://img.shields.io/badge/Dataset-Kaggle-orange)

---

## 📌 Project Overview

This project analyzes customer churn data from **RavenStack**, a fictional SaaS company, covering **500 accounts** from **January 2023 to December 2024**.

The goal is to identify patterns in customer churn across 5 interconnected data tables — and translate those patterns into actionable business recommendations.

**Dataset source:** [SaaS Subscription & Churn Analytics — Kaggle (rivalytics)](https://www.kaggle.com/datasets/rivalytics/saas-subscription-and-churn-analytics-dataset/data)

---

## 🗂️ Data Structure

The dataset consists of 5 tables connected through `account_id`:

```
ravenstack_accounts          ← core customer info
    ├── ravenstack_subscriptions     (account_id → subscription_id)
    │       └── ravenstack_feature_usage   (subscription_id)
    ├── ravenstack_churn_events      (account_id)
    └── ravenstack_support_tickets   (account_id)
```

| Table | Key Columns | Rows |
|---|---|---|
| `ravenstack_accounts` | account_id, industry, plan_tier, churn_flag | 500 |
| `ravenstack_subscriptions` | subscription_id, mrr_amount, arr_amount, churn_flag | ~500+ |
| `ravenstack_churn_events` | churn_event_id, reason_code, refund_amount_usd | ~600 |
| `ravenstack_feature_usage` | usage_id, usage_count, error_count, is_beta_feature | ~2000 |
| `ravenstack_support_tickets` | ticket_id, satisfaction_score, resolution_time_hours | ~2000 |

---

## 🔍 Key Findings

| # | Finding | Insight |
|---|---|---|
| 1 | **DevTools** has the highest churn at **30.97%** | Nearly 2× Cybersecurity (16%) — a clear outlier |
| 2 | **Event-acquired** customers churn at **30.21%** | Partner channel is most loyal at only 14.61% |
| 3 | **Churned customers paid more** — avg MRR $2,426 vs $2,251 | Price is NOT the churn driver |
| 4 | Support quality showed **no difference** between churned vs active | Ruling out service as a cause |
| 5 | **Trial users** churn at 25.77% vs 21.09% non-trial | Onboarding gap is likely |

---

## 🧹 Data Cleaning Process

Before analysis, the following cleaning steps were performed:

**In Excel:**
- Converted `TRUE/FALSE` boolean columns → `1/0` using Find & Replace
- Replaced missing `satisfaction_flag` values (`"Missing"` → `0`, `"OK"` → `1`)
- Verified no NULL values existed in key columns using `COUNTBLANK()`

**In SQL (after import):**
- Date columns were stored as `VARCHAR` — converted to proper `DATE` type:
```sql
ALTER TABLE ravenstack_accounts
ADD signup_date_clean DATE;

UPDATE ravenstack_accounts
SET signup_date_clean = STR_TO_DATE(signup_date, '%m/%d/%Y');
```
- Applied same fix to `churn_date`, `start_date`, `end_date`, `submitted_at`, `closed_at`
- Changed `satisfaction_flag` from text to `INT`:
```sql
UPDATE ravenstack_support_tickets
SET satisfaction_flag = CASE
    WHEN satisfaction_flag = 'OK' THEN 1
    WHEN satisfaction_flag = 'Missing' THEN 0
END;

ALTER TABLE ravenstack_support_tickets
MODIFY satisfaction_flag INT;
```

---

## 📂 SQL Queries

### 1. Overall Churn Rate

```sql
SELECT
    COUNT(*)                                              AS total_accounts,
    SUM(churn_flag)                                       AS total_churned,
    ROUND(SUM(churn_flag) / COUNT(*) * 100, 2)           AS churn_rate_pct
FROM ravenstack_accounts;
```

**Result:** 500 accounts, 110 churned → **22% churn rate**

---

### 2. Churn Rate by Industry

```sql
SELECT
    industry,
    COUNT(*)                                              AS total,
    SUM(churn_flag)                                       AS churned,
    ROUND(SUM(churn_flag) / COUNT(*) * 100, 2)           AS churn_rate_pct
FROM ravenstack_accounts
GROUP BY industry
ORDER BY churn_rate_pct DESC;
```

| Industry | Total | Churned | Churn Rate |
|---|---|---|---|
| DevTools | 113 | 35 | **30.97%** |
| FinTech | 112 | 25 | 22.32% |
| HealthTech | 96 | 21 | 21.88% |
| EdTech | 79 | 13 | 16.46% |
| Cybersecurity | 100 | 16 | 16.00% |

---

### 3. Churn Rate by Referral Source

```sql
SELECT
    referral_source,
    COUNT(*)                                              AS total,
    SUM(churn_flag)                                       AS churned,
    ROUND(SUM(churn_flag) / COUNT(*) * 100, 2)           AS churn_rate_pct
FROM ravenstack_accounts
GROUP BY referral_source
ORDER BY churn_rate_pct DESC;
```

| Source | Churn Rate |
|---|---|
| Event | **30.21%** |
| Other | 24.27% |
| Ads | 23.47% |
| Organic | 17.54% |
| Partner | **14.61%** |

---

### 4. Average MRR — Churned vs Active

```sql
SELECT
    churn_flag,
    ROUND(AVG(mrr_amount), 2)                            AS avg_mrr,
    ROUND(AVG(arr_amount), 2)                            AS avg_arr
FROM ravenstack_subscriptions
GROUP BY churn_flag;
```

| Status | Avg MRR | Avg ARR |
|---|---|---|
| Active (0) | $2,250.69 | $27,008.26 |
| Churned (1) | **$2,426.21** | **$29,114.54** |

> 💡 Counter-intuitive finding: churned customers were paying *more* — ruling out price as the churn driver.

---

### 5. Support Quality vs Churn

```sql
SELECT
    a.churn_flag,
    ROUND(AVG(st.satisfaction_score), 2)                 AS avg_satisfaction,
    ROUND(AVG(st.resolution_time_hours), 2)              AS avg_resolution_hours,
    ROUND(AVG(st.first_response_time_minutes), 2)        AS avg_first_response_mins
FROM ravenstack_support_tickets st
JOIN ravenstack_accounts a ON st.account_id = a.account_id
GROUP BY a.churn_flag;
```

| Status | Avg Satisfaction | Resolution (hrs) | First Response (min) |
|---|---|---|---|
| Active | 3.97 | 35.92 | 89.5 |
| Churned | 4.01 | 35.66 | 84.78 |

> 💡 Churned customers actually received *slightly better* support — support quality is not the churn driver.

---

### 6. Feature Usage vs Churn

```sql
SELECT
    a.churn_flag,
    ROUND(AVG(fu.usage_count), 2)                        AS avg_usage_count,
    ROUND(AVG(fu.usage_duration_secs), 2)                AS avg_duration_secs,
    ROUND(AVG(fu.error_count), 2)                        AS avg_errors
FROM ravenstack_feature_usage fu
JOIN ravenstack_subscriptions s  ON fu.subscription_id = s.subscription_id
JOIN ravenstack_accounts a       ON s.account_id = a.account_id
GROUP BY a.churn_flag;
```

| Status | Avg Usage | Avg Duration (secs) | Avg Errors |
|---|---|---|---|
| Active | 10.02 | 3,042.12 | 0.57 |
| Churned | 10.03 | 3,042.49 | 0.54 |

> 💡 Nearly identical — feature usage and errors are not churn drivers either.

---

### 7. Top Churn Reasons (Overall vs DevTools)

```sql
-- Overall
SELECT
    reason_code,
    COUNT(*)                                              AS total,
    ROUND(COUNT(*) / SUM(COUNT(*)) OVER() * 100, 2)      AS pct
FROM ravenstack_churn_events
GROUP BY reason_code
ORDER BY total DESC;

-- DevTools only
SELECT
    ce.reason_code,
    COUNT(*)                                              AS total
FROM ravenstack_churn_events ce
JOIN ravenstack_accounts a ON ce.account_id = a.account_id
WHERE a.industry = 'DevTools'
GROUP BY ce.reason_code
ORDER BY total DESC;
```

| Reason (Overall) | Count | % | Reason (DevTools) | Count |
|---|---|---|---|---|
| Features | 114 | 19% | Support | 27 |
| Support | 104 | 17.33% | Budget | 27 |
| Budget | 104 | 17.33% | Features | 26 |
| Unknown | 95 | 15.83% | Competitor | 25 |
| Competitor | 92 | 15.33% | Pricing | 22 |

> 💡 In DevTools, all 5 reasons are nearly equal — no single dominant driver. This suggests a broader product-market fit issue.

---

### 8. Trial vs Non-Trial Churn

```sql
SELECT
    is_trial,
    COUNT(*)                                              AS total,
    SUM(churn_flag)                                       AS churned,
    ROUND(SUM(churn_flag) / COUNT(*) * 100, 2)           AS churn_rate_pct
FROM ravenstack_accounts
GROUP BY is_trial;
```

| Is Trial | Churn Rate |
|---|---|
| Non-Trial (0) | 21.09% |
| Trial (1) | **25.77%** |

---

## 📊 Dashboard

Built in Power BI — connected directly from MySQL via native connector.

![Dashboard Preview](./Ravenstack-Assets.png)

**Visuals included:**
- 4 KPI cards (Total Accounts, Churned, Churn Rate %, Avg MRR Churned)
- Churn rate by industry (bar chart)
- Churn rate by referral source (bar chart)
- Top churn reasons (table)
- Churn by plan tier (table)
- Trial vs non-trial churn (bar chart)
- Slicer filters: Industry, Plan Tier, Referral Source

---

## 💡 Recommendations

| # | Recommendation | Based On |
|---|---|---|
| 1 | **Investigate DevTools churn** — conduct qualitative interviews | DevTools at 30.97%, no single dominant reason |
| 2 | **Shift budget to Partner channel** — 2× better retention than Events | Partner 14.61% vs Event 30.21% |
| 3 | **Improve trial onboarding** — structured setup flow, early value moments | Trial churn 4.68pp higher than non-trial |
| 4 | **Protect high-MRR accounts** — proactive success management for $2k+ MRR | Churned customers paid $175/mo more |

---

## 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| **Excel** | Initial data inspection, boolean cleaning, date formatting |
| **MySQL + DBeaver** | Data import, type fixing, exploratory analysis |
| **Power BI** | Dashboard and visualization |

---

## 📁 Repository Structure

```
ravenstack-churn-analytics/
├── README.md
├── sql/
│   ├── 01_data_cleaning.sql       ← ALTER TABLE, STR_TO_DATE, type fixes
│   ├── 02_exploratory_analysis.sql ← churn rate, industry, referral queries
│   └── 03_deep_dive.sql            ← support quality, feature usage, JOIN queries
├── assets/
│   └── dashboard.png              ← Power BI dashboard screenshot
└── presentation/
    └── RavenStack_Churn_Analytics.pptx
```

---

## 🔗 About This Portfolio

This is **Project #2** of my Data Analyst portfolio, focusing on:
- Multi-table SQL analysis with JOINs and window functions
- Data cleaning across Excel and SQL
- Translating data findings into business recommendations
- Dashboard storytelling in Power BI

---

*Dataset: RavenStack SaaS Subscription & Churn Analytics — Kaggle (rivalytics)*
