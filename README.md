# Crypto Market Sentiment and Bitcoin Price Behaviour: A Statistical Analysis

## 1. Overview
This project investigates whether the **Crypto Fear & Greed Index** - a daily composite sentiment indicator - is statistically associated with Bitcoin price behaviour metrics including daily log-returns, realised intraday volatility, and trading volume.

The analysis combines two independent data sources, applies rigorous data preprocessing and feature engineering, and uses non-parametric hypothesis testing to formally evaluate the relationship between market sentiment and price dynamics.

---

## 2. Research Problem

Cryptocurrency markets are widely considered to be sentiment-driven. Yet it remains an open empirical question whether measurable differences in price behaviour - returns, volatility, and volume - can be attributed to the prevailing market sentiment regime.

This project investigates that question statistically.

---

## 3. Objectives

1. Consolidate two independent datasets from different sources into a single analytical frame;
2. Compute financially meaningful daily metrics (log-returns, realised volatility, daily volume) from raw minute-level price data;
3. Perform EDA to understand distributions, temporal patterns, and group-level differences;
4. Formally test whether key price metrics differ statistically across sentiment categories;
5. Quantify effect sizes to distinguish statistical significance from practical significance.

---

## 4. Research Questions

1. Does daily log-return differ significantly across Fear & Greed sentiment categories?
2. Does realised intraday volatility differ significantly across sentiment categories?
3. Does daily trading volume differ significantly across sentiment categories?
4. Are sentiment classification and the direction of daily price movement (Up/Down) statistically dependent?
5. How do log-returns and volatility correlate with the raw Fear & Greed Index value?

---

## 5. Datasets

