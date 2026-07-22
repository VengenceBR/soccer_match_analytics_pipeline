# European Soccer Match Analytics

An end-to-end data pipeline and exploratory analysis of the [European Soccer Database](https://www.kaggle.com/datasets/hugomathien/soccer) (25,979 matches, 11,060 players, 299 teams, 11 leagues, 2008–2016) — from raw SQLite to a clean, model-ready dataset, plus a full set of trend, time-series, and performance visualizations.

## What this project does

**Data pipeline**
- Acquires data programmatically from Kaggle via the `kagglehub` API — no manual downloads
- Extracts every table from the source SQLite database to CSV and loads it into pandas
- Cleans each table with a consistent, documented imputation strategy (median for numeric columns, mode for categorical, drop for high-missingness columns)
- Engineers match outcome labels (`home_win`, `away_win`, `draw`, `goal_difference`)
- Parses embedded match-event XML to extract possession percentage, shots on/off target, fouls, and cards per match
- Consolidates everything into a single match-level dataset, joined with team-level build-up speed and defensive pressure attributes

**Exploratory analysis & visualization**
- Goal distribution analysis (overall, and home vs. away)
- Monthly trends in scoring and home-win rate
- Time series of scoring across the full 2008–2016 timeline
- Team-level time series of cumulative goals for the most active teams
- League-by-league ("tournament type") comparison of average goals per match
- Top 10 teams by wins and top 10 players by average rating
- Box plots comparing goal distributions across leagues
- IQR-based outlier detection for unusually high-scoring matches

**Match outcome prediction**
- Leakage-free, pre-match feature set: team playing style (`buildUpPlaySpeed`, `chanceCreationPassing`, `defencePressure`) as of the most recent snapshot before each match, via time-aware `merge_asof` joins
- Rolling team form (win rate, avg. goals scored/conceded over each team's last 5 matches, computed with a shifted rolling window so no match ever sees its own outcome)
- Head-to-head win rate between the two specific teams, from prior meetings only
- Starting-XI player quality, using each player's most recent `overall_rating` before the match
- A short literature review of what's worked in published soccer-outcome-prediction research, used to justify which model families to test rather than defaulting to one
- Cross-validated screening (time-series folds) across Logistic Regression, Random Forest, and Gradient Boosting on the training set only, before touching the test set
- Time-based train/test split (train on earlier seasons, test on later ones) — no random shuffling, since matches are sequential
- Full held-out evaluation of all three candidates against a naive "always predict Home Win" baseline, using accuracy, macro F1, confusion matrices, and feature importances/coefficients for the best performer

## Repository structure

```
.
├── soccer_match_analytics_pipeline.ipynb   # Main notebook — the full pipeline, step by step
├── requirements.txt
├── .gitignore
├── LICENSE
└── data/
    └── processed/
        └── soccer_match_master.csv   # generated on first run
```

## Getting started

1. Clone the repo and install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
2. Set up a Kaggle API token (`~/.kaggle/kaggle.json`) — see the [Kaggle API docs](https://github.com/Kaggle/kaggle-api#api-credentials) if you don't have one yet.
3. Open `soccer_match_analytics_pipeline.ipynb` and run all cells top to bottom. The notebook downloads the dataset automatically on first run.

## Notebook contents

| # | Section | What it covers |
|---|---|---|
| 0 | Setup | Imports and global plotting config |
| 1 | Data Acquisition (Kaggle API) | Programmatic dataset download via `kagglehub` |
| 2 | Extracting SQLite Tables to CSV | Dumps every table in the source database to CSV |
| 3 | Loading Tables into DataFrames | Loads all CSVs into a single `dataframes` dict |
| 4 | Data Quality Assessment & Cleaning | Missing-value handling per table (Player, Player_Attributes, Team, Team_Attributes, Match, Country, League) |
| 5 | Match Outcome Labels | `goal_difference`, `home_win`, `away_win`, `draw` |
| 6 | XML Feature Extraction — Possession & Shots | Parses embedded match-event XML into numeric features |
| 7 | XML Feature Extraction — Fouls & Cards | Parses discipline events, correlates with outcomes |
| 8 | Consolidated Master Dataset | Joins SQL-derived team attributes with engineered features |
| 10 | Goal Distribution | Histograms of total, home, and away goals |
| 11 | Monthly Trends Analysis | Goals and home-win rate by calendar month |
| 12 | Time Series Visualizations | Season-long scoring trend, team-level cumulative goals, league comparison |
| 13 | Performance Visualizations | Top teams/players, box plots by league, outlier detection |
| — | **Part II: Match Outcome Prediction** | |
| 15 | Modeling Setup | Additional imports for feature engineering and modeling |
| 16 | Target Variable | Builds `match_result` (Home Win / Draw / Away Win) |
| 17 | Pre-Match Team Attributes | Time-aware `merge_asof` join for team style, no leakage |
| 18 | Rolling Team Form | Shifted rolling win rate and goals scored/conceded (last 5 matches) |
| 19 | Head-to-Head History | Home team's win rate vs. this opponent, prior meetings only |
| 20 | Starting-XI Player Quality | Pre-match `overall_rating` averaged across each starting XI |
| 21 | Assembling the Modeling Dataset | Combines all features, attaches league/stage, imputes gaps |
| 22 | Time-Based Train/Test Split | Train on earlier seasons, test on later ones |
| 23 | Model Selection Research (R&D) | Literature review of what's worked in published soccer-prediction studies |
| 24 | Cross-Validated Candidate Screening | Time-series CV comparison of Logistic Regression, Random Forest, Gradient Boosting on training data only |
| 25 | Baseline | Naive "always predict Home Win" reference score |
| 26 | Training the Final Models | Fits all three screened candidates on the full training set |
| 27 | Model Evaluation | Accuracy, macro F1, confusion matrices, and feature importances for the best-performing model |
| 28 | Summary & Next Steps | Recap and suggested modeling directions |

## Output

The final artifact is `data/processed/soccer_match_master.csv` — one row per match, combining:

| Category | Columns |
|---|---|
| Match context | date, country, league, home/away team names |
| Goals | home_team_goal, away_team_goal, goal_difference, total_goals |
| Outcome labels | home_win, away_win, draw |
| Team style | home/away build-up speed, home/away defensive pressure |
| Match events | home_possession, total_shots_on, total_shots_off, home/away_fouls, home/away_cards |

## Next steps

- Train a baseline classifier (logistic regression) to predict match outcome
- Add rolling team-form features (recent win rate, goals scored/conceded)
- Incorporate aggregated player ratings from `Player_Attributes`
- Use a time-respecting train/test split, since matches are sequential

## License

MIT — see [LICENSE](LICENSE).
