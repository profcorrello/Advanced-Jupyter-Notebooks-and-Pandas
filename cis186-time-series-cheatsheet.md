# CIS 186 Laboratory Reference: Jupyter Notebooks, Time-Series Pandas, and Seaborn in VS Code

This laboratory reference document provides a consolidated, practical syntax guide for introductory computer science students working with time-series data pipelines in the Visual Studio Code ecosystem. It synthesizes keyboard navigation, data ingestion via Kaggle, chronological manipulation in Pandas, and statistical charting with Seaborn into an accessible bench reference.

---

## 1. VS Code Jupyter Environment & Keyboard Navigation

The Jupyter extension in Visual Studio Code separates the front-end user interface from computational execution. Code execution is delegated to an underlying IPython kernel process via ZeroMQ protocols, allowing variables and imported modules to persist in memory across discrete notebook cells.

To work efficiently, developers toggle between two primary interface modes:

* **Command Mode (Blue/Grey Margin):** Active when pressing `Escape`. Keystrokes manipulate cells and notebook layout rather than editing text.
* **Edit Mode (Green Margin):** Active when pressing `Enter` or clicking inside a cell editor. Keystrokes modify the code or Markdown source.

### Cell Execution & Navigation Shortcuts

The following table summarizes essential keyboard shortcuts across both operating modes in the VS Code notebook interface.

| Keystroke / Action | Active Mode | Operational Effect |
| :--- | :--- | :--- |
| `Shift + Enter` | Both | Executes the focused cell and advances cursor to the next cell (creates a new cell if at end). |
| `Ctrl + Enter` (`Cmd + Enter`) | Both | Executes the focused cell in place without advancing cursor position. |
| `Alt + Enter` (`Option + Enter`) | Both | Executes the focused cell and immediately inserts a blank code cell below. |
| `A` | Command Mode | Inserts a new code cell **Above** the currently focused cell. |
| `B` | Command Mode | Inserts a new code cell **Below** the currently focused cell. |
| `D, D` (press twice) | Command Mode | Deletes the currently focused cell. |
| `M` | Command Mode | Converts the focused cell into a formatted **Markdown** narrative cell. |
| `Y` | Command Mode | Converts the focused cell into an executable **Python Code** cell. |
| `Z` | Command Mode | Undoes the last cell-level structural operation (e.g., cell deletion). |
| `Ctrl + Shift + P` | Global | Opens the VS Code Command Palette to select interpreters, restart kernels, or clear output. |

### Kernel Management and Variable Inspection

Visual Studio Code provides integrated development utilities that replace temporary print statements during iterative analysis:

* **Kernel Selector:** Located in the top right corner of the notebook editor. Click to switch between isolated Python virtual environments (`.venv`), Conda environments, or system installations.
* **Variable Explorer:** Accessed via the **Variables** button on the notebook toolbar. It lists all active variables, data types, shapes, and memory consumption currently held by the IPython kernel.
* **Data Viewer:** Clicking the table icon beside any Pandas DataFrame in the Variable Explorer launches a spreadsheet-like tab for sorting, column filtering, and inspecting records.
* **Kernel Restarting:** When code enters an infinite loop or memory becomes cluttered, click **Restart** on the toolbar to reset kernel state, followed by **Run All** to verify reproducibility from top to bottom.

---

## 2. Dataset Acquisition & Kaggle Pipeline

Automated time-series data pipelines retrieve assets directly into the project structure. Kaggle authentication requires placing an API credential file on the local machine before executing download scripts.

### Authentication Configuration

1. Log into Kaggle, navigate to **Account Settings**, and select **Create New Token** to download `kaggle.json`.
2. Place the token into the standard user home path:
   * **Linux/macOS:** `~/.kaggle/kaggle.json`
   * **Windows:** `C:\Users\<Username>\.kaggle\kaggle.json`
3. Secure credentials on Unix-based systems via the terminal:
   ```bash
   chmod 600 ~/.kaggle/kaggle.json
   ```

### Pythonic Ingestion via `kagglehub`

The modern `kagglehub` library automates downloading and local caching of dataset versions:

