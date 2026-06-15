# 📋 METHODOLOGY — The GitHub Illusion
## Full Step-by-Step Process

This document explains the complete workflow used to go from a raw, dirty
dataset to a finished portfolio project with 18 visualizations and 20
answered business questions.

---

## Phase 1 — Data Exploration

**Goal:** Understand the raw dataset before touching anything.

Steps performed:
- Loaded `github_illusion_dirty.csv` (105,000 rows, 21 columns)
- Checked shape, data types, and summary statistics (`.shape`, `.dtypes`,
  `.describe()`)
- Counted missing values per column (`.isnull().sum()`) — found 72,000+
  missing values across multiple columns
- Identified specific dirty columns:
  - `account_created` / `last_commit_date` — multiple date formats
  - `commit_streak` — stored as text (`"145 days"`) instead of numbers
  - `avg_commit_hour` — out-of-range values (`-3`, `99`, `100`)
  - `language` — inconsistent casing (`python`, `Python`, `PYTHON`)
  - `country` — inconsistent naming (`IN`, `india`, `IND`, `India`)
  - `bot_like_score` — mixed types (`1`, `"yes"`, `True`, `"0"`)
  - `star_fork_ratio` — contained `inf` values where forks = 0
  - `suspicion_score` — 30% missing (needed to be engineered)
- Checked for duplicates: 3,193 duplicate usernames found

---

## Phase 2 — Data Cleaning

**Goal:** Fix every issue identified in Phase 1.

| Step | Action |
|---|---|
| 1 | Removed 3,193 duplicate usernames (`drop_duplicates`) |
| 2 | Converted `commit_streak` from text to numeric (stripped `" days"`, `pd.to_numeric`) |
| 3 | Standardized `bot_like_score` to clean `0`/`1` integers |
| 4 | Parsed both date columns using `format='mixed'`, converting invalid strings (`"N/A"`, `"never"`) to `NaN` first |
| 5 | Fixed `avg_commit_hour` outliers (values outside 0-23) by setting to `NaN` and filling with the median |
| 6 | Fixed `profile_age_days` outliers (negative or >6000 days) the same way |
| 7 | Recomputed `star_fork_ratio` and capped `inf` values at the 99th percentile (avoiding lost signal from zero-fork profiles) |
| 8 | Standardized `language` (lowercase → replace nulls/unknowns → title case) |
| 9 | Standardized `country` using a mapping dictionary covering all variant spellings/codes |
| 10 | Filled remaining missing values: `stars`/`forks` → 0, `followers`/`following`/`readme_score`/`total_repos`/`total_commits`/`commit_streak` → median, `has_bio`/`has_profile_pic` → 0 |

**Result:** 101,807 clean rows, 0 unexpected missing values in core columns.

---

## Phase 2b — Feature Engineering

New columns created to enable deeper analysis:

```python
account_age_years  = (today - account_created) in years
commit_frequency   = total_commits / account_age_years
follow_ratio       = followers / (following + 1)
repo_commit_ratio  = total_repos / (total_commits + 1)
suspicious_hrs     = 1 if avg_commit_hour <= 4 or >= 23, else 0
```

A **suspicion score** (0-1) was engineered for the 30% of rows where it was
missing, using a weighted combination of 7 red-flag signals:

| Signal | Weight |
|---|---|
| `star_fork_ratio` > 50 | +0.25 |
| `follow_ratio` < 0.05 | +0.20 |
| `readme_score` < 3 | +0.20 |
| `suspicious_hrs` == 1 | +0.15 |
| `commit_streak` is a suspiciously round number (100/200/300/365/500) | +0.10 |
| `has_bio` == 0 | +0.05 |
| `has_profile_pic` == 0 | +0.05 |

Profiles with `suspicion_score >= 0.5` were labeled `is_suspicious = 1`.

Additional categorical features were created for grouping:
- `suspicion_tier` — Clean / Borderline / Likely Bot / Confirmed Bot
- `age_bucket` — <1yr, 1-2yr, 2-4yr, 4-6yr, 6-8yr, 8yr+
- `freq_level` — Low / Medium / High commit frequency (based on tertiles)

---

## Phase 2c — Answering Business Questions

20 questions were answered directly using Pandas, ranging from basic counts
to multi-condition filters, pivot tables, and custom functions. Examples:

- What % of all profiles are flagged suspicious? **34.08%**
- How many profiles have 1000+ stars but <10 forks? **2,668 (2.6%)** — 99.5%
  of these are flagged suspicious
- What's the average README score for genuine vs suspicious profiles?
  **6.29 vs 2.28**
- What % of profiles commit during suspicious hours (12am-4am)?
  **0.01% genuine vs 66.09% suspicious**
- Which language has the highest suspicion rate (500+ profiles)?
  **Rust at 34.7%**
- Does account age predict authenticity? **No — suspicion rate stays flat
  (~34%) across all account ages**
- Are high-frequency committers more suspicious? **Yes — 5.9x more likely**
  than low-frequency committers

Pandas patterns used: `value_counts`, `groupby + agg`, boolean filtering,
`.isin()`, `pd.cut()`, `pd.crosstab()`, `pd.pivot_table()`, custom functions
with `.apply()`.

---

## Phase 3 — Visualization

18 charts were created across 8 different chart types using Matplotlib and
Seaborn:

| Chart Type | Charts |
|---|---|
| Bar | Genuine vs Suspicious counts, README score, follow ratio, suspicion by language/country/age/frequency |
| Pie | Suspicion tier breakdown |
| Scatter | Stars vs Forks, Followers vs Following |
| Histogram | README score distribution, commit streak distribution |
| Box plot | README score by suspicion tier |
| Violin | README score distribution by tier |
| Strip | Follow ratio (sampled) |
| Swarm | README score (sampled) |
| Heatmap | Feature correlation matrix |
| Reg plot | README score vs suspicion score (with trend line) |

Each chart is saved as a standalone PNG in [`charts/`](charts/) and embedded
in the main [`README.md`](README.md).

---

## Final Outcome

A reproducible, end-to-end analysis pipeline that takes a 105,000-row dirty
CSV and produces:
- A clean, feature-engineered dataset (101,807 rows, 30 columns)
- 20 answered business questions with quantified findings
- 18 charts across 8 chart types
- A single headline finding: **README quality (not stars, followers, or
  account age) is the strongest predictor of profile authenticity**
  (correlation: -0.74)
