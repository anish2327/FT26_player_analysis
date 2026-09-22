# FC 26 Player Scouting & Valuation Analysis

An interactive player-valuation and scouting analysis over **18,405 FC 26 players**. The project uses DuckDB and SQL for analytical queries, scikit-learn for KMeans clustering and linear regression, Plotly for interactive visualizations, and Jupyter widgets for player search and scouting filters.

## Problem statement

A football recruitment team wants to understand what drives player market value, identify natural player archetypes from their attributes, compare position and league economics, and find players whose market values appear high or low relative to a simple regression model.

## Dataset

- **Source**: FC 26 player database export (SoFIFA database snapshot, dated 2025-09-19)
- **Domain**: sports player valuation & scouting
- **Size**: 18,405 players, 110 columns
- **Core fields**: player name, age, club, league, position, overall, potential, market value, wage, and detailed player attributes

### Data quality notes

- Goalkeepers do not have the six core outfield attributes (`pace`, `shooting`, `passing`, `dribbling`, `defending`, `physic`) because they use goalkeeper-specific `gk_*` attributes instead. Outfield-only analyses therefore filter on non-null `pace`.
- `player_positions` is used for position analysis because it contains a player's listed playing positions. A player listed as `RW, ST, CF` is counted in each of those positions.
- `club_position` is not used for position analysis because it contains many `SUB` and `RES` entries rather than actual playing positions.

## Tech stack

| Layer | Tool |
|---|---|
| Storage / analysis | DuckDB + SQL |
| Data processing | Python, pandas, NumPy |
| Clustering / modeling | scikit-learn (KMeans, LinearRegression) |
| Visualization | Plotly |
| Interactive notebook controls | Jupyter + ipywidgets |

## How to run

```bash
python -m venv .venv

# Windows
.venv\Scripts\activate

pip install duckdb pandas numpy scikit-learn plotly jupyter ipywidgets
```

Place the dataset at:

```text
data/players_fc26.csv
```

Then launch the notebook:

```bash
jupyter notebook analysis_interactive_fc26.ipynb
```

> **VS Code users:** open the notebook with the Jupyter extension and make sure the selected Python interpreter is the same environment where the packages above were installed.

## Project structure

```text
fc26-player-analysis/
│
├── data/
│   └── players_fc26.csv
│
├── analysis_interactive_fc26.ipynb
├── queries.sql
├── db.py
├── README.md
└── requirements.txt        # optional
```

The notebook is the main analysis and visualization interface, so a separate Streamlit application is not required.

## Architecture

The project follows a SQL-first approach:

```text
FC 26 CSV
   │
   ▼
DuckDB
   │
   ├── SQL analytics
   │     ├── attribute correlations
   │     ├── position economics
   │     ├── league wage analysis
   │     └── age curves
   │
   └── pandas / scikit-learn
         ├── KMeans player archetypes
         └── Linear Regression value model
                  │
                  ▼
          Interactive Jupyter Notebook
                  ├── Plotly charts
                  ├── Player search
                  └── Scouting filters
```

- [`queries.sql`](./queries.sql) contains the SQL queries used throughout the analysis.
- [`db.py`](./db.py) loads the FC 26 CSV into DuckDB and exposes a small `run_query()` helper.
- [`analysis_interactive_fc26.ipynb`](./analysis_interactive_fc26.ipynb) contains the complete analysis, modeling, interactive visualizations, player search, and scouting explorer.

## Analysis walkthrough

### 1. Dataset overview

Basic coverage, player counts, rating levels, and dataset structure.

### 2. Attribute relationships with market value

Pearson correlations are calculated in DuckDB to examine how player attributes, overall rating, potential, and age relate to market value.

### 3. Position economics

Players are grouped by their listed positions to compare average market value and value per overall-rating point.

### 4. League wage economics

Leagues are compared using average wage per overall-rating point, with a minimum player-count threshold to avoid very small samples dominating the results.

### 5. Career arc

Interactive age curves show how average overall, potential, and market value change across player age groups.

### 6. Player archetypes with KMeans

Six core outfield attributes are standardized before KMeans clustering. An elbow curve helps evaluate the number of clusters, and the resulting groups can be explored through interactive scatter plots.

### 7. Market-value prediction with linear regression

The model predicts `log(value_eur)` using:

- overall
- potential
- age
- pace
- shooting
- passing
- dribbling
- defending
- physic

The notebook uses a proper **80/20 train/test split** and reports R² and MAE on the held-out test set.

### 8. Regression residuals for scouting

The fitted model is used to estimate player value for the full eligible player pool. Large residuals are surfaced as potential **under-valued** or **over-valued** scouting candidates.

These should be treated as scouting signals rather than definitive transfer valuations.

### 9. Interactive player search

The notebook includes a searchable player selector with an individual scouting profile showing key player information, actual value, regression-predicted value, and an interactive attribute radar.

### 10. Interactive scouting explorer

Use notebook controls to filter the player pool by position, minimum potential, age range, and maximum market value to build a recruitment shortlist.

## Key findings

1. **Overall rating and potential** are the strongest single-attribute relationships with market value in the dataset.
2. **Passing and dribbling** show stronger relationships with market value than defending, indicating a stronger market association with creative and attacking attributes.
3. **CAM** has the highest average market value and value-per-overall-point among the analyzed positions.
4. The **Premier League** has substantially higher wage per overall-rating point than La Liga and Serie A in the filtered league analysis.
5. KMeans produces recognizable player-profile clusters based only on standardized attributes, without using position labels as an input.
6. The linear regression provides an interpretable baseline for estimating expected market value and highlighting players whose actual values differ substantially from model estimates.

## Model limitations

The regression model is a **scouting signal, not a transfer-price truth**. It uses player attributes, age, overall, and potential, but does not capture the complete context behind a real-world transfer valuation, including contract situation, club finances, reputation, injuries, demand, or transfer-market dynamics.

The KMeans clusters are also descriptive rather than definitive player roles. Cluster labels should be interpreted from the attribute profiles rather than treated as official positions.

## Skills demonstrated

- Analytical SQL with DuckDB
- Correlation analysis across multiple player attributes
- Exploding multi-position fields with `UNNEST` and `string_split`
- Aggregation and filtering with `GROUP BY`, `HAVING`, and parameterized queries
- Feature scaling and KMeans clustering
- Linear regression on a log-transformed target
- Train/test evaluation using R² and MAE
- Residual analysis for scouting signals
- Interactive Plotly visualization
- Interactive player search and filtering with Jupyter widgets
- Communicating model limitations clearly

## Interactive notebook

The notebook is designed to be explored rather than simply read. Plotly charts support hover, zoom, pan, and legend controls, while the scouting sections provide searchable and filterable player exploration directly inside Jupyter.

## Why this project matters

Football analytics projects are a useful way to demonstrate the full path from raw data to an analytical product: SQL-based exploration, statistical analysis, unsupervised learning, interpretable regression, and an interactive interface for exploring the results.