```python
import kagglehub
import pandas as pd

# Download latest dataset files to local cache directory
dataset_path = kagglehub.dataset_download("camnugent/sandp500")
print("Files downloaded to:", dataset_path)

# Ingest target CSV into Pandas
df = pd.read_csv(f"{dataset_path}/all_stocks_5yr.csv")
```

### Direct Terminal Retrieval via Kaggle CLI

For scripted workspace automation, execute the command-line interface directly:

```bash
# Download and unzip dataset archive into a local project folder
kaggle datasets download -d camnugent/sandp500 -p ./data --unzip
```

---

## 3. Pandas Time-Series Data Structures & Manipulation

Time-series analysis in Pandas requires converting textual timestamp strings into specialized 64-bit nanosecond datetime objects (`datetime64[ns]`) and indexing the DataFrame along this temporal axis.

### Temporal Ingestion & Inspection

When loading CSV data, set `parse_dates` to identify timestamp columns, and set `index_col` to establish a `DatetimeIndex`.

```python
import pandas as pd

# Ingest and convert Date column to DatetimeIndex in a single call
df = pd.read_csv(
    "energy_data.csv",
    parse_dates=["Date"],
    index_col="Date"
)

# Alternative manual conversion if already loaded
df["Date"] = pd.to_datetime(df["Date"], format="%Y-%m-%d")
df.set_index("Date", inplace=True)

# Verify temporal data types and chronological range
print(type(df.index))       # <class 'pandas.core.indexes.datetimes.DatetimeIndex'>
print(df.index.dtype)      # datetime64[ns]
print(df.info())
```

### Calendar Feature Extraction

Once indexed as a `DatetimeIndex`, temporal components can be accessed directly through the index properties without string parsing:

```python
# Extract calendar features into distinct analysis columns
df["Year"] = df.index.year
df["Month"] = df.index.month
df["Weekday"] = df.index.day_name()
df["DayOfWeek"] = df.index.weekday  # Monday=0, Sunday=6
df["Is_Weekend"] = df.index.weekday >= 5
```

### Intuitive Partial-String Slicing

Pandas leverages the `DatetimeIndex` to evaluate partial date strings, enabling human-readable temporal lookups:

```python
# Specific exact date lookup
day_record = df.loc["2017-08-10"]

# Entire year or month slicing
year_2016 = df.loc["2016"]
feb_2012 = df.loc["2012-02"]

# Inclusive chronological range slicing
multi_year_window = df.loc["2014-06-01":"2016-12-31"]
```

---

## 4. Resampling vs. Rolling Window Aggregations

A critical distinction in temporal data wrangling is the difference between changing observation frequency (**resampling**) versus calculating moving statistical windows across existing records (**rolling calculations**).

The following comparative table highlights the architectural differences between these two operations:

| Attribute | `.resample()` (Downsampling) | `.rolling()` (Moving Window) |
| :--- | :--- | :--- |
| **Output Row Count** | Compresses records (e.g., 365 daily rows become 12 monthly rows). | Retains exact original row count (preserves daily granularity). |
| **Frequency Index** | Modifies the index to new interval bins (`W`, `ME`, `YE`). | Keeps original index intact. |
| **Primary Purpose** | Structural aggregation across calendar periods. | Smoothing short-term noise to isolate underlying trends. |
| **Edge Behavior** | Summarizes complete intervals; requires `min_count` for partial data. | Generates leading `NaN` values equal to `(window_size - 1)`. |

### Frequency Codes for Resampling

Standard Pandas frequency aliases used with `.resample()` include:

* `'D'`: Calendar day
* `'B'`: Business day (excludes weekends)
* `'W'`: Weekly frequency
* `'ME'`: Month end (replaces legacy `'M'`)
* `'QE'`: Quarter end (replaces legacy `'Q'`)
* `'YE'`: Year end (replaces legacy `'A'` / `'Y'`)

### Syntax Implementation

```python
# 1. Resampling to weekly and monthly aggregates
weekly_summary = df.resample("W").mean()

# Prevent false zeroes in sum aggregations over periods with missing data
monthly_consumption = df["Consumption"].resample("ME").sum(min_count=28)

# 2. Rolling window calculations (7-day and 365-day moving averages)
# Center window to align smoothed value with mid-interval date
df["Consumption_7D_Avg"] = df["Consumption"].rolling(window=7, center=True).mean()

# Long-term annual trend line (requires minimum 360 valid daily observations)
df["Consumption_365D_Avg"] = df["Consumption"].rolling(
    window=365,
    center=True,
    min_periods=360
).mean()
```