### Dataset 1 - BTC/USD Minute Price Data
**Source:** Bitstamp exchange, via [github.com/ff137/bitstamp-btcusd-minute-data](https://github.com/ff137/bitstamp-btcusd-minute-data)

**Size:**
* 6,846,600 rows
* 6 columns

**Period covered:** January 2012 - January 2025

**File included in repository:**

```
data/btcusd_bitstamp_1min_2012-2025.csv.gz
```

| Column      | Type               | Description                                     |
| ----------- | ------------------ | ----------------------------------------------- |
| `timestamp` | int64 (Unix/POSIX) | POSIX timestamp (seconds since epoch)           |
| `open`      | float              | Opening price in USD at the start of the minute |
| `high`      | float              | Highest price reached during the minute         |
| `low`       | float              | Lowest price reached during the minute          |
| `close`     | float              | Closing price at the end of the minute          |
| `volume`    | float              | BTC volume traded during the minute             |

---

### Dataset 2 - Crypto Fear & Greed Index
**Source:** Alternative.me, via [api.alternative.me/fng/?limit=0&format=csv](https://api.alternative.me/fng/?limit=0&format=csv)

**Size:**
* 3,007 rows (one per day)
* 3 columns

**Period covered:** February 2018 - present (daily)

**Fetched directly from API** - no file download required:

```python
FNG_URL = 'https://api.alternative.me/fng/?limit=0&format=csv'
fng_raw = pd.read_csv(FNG_URL, skiprows=3)
```

| Column                 | Type                 | Description                                           |
| ---------------------- | -------------------- | ----------------------------------------------------- |
| `timestamp`            | date string          | Date of the observation                               |
| `value`                | int                  | Composite sentiment score (0–100)                     |
| `value_classification` | categorical          | Extreme Fear / Fear / Neutral / Greed / Extreme Greed |

**Effective analysis period:** February 2018 – January 2025 (6 years and 11 months)

---

## 6. Statistical Framework

Significance level pre-registered before running any test (to avoid p-hacking):
```
α = 0.05
```

**Standard testing protocol applied to all hypotheses:**
1. **Normality check** - Shapiro-Wilk test on each group;
2. **Variance homogeneity** - Levene's test;
3. **Main test** - Kruskal-Wallis (Tests 1–3) or Chi-Square (Test 4);
4. **Effect size** - $\eta ^ 2$ for Kruskal-Wallis, Cramér's V for Chi-Square;
5. **Post-hoc pairwise comparisons** - Dunn's test with Bonferroni correction (implemented manually, as SciPy has no native implementation).

---

## 7. Project Structure

```
crypto_sentiment_btc_analysis.ipynb         # Main analysis notebook
data/                                       # Datasets folder
├── btcusd_bitstamp_1min_2012-2025.csv.gz   # BTC/USD minute price data
└── api.txt                                 # URL for Crypto Fear & Greed Index Dataset
README.md                                   # Project documentation
```

---

## 8. Exploratory Data Analysis

EDA is structured to inform test selection before hypothesis testing, not after. Key sections include:

**Distribution analysis**
* `log_return`, `volatility`, and `volume` are all non-normal (Shapiro-Wilk p < 0.001), ruling out parametric tests;
* `log_return` exhibits heavy tails - excess kurtosis is a well-documented feature of crypto returns;
* `volatility` and `volume` are strongly right-skewed.

**Outlier detection and treatment**
* IQR method applied to `log_return`, `volatility`, and `volume`;
* **Winsorization (capping)** used instead of row removal - extreme price moves are genuine market events, not data errors.

**Temporal patterns**
* Clear cyclical behaviour: 2018–2019 bear market, the COVID crash of March 2020, 2020–2021 bull run, 2022 crypto winter, 2023–2024 recovery;
* Fear & Greed Index closely tracks price cycles - *Extreme Greed* periods coincide with price peaks, *Extreme Fear* with drawdowns;
* Volatility spikes cluster around sentiment regime transitions.

**Group-level comparisons**
* Log-returns show a broadly monotonic pattern - negative medians in fear categories, positive in greed categories;
* Volatility exhibits a U-shaped pattern - both sentiment extremes show elevated intraday turbulence relative to *Neutral*;
* Volume partially follows a U-shaped pattern - `Neutral` days have the lowest trading activity, though the effect is asymmetric: `Extreme Greed` exhibits substantially higher volume than `Extreme Fear`.

**Correlation analysis**
* Both Pearson and Spearman correlations computed. All correlations between `fng_value` and price metrics are below 0.12 in absolute value - sentiment is a noisy indicator;
* `volatility` and `volume` are strongly correlated (r $\approx$ 0.757) - elevated intraday turbulence consistently coincides with higher trading activity.

---

## 9. Key Findings

* **Volatility is strongly linked to sentiment** -> $\eta^2$ = 0.1092 (medium effect); both *Extreme Fear* and *Extreme Greed* exhibit elevated turbulence relative to intermediate categories, confirming a U-shaped pattern;
* **Volume patterns follow sentiment extremes** -> $\eta^2$ = 0.0523 (small effect); trading activity is highest during *Extreme Greed*, partially consistent with a U-shaped relationship;
* **Log-return differs statistically, but with negligible practical effect** -> $\eta^2$ = 0.0044 (negligible); only *Extreme Fear* vs. *Greed* and *Fear* vs. *Greed* differ significantly in post-hoc analysis;
* **Sentiment and price direction are weakly dependent** —> Cramér's V = 0.0715 (small); *Fear* periods show a slight excess of Down days, *Greed* periods a slight excess of Up days;
* **Correlations between sentiment and price metrics are weak** -> Both Pearson and Spearman correlations between `fng_value` and price metrics are below 0.12 in absolute value, indicating limited predictive power for daily price behaviour;
* **Effect size hierarchy: volatility > volume > log_return** —> sentiment most meaningfully influences market risk and participation, while its effect on returns is practically negligible.

---

## 10. Technologies Used
* Python
* Pandas
* NumPy
* Seaborn
* Matplotlib
* SciPy
* Statsmodels
* Itertools
* Jupyter Notebook

---

## 11. How to Run the Project

1. Clone the repository

```bash
git clone https://github.com/lozeva/crypto-sentiment-btc-analysis.git
cd crypto-sentiment-btc-analysis
```

2. Install dependencies

```bash
pip install pandas numpy matplotlib seaborn scipy statsmodels jupyter
```

3. Run the notebook

```bash
jupyter notebook
```

Open:
```
crypto_sentiment_btc_analysis.ipynb
```

> **Note:** Dataset 2 (Fear & Greed Index) is fetched directly from the Alternative.me API at runtime - no manual download required. An internet connection is needed when running the notebook.