[README.md](https://github.com/user-attachments/files/31106731/README.md)
# Google Play Store Analytics Dashboard

A set of six data-analysis tasks built on the [Google Play Store Apps dataset](https://www.kaggle.com/datasets/lava18/google-play-store-apps), each producing an interactive Plotly visualization. Every chart is documented in its own Jupyter notebook and also lives inline in a single combined dashboard (`dashboard.html`), where each chart is only visible during a specific time-of-day window (IST).

## Live dashboard

Open `dashboard.html` in any browser — no server or build step required. Each chart shows either its visualization (if the current time in IST falls in its window) or a countdown placeholder ("Opens in Xh Ym") if it doesn't. The page checks the time every 30 seconds, so it updates live without a refresh.

| Chart | Visible window (IST) |
|---|---|
| 1. Top 10 categories — rating vs. reviews | 3:00 PM – 5:00 PM |
| 2. Global installs by category (choropleth) | 6:00 PM – 8:00 PM |
| 3. Free vs. paid — installs & revenue | 1:00 PM – 2:00 PM |
| 4. Installs trend by category (line, growth-shaded) | 6:00 PM – 9:00 PM |
| 5. Size vs. rating bubble chart | 5:00 PM – 7:00 PM |
| 6. Cumulative installs (stacked area) | 4:00 PM – 6:00 PM |

## Repository structure

```
.
├── dashboard.html                     # combined dashboard, all 6 charts, time-gated
├── googleplaystore.csv                # source dataset (apps)
├── user_reviews.csv                   # source dataset (review sentiment)
│
├── task1_grouped_bar_chart.ipynb      # Task 1 notebook
├── task1_top10_categories.csv         # Task 1 output data
├── task1_chart.png                    # Task 1 static reference image
│
├── task2_choropleth_map.ipynb         # Task 2 notebook
├── task2_top5_categories.csv          # Task 2 output data
├── task2_choropleth_data.csv          # Task 2 per-country estimate data
│
├── task3_dual_axis_chart.ipynb        # Task 3 notebook
├── task3_free_vs_paid.csv             # Task 3 output data
│
├── task4_time_series_chart.ipynb      # Task 4 notebook
├── task4_cumulative_installs.csv      # Task 4 output data
├── task4_growth_flags.csv             # Task 4 growth-spike flags
│
├── task5_bubble_chart.ipynb           # Task 5 notebook
├── task5_bubble_data.csv              # Task 5 output data
│
├── task6_stacked_area_chart.ipynb     # Task 6 notebook
├── task6_cumulative_installs.csv      # Task 6 output data
├── task6_month_highlight.csv          # Task 6 highlighted-month flags
│
└── README.md
```

## Tasks

### 1. Top 10 categories by installs — avg rating vs. total reviews
Grouped bar chart. Filters: rating ≥ 4.0, size ≥ 10 MB, last updated in January. Top 10 categories by total installs, comparing average rating (left axis) against total review count (right axis).

### 2. Global installs by category — choropleth
Interactive world map with a category dropdown. Filters: category must not start with A, C, G, or S; top 5 remaining categories by installs. Categories exceeding 1M installs are highlighted (★, warm color scale).

> **Data note:** the dataset has no per-country install counts. Per-country values are *estimated* by distributing each category's total installs using each country's approximate public share of global Android users — illustrative, not exact.

### 3. Free vs. paid — avg installs & revenue (top 3 categories)
Dual-axis chart (bars = avg installs, lines = avg revenue). Filters: installs ≥ 10,000, revenue ≥ $10,000 (paid apps only), Android version > 4.0, size > 15 MB, content rating = Everyone, app name ≤ 30 characters.

> **Data note:** `Revenue = Price × Installs`. Free apps always have $0 revenue by construction, so the $10,000 revenue filter is applied only to paid apps — otherwise no free app could ever appear in a "free vs. paid" comparison.

### 4. Installs trend by category — line chart with growth shading
Filters: app name doesn't start with x/y/z, category starts with E/C/B, app name doesn't contain "s", reviews > 500. Area under each category's line is shaded wherever that category's month-over-month growth exceeds 20%. Beauty → Hindi, Business → Tamil labels are translated on the legend.

> **Data notes:** (1) "trend over time" is approximated via cumulative installs bucketed by each app's *Last Updated* month, since the dataset has no real install history. (2) The brief also asks to translate "Dating" into German, but Dating doesn't start with E/C/B, so it's excluded by the category filter and never appears on this chart.

### 5. App size vs. rating — bubble chart
Bubble size = installs. Filters: rating > 3.5, category in {Game, Beauty, Business, Comics, Communication, Dating, Entertainment, Social, Events}, reviews > 500, name doesn't contain "s", avg review sentiment subjectivity > 0.5, installs > 50,000. Game category highlighted in pink; Beauty/Business/Dating labels translated (Hindi/Tamil/German).

> **Data note:** sentiment subjectivity comes from `user_reviews.csv` (per-review), averaged per app before filtering.

### 6. Cumulative installs by category — stacked area chart
Filters: rating ≥ 4.2, app name has no digits, category starts with T or P, reviews > 1,000, size 20–80 MB. A lighter vertical band marks any month where *any* category grew more than 25% month-over-month. Travel & Local → French, Productivity → Spanish, Photography → Japanese legend labels.

## Requirements

```
pandas
numpy
plotly
jupyter
nbconvert
```

Install with:
```bash
pip install pandas numpy plotly jupyter nbconvert
```

## Running the notebooks

Each notebook is self-contained — it reads `googleplaystore.csv` (and, for Task 5, `user_reviews.csv`) from the same directory, so keep the CSVs alongside the `.ipynb` files.

```bash
jupyter nbconvert --to notebook --execute --inplace task1_grouped_bar_chart.ipynb
# ...repeat for task2 through task6
```

Or open them normally in Jupyter Lab / Notebook and run all cells.

## How the time-gating works

`dashboard.html` computes the visitor's current time in IST directly from their browser clock (not the server), independent of their actual timezone. Each chart's `<section>` declares a `data-start-hour` / `data-end-hour`; a small JS registry (`chartRegistry`) checks every 30 seconds whether "now" falls inside that chart's window, and swaps between the rendered Plotly chart and a placeholder card showing a live countdown to the next window.
