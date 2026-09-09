# ConnectaTel — Customer Behavior & Usage Segmentation

## 🔍 Overview

Python/pandas exploratory analysis of customer behavior for ConnectaTel, a telecommunications company operating in Latin America. Cleaned and merged three datasets (plans, users, usage) covering 4,000 customers and 40,000 usage records through 2024, built a per-user consumption profile, flagged and resolved data-quality issues, and segmented customers by usage intensity and age to surface retention and upsell opportunities.

[![View Repository Files](https://img.shields.io/badge/📂_View_Repository_Files-100000?style=for-the-badge&logo=github&logoColor=white)](https://github.com/maxsantana-data2strategy/connectatel-customer-behavior-analysis)
[![Download Infographic PDF](https://img.shields.io/badge/📥_Download_Infographic_PDF-2EA44F?style=for-the-badge&logo=adobeacrobatreader&logoColor=white)](https://raw.githubusercontent.com/maxsantana-data2strategy/connectatel-customer-behavior-analysis/main/assets/Infographic_ConnectaTel_EN.pdf)

## 🎯 Problem Statement

**Business Question:** Which customers drive the most value for ConnectaTel, and how should the company adjust its plans and retention strategy to match actual usage behavior?

**Sub-questions:**

- "Which users display outliers that might indicate unusual behavior, potential fraud, or logging errors?"

- "How does usage vary according to age and the type of subscribed plan?"

- "What patterns can help design better plans, optimize the current offerings, and improve overall customer satisfaction?

The business needed to understand who its heaviest users are, whether the current Basic/Premium plan structure fits real consumption patterns, and where "power users" — customers who push past typical usage — represent an upsell opportunity rather than noise to be cleaned away.

---

## 💡 What I Did

### Phase 1: Data Loading & Structure Review
- Loaded three datasets: `plans.csv` (2 plans), `users.csv` (4,000 customers), `usage.csv` (40,000 call/text records)
- Reviewed shape, dtypes, and non-null counts for each table with `.info()` and `.shape`

### Phase 2: Data Quality Diagnosis
- **Nulls:** `city` (11.7%) and `churn_date` (88.4%) missing in `users`; `duration` (55.2%) and `length` (44.7%) missing in `usage`
- **Sentinels:** `age` contained an impossible value of **-999**; `city` contained a literal `'?'` placeholder
- **Impossible dates:** 40 registration records (1%) were dated **2026**, outside the dataset's stated 2024 cutoff
- **MAR verification:** Confirmed `duration`/`length` nulls are Missing At Random — they depend entirely on `type` (a text has no call duration, a call has no message length), so they were kept rather than imputed or dropped

### Phase 3: Cleaning
- Replaced the `-999` age sentinel with the column median
- Replaced `'?'` city values with `pd.NA`
- Converted `reg_date` and `date` to proper datetime, marking the 2026 records as `pd.NaT`
- Dropped/flagged the 50 null `usage['date']` rows (0.125%, negligible)

### Phase 4: Feature Engineering & Profile Build
- Aggregated `usage` by `user_id` into `cant_mensajes` (messages), `cant_llamadas` (calls), and `cant_minutos_llamada` (total call minutes)
- Merged the aggregation into a single `user_profile` table alongside plan, age, and city

### Phase 5: Distribution & Outlier Analysis
- Plotted histograms of age, messages, calls, and call minutes split by plan (`Basic` vs. `Premium`)
- Used boxplots, the **IQR method**, and **Z-scores (|Z| > 3)** to identify outliers in usage variables
- **Decision: kept the outliers.** They represent legitimate high-engagement "power users," not data errors — removing them would hide the exact segment most likely to trigger overage fees or plan upgrades

### Phase 6: Customer Segmentation
- **Usage segments:** `Low use` (calls < 5 and messages < 5), `Medium use` (calls < 10 and messages < 10), `High use` (remaining)
- **Age segments:** `Young adult` (< 30), `Adult` (30–59), `Senior adult` (≥ 60)
- Visualized both segmentations against plan type with count plots

---

## 🛠️ Technologies Used

| Category | Tools |
|----------|-------|
| **Python** | pandas, numpy |
| **Visualization** | seaborn, matplotlib |
| **Techniques** | Null/MAR diagnosis, sentinel detection, datetime standardization, IQR & Z-score outlier detection, groupby aggregation, rule-based segmentation |
| **Environment** | Google Colab / Jupyter Notebook |

---

## 📊 Key Findings

### Data Quality Issues Resolved

| Column | Issue | Scope | Resolution |
|---|---|---|---|
| `users['age']` | Sentinel value -999 | 0.025% of rows | Replaced with median age |
| `users['city']` | `'?'` placeholder | 2.4% of rows | Replaced with `pd.NA` |
| `users['reg_date']` | Impossible year 2026 | 1% of rows (40) | Replaced with `pd.NaT` |
| `usage['date']` | Null timestamps | 0.125% of rows (50) | Replaced with `pd.NaT` |
| `usage['duration']` / `usage['length']` | High nulls (~55% / ~45%) | Confirmed MAR by `type` | Kept as-is (meaningful, not missing) |

### Customer Segments

- **Age:** `Adult` is the largest segment, followed by `Senior adult` and `Young adult`. `Basico` leads in every age group, most notably among `Adults`.
- **Usage:** `Medium use` is the largest segment, followed by `Low use` and `High use`. `Premium` has a meaningfully larger share within `Medium` and `High use` than within `Low use`.
- **Distributions:** Messages, calls, and call minutes are all **right-skewed** across both plans — most customers cluster at modest usage, with a long tail of heavy users. `Premium` consistently shows the wider spread and longer tail.

### 🖼️ Visualization

<p align="center">
<img src="https://raw.githubusercontent.com/maxsantana-data2strategy/connectatel-customer-behavior-analysis/main/assets/usage_and_age_segments_by_plan.png" alt="Customers by usage segment and age segment, split by plan (Basico vs Premium)" width="800">
</p>

`Basico` outnumbers `Premium` in every usage and age segment, including the 278 `High use` customers (185 on `Basico`) — the clearest signal that heavy users are under-monetized on a plan not built for their consumption.

### Outliers Are the Opportunity, Not the Noise

| Metric | IQR upper bound | Z-score outliers (\|Z\| > 3) | Max observed |
|---|---|---|---|
| Messages sent | 11.50 | 21 users | high-volume "power texters" |
| Calls made | 10.50 | 30 users | high-volume callers |
| Call minutes | 61.86 min | 47 users | up to 155 minutes |

### Context → Findings → Implications (C→F→I)

**📍 Context:**
ConnectaTel serves 4,000 customers across two plans (`Basico`, `Premium`) in Latin America, with usage data spanning calls and text messages recorded through 2024.

**🔍 Findings:**
1. **Usage is heavily right-skewed:** most customers are light-to-moderate users, but a consistent minority — 21 to 47 users depending on the metric — drive disproportionate consumption.
2. **`Medium use` is the center of gravity:** the largest usage segment, and where `Premium` already has its strongest foothold outside of `High use`.
3. **`Basic plan` dominates every segment**, including among the heaviest users — a signal that current heavy users may be under-monetized on a plan not built for their consumption.
4. **Missingness is informative, not accidental:** the MAR pattern in `duration`/`length` reflects genuine call-vs-text structure, not a data collection failure — the retention risk signal, `churn_date`, is the null column that actually matters for follow-up.

**💡 Implications:**
- 🎯 ** Extremely high users are an upsell target, not an outlier to clean away.** The 21–47 heaviest users per metric are prime candidates for a higher-tier plan.
- 🔄 **`Medium use` is the highest-leverage segment to move.** It's already the largest group and already has real `Premium` penetration — targeted incentives here have the shortest path to conversion.
- 📊 **Re-examine the `Basic` plan's ceiling.** Its dominance even among `High use` customers suggests plan limits and messaging aren't differentiating usage tiers effectively.
- 🩺 **Prioritize `churn_date` follow-up.** With 88.4% of customers active (no churn date), a deeper look at the 11.6% who did churn is the next highest-value analysis.

---

```
├── ConnectaTel_Customer_Behavior_Analysis.ipynb # Full analysis notebook
├── plans.csv                                    # Plan catalog (2 plans)
├── users.csv                                    # Customer records (4,000 rows)
├── usage.csv                                    # Call/text usage log (40,000 rows)
├── assets/                                      # Infographic & chart images
└── README.md                                    # This file
```

---

## 🚀 How to Use

> ✅ **Raw Data Included:** The original datasets (`plans.csv`, `users.csv`, `usage.csv`) are included in this repository.

1. **Explore the notebook:** Open `ConnectaTel_Customer_Behavior_Analysis.ipynb` in Google Colab or Jupyter to review the full cleaning, EDA, and segmentation workflow.
2. **Reproduce the analysis:** Run the notebook against the included `plans.csv`, `users.csv`, and `usage.csv`, or point the data-loading cell at your own CSVs with an equivalent schema (plan info, customer info, and call/text usage records).

---

## 📚 Learnings & Best Practices

- **Not all missing data should be treated the same way:** distinguishing MAR nulls (`duration`/`length`, driven by `type`) from true gaps (`churn_date`, `city`) prevented both over-imputation and the loss of a meaningful retention signal.
- **Outliers can be the most valuable rows in the dataset:** IQR and Z-score flagged the same power users from two different angles — the right call was to keep them, since they define ConnectaTel's highest-value segment.
- **Sentinel values hide in plain sight:** a `-999` age and a `'?'` city both passed a naive `.head()` check; only `.describe()` and `.value_counts()` surfaced them.
- **Segmentation is only useful when it's actionable:** usage and age segments were deliberately built with simple, explainable thresholds so the business can act on them directly.

---

**Status:** ✅ Complete | **Data Quality:** ✅ Validated | **Ready for Decisions:** ✅ Yes
