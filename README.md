# 🧑‍💻 Behind the Green Squares
### Detecting Fake & Bot-Like Developer Profiles Using Python

> **34.08% of analyzed GitHub profiles show strong signs of automated, bot-like, or manufactured activity — and README quality turns out to be the single strongest predictor of authenticity (correlation: -0.74).**

---

## 📌 Overview

This project analyzes **101,807 GitHub developer profiles** to expose patterns
of inauthentic activity — manufactured star counts, automated commit
patterns, mass-following behavior, and fabricated activity streaks.

Using only **Pandas, Matplotlib, and Seaborn**, the project cleans a
deliberately messy 105,000-row dataset (21 columns, 72,000+ missing/dirty
values), engineers 9 new features, and answers 20 business questions before
visualizing the findings across 18 charts spanning 8 different chart types.

---

## 🔍 Key Findings

| Finding | Number |
|---|---|
| Profiles flagged suspicious | **34,699 / 101,807 (34.08%)** |
| README score — genuine vs suspicious | **6.29/10 vs 2.28/10** |
| Correlation: README score ↔ suspicion | **-0.74** (strongest predictor) |
| "Fake popularity" profiles (1000+ stars, <10 forks) | **2,668 (2.6%)** — 99.5% flagged suspicious |
| Profiles committing during 12am–4am | **0.01% of genuine vs 66.09% of suspicious** |
| Mass-follow profiles (follow ratio < 0.05) | **28,080 (27.6%)** — 98.4% flagged suspicious |
| Follow ratio — genuine vs suspicious | **13.53 vs 2.52** |
| High commit-frequency profiles flagged suspicious | **5.9x more often** than low-frequency profiles |
| Suspicion rate by account age | **Flat at ~34%** — account age does NOT predict authenticity |
| Highest suspicion language (500+ profiles) | **Rust — 34.7%** |

---

## 🛠️ Tech Stack

`Python` · `Pandas` · `NumPy` · `Matplotlib` · `Seaborn` · `Jupyter Notebook`

---

## 📁 Repository Structure

```
Behind-the-Green-Squares
│
├── data/
│   ├── github_illusion_dirty.csv      ← raw dataset (105,000 rows, 21 cols, dirty)
│   ├── github_illusion_clean.csv      ← cleaned dataset (101,806 rows, 27 cols)
│   └── github_illusion_analysis.csv   ← cleaned + feature-engineered (101,806 rows, 31 cols)
│
├── notebooks/
│   ├── 01_data_exploration.ipynb      ← Phase 1: initial EDA on raw data
│   ├── 02_data_cleaning.ipynb         ← Phase 2: cleaning + feature engineering
│   ├── 03_business_questions.ipynb    ← Phase 2b: 20 pandas business questions
│   └── 04_visualizations.ipynb        ← Phase 3: 18 charts
│
├── charts/                            ← all 18 exported chart images
│
├── METHODOLOGY.md                     ← full step-by-step process writeup
└── README.md
```

---

## 🧹 Data Cleaning Highlights

The raw dataset was intentionally messy. Key issues fixed:

- **3,193 duplicate usernames** removed
- **5 inconsistent date formats** standardized into proper datetimes
- `commit_streak` stored as text (`"145 days"`) → converted to integer
- `bot_like_score` mixed types (`1`, `"yes"`, `True`, `"0"`, `"no"`) → standardized to `0`/`1`
- `avg_commit_hour` outliers (`-3`, `99`, `100`) → corrected to valid 0-23 range
- `star_fork_ratio` `inf` values (zero forks) → capped at the 99th percentile
- `language` and `country` columns → standardized casing & naming (`python`/`PYTHON`/`Python` → `Python`, `IN`/`india`/`IND` → `India`)
- All remaining missing values handled via median/mode imputation

---

## ⚙️ Feature Engineering

| New Feature | Description |
|---|---|
| `account_age_years` | Account age in years |
| `commit_frequency` | Commits per year |
| `follow_ratio` | followers ÷ (following + 1) |
| `repo_commit_ratio` | repos ÷ (commits + 1) |
| `suspicious_hrs` | Flag for commits made between 11pm-4am |
| `suspicion_score` | Weighted composite score (0-1) of 7 red-flag signals |
| `is_suspicious` | Binary label (`suspicion_score >= 0.5`) |
| `suspicion_tier` | 4-level trust tier (Clean / Borderline / Likely Bot / Confirmed Bot) |
| `age_bucket` / `freq_level` | Categorical bins for analysis |

---

## 📊 Sample Visualizations

### Stars vs Forks — The Fake Popularity Detector
![Stars vs Forks](charts/04_stars_vs_forks.png)

High stars + near-zero forks = manufactured popularity. 2,668 profiles fall
into this trap, and 99.5% of them are independently flagged as suspicious.

### README Quality: The Strongest Signal
![Correlation Heatmap](charts/14_correlation_heatmap.png)

`suspicion_score` correlates most strongly with `readme_score` (-0.74) —
bots can fake stars, streaks, and follower counts, but they can't fake good
documentation.

### Suspicion Tier Breakdown
![Suspicion Tier Pie](charts/02_suspicion_tier_pie.png)

➡️ See [`charts/`](charts/) for all 18 visualizations, and
[`METHODOLOGY.md`](METHODOLOGY.md) for the full step-by-step process.

---

## 🚀 How to Run

```bash
git clone https://github.com/<your-username>/github-illusion.git
cd github-illusion
pip install pandas numpy matplotlib seaborn jupyter

jupyter notebook notebooks/01_data_exploration.ipynb
```

---

## 🎯 Conclusion

Roughly **1 in 3 GitHub profiles** in this dataset show clear signs of
inauthentic, automated activity — inflated stars with no real engagement,
mass-following with almost no followers back, commit activity concentrated
in inhuman hours, and suspiciously round commit streaks. The single most
reliable signal of authenticity isn't stars, followers, or account age — it's
the **quality of a developer's README**.

---

*Built as a portfolio data analysis project using Python, Pandas, Matplotlib,
and Seaborn.*