---

## 5. Statistical Visualization with Seaborn

Seaborn integrates natively with Pandas DataFrames and Matplotlib. It automatically detects `DatetimeIndex` timestamps along the horizontal axis, applies appropriate date tick formatting, and computes aggregate error bands for repeated temporal observations.

### Tidy Data Reshaping for Multi-Series Charting

While Pandas typically stores metrics across multiple wide columns, Seaborn plotting thrives when wide tables are reshaped into tidy long-form format using `pd.melt()`:

```python
import seaborn as sns
import matplotlib.pyplot as plt

# Apply consistent aesthetic styling
sns.set_theme(style="whitegrid", palette="tab10")

# Reshape wide energy columns into a single categorical variable
tidy_energy = df.reset_index().melt(
    id_vars=["Date"],
    value_vars=["Consumption", "Wind", "Solar"],
    var_name="Energy_Type",
    value_name="GWh"
)
```

### Multi-Series Trend Line Plot

Plotting multiple temporal trajectories is achieved through the `hue` and `style` parameters:

```python
plt.figure(figsize=(12, 5))
sns.lineplot(
    data=tidy_energy,
    x="Date",
    y="GWh",
    hue="Energy_Type",
    style="Energy_Type",
    linewidth=1.8
)

plt.title("Daily Electricity Metrics in Germany (2006–2017)", fontsize=14, fontweight="bold")
plt.xlabel("Timeline", fontsize=11)
plt.ylabel("Gigawatt Hours (GWh)", fontsize=11)
plt.tight_layout()
plt.show()
```

### Seasonal Distribution Analysis via Box Plots

Box plots uncover cyclic variability by grouping distributions across calendar periods:

```python
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Monthly seasonality: Winter heating peaks vs summer troughs
sns.boxplot(
    data=df,
    x="Month",
    y="Consumption",
    ax=axes[0],
    color="#3B82F6"
)
axes[0].set_title("Electricity Consumption Distribution by Month")
axes[0].set_xlabel("Month of Year")

# Weekly cycle: High industrial weekday baselines vs weekend drops
day_order = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"]
sns.boxplot(
    data=df,
    x="Weekday",
    y="Consumption",
    order=day_order,
    ax=axes[1],
    color="#10B981"
)
axes[1].set_title("Electricity Consumption Distribution by Day of Week")
axes[1].set_xticklabels(axes[1].get_xticklabels(), rotation=30)

plt.tight_layout()
plt.show()
```

---

## 6. Diagnostic Troubleshooting & Common Traps

Laboratory debugging frequently traces back to subtle data type mismatches or unaligned indices. Review this diagnostic checklist when encountering unexpected behavior:

1. **Date Column Stored as Object/String:**
   * *Symptom:* `df.loc['2016']` raises a `KeyError`, or plotting x-axes results in a solid black smear of overlapping strings.
   * *Resolution:* Run `df.index = pd.to_datetime(df.index)`. Verify that `df.index.dtype` reports `datetime64[ns]`.
2. **False Zeroes in Missing Sum Resamples:**
   * *Symptom:* Missing dates default to `0.0` when calling `.sum()`, drastically distorting monthly totals.
   * *Resolution:* Always pass `min_count` (e.g., `df.resample('ME').sum(min_count=20)`) to ensure intervals with sparse data evaluate to `NaN` rather than zero.
3. **SettingWithCopyWarning:**
   * *Symptom:* Pandas warns about assigning values to a slice.
   * *Resolution:* When filtering a date range before adding new columns, explicitly clone the memory slice: `subset = df.loc['2015':'2017'].copy()`.
4. **Out-of-Order Execution Bugs:**
   * *Symptom:* Cells fail or produce inconsistent outputs after editing prior blocks.
   * *Resolution:* Clear kernel memory and re-run all cells sequentially via **Restart Kernel and Run All Cells**.
