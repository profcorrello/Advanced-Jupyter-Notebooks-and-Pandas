# Advanced Jupyter Notebooks and Pandas

CIS 186 (*Introduction to Python*) companion materials for running and analyzing time-series data in **Jupyter Notebooks inside VS Code**, using **Pandas** and **Seaborn**. The project pairs a hands-on tutorial notebook with a laboratory reference cheatsheet and a bundled benchmark dataset.

## Project Contents

| File | Description |
| --- | --- |
| `CIS186_Time_Series_Tutorial.ipynb` | Main tutorial notebook (see below) |
| `cis186-time-series-cheatsheet.md` | Condensed syntax reference: VS Code shortcuts, Kaggle ingestion, time-series Pandas, resampling vs. rolling windows, Seaborn charting, and troubleshooting |
| `opsd_germany_daily.csv` | Daily German energy dataset (2006–2017): `Date`, `Consumption`, `Wind`, `Solar`, `Wind+Solar` in GWh |
| `pyproject.toml` / `uv.lock` | uv-managed Python project (Python ≥ 3.12; dependencies: `pandas>=3.1.1`, `seaborn>=0.13.2`) |
| `main.py` | Minimal placeholder entry point |

## Tutorial Notebook Overview

`CIS186_Time_Series_Tutorial.ipynb` is a companion to the deck `CIS186_VSCode_Jupyter_Tutorial.pptx`. It walks students through a complete time-series workflow:

1. **Library imports & styling** — `pandas`, `numpy`, `matplotlib`, `seaborn` with a configured Seaborn `whitegrid` theme.
2. **Data ingestion** — Kaggle ingestion pattern via `kagglehub` (documented in markdown) plus a classroom fallback that downloads or synthesizes `opsd_germany_daily.csv` if missing.
3. **Time-aware DataFrames** — `pd.read_csv(..., index_col=0, parse_dates=True)` to build a `DatetimeIndex` in one step.
4. **Calendar feature engineering** — extracting `Year`, `Month`, `Weekday_Name`, and `Is_Weekend` from the index without Python loops.
5. **Time-based slicing** — partial-string lookups (`opsd.loc['2012-02']`) and inclusive range slices (`opsd.loc['2016-12-24':'2017-01-02']`).
6. **Seasonality visualization** — `sns.boxplot` distributions for monthly (annual) seasonality and weekday-vs-weekend weekly seasonality.
7. **Resampling vs. rolling windows** — weekly/monthly `.resample()` downsampling compared against a 7-day centered `.rolling()` mean, with row-count inspection and an overlaid plot.
8. **Multi-year trends** — a 365-day rolling mean that averages out seasonality to isolate long-term consumption trends.
9. **Multi-series line plots** — reshaping wide columns to tidy long format with `pd.melt()`, then plotting Consumption/Wind/Solar with `sns.lineplot(hue=...)`.
10. **Student lab exercises** — three challenges (date slicing, annual renewable-share resampling, and an `sns.barplot` visualization) with a commented solution scratchpad.

## Getting Started

Requires [uv](https://docs.astral.sh/uv/) (or any Python ≥ 3.12 environment with the listed dependencies).

```bash
uv sync
uv run jupyter lab
```

Then open `CIS186_Time_Series_Tutorial.ipynb`, select the project kernel, and run cells with `Shift + Enter`. On first run, the notebook generates `opsd_germany_daily.csv` if it is not already present; delete the CSV to regenerate it.

### VS Code Tips (from the notebook)

- **Execute cell:** `Shift + Enter` (run and advance) or `Ctrl + Enter` (run in place)
- **Command mode:** `Esc` — `A` insert above, `B` insert below, `D, D` delete
- **Edit mode:** `Enter` to edit code or Markdown
- **Variable Explorer:** click **Variables** in the notebook toolbar to inspect DataFrames live
