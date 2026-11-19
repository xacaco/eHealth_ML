# Usage Analytics of a Digital Health Service: Demand Forecasting and User Profiling (Python)

A user experience (UX) research project on how a digital health service is used, based on a session log (one row per session, with a provider ID, a user ID and a date). It looks at usage from two angles:

1. **Usage over time**: describing and forecasting daily and weekly demand (time-series analysis), and estimating capacity and cost.
2. **Usage patterns across users**: grouping users by how they engage with the service (unsupervised clustering).

Both analyses use only behavioural usage data (who had a session with whom, and when).

> **All data in this repository are simulated.** Variable names, volumes, dates, calendars, parameter values and cluster solutions are invented for demonstration and do not correspond to any real dataset or service.

**Tools:** Python (pandas, NumPy, SciPy, statsmodels, Prophet, scikit-learn, matplotlib, seaborn), Jupyter.

---

## Repository contents

| File | Purpose |
| --- | --- |
| `Demand_Forecasting.ipynb` | Aggregation, exploration, seasonal-trend decomposition, Prophet forecasting, hyperparameter tuning, validation, and capacity and cost indicators |
| `Usage_Clustering.ipynb` | Simulated session log with programme rules, feature engineering, imputation and standardisation, k-means model selection with four validity indices and a weighted rank average, cluster profiles and plots |

---

## Part 1: Time-series analysis (`Demand_Forecasting.ipynb`)

### 1. Data and aggregation
- Simulated session log with a growing trend, weekly and yearly seasonality, and low-demand calendar events.
- The most recent weeks are trimmed so incomplete periods are not analysed.
- Sessions aggregated to **daily**, **weekly** and **monthly** counts.
- The first appearance of each provider and each user gives the series of **new providers** and **new users**.

### 2. Exploratory visualisation
- Bar charts of sessions per day, week and month (publication-style, exportable at high resolution).
- Daily sessions split by weekdays, Saturdays and Sundays.
- A combined weekly view of sessions, new users and new providers.

### 3. Seasonal-trend decomposition
- Weekly counts decomposed with **STL** (period = 52 weeks).
- Multiplicative variants: classical multiplicative decomposition, and robust STL on log counts back-transformed to the original scale.
- One yearly seasonal cycle extracted and overlaid with calendar events.

### 4. Forecasting with Prophet
- **Model:** [Prophet](https://facebook.github.io/prophet/), a generalised additive model with a piecewise-linear trend, Fourier-based seasonality and holiday effects.
- **Baseline model:** multiplicative seasonality on the raw daily counts.
- **Final model:** `log1p`-transformed counts (no negative forecasts), additive seasonality.
- **Hyperparameter tuning:** grid search over `seasonality_prior_scale`, `changepoint_prior_scale` and `holidays_prior_scale`, scored by rolling-origin cross-validation MAPE.
- **Validation:** Prophet's `cross_validation` and `performance_metrics` at several horizons, with a **log-normal bias correction** when back-transforming log-scale forecasts.

### 5. Capacity, saturation and cost
- **Daily carrying capacity:** active providers (rolling window) × mean sessions per provider per day.
- **Saturation date:** first forecast day on which expected demand exceeds current capacity, assuming no new providers.
- **Expected expenditure:** forecast sessions × a unit cost per session.

---

## Part 2: Usage clustering (`Usage_Clustering.ipynb`)

### 1. Simulated session log
- 7,500 users simulated over a **36-month** observation window.
- Each user is assigned a usage style (one-off, short, moderate or long) that sets how many months they are active, how spread out those months are, and how many sessions they have per active month.
- Two programme rules are applied and checked:
  - **At most 12 sessions per year** per user, the maximum the programme covers.
  - **Inclusion criterion:** only users whose first session is at least 12 months before the end of the window are kept, so every user has at least one year of possible activity.

### 2. Feature engineering
Each user's session history is aggregated into monthly engagement features:

| Feature | Description |
| --- | --- |
| `active_months` | Distinct months with at least one session |
| `total_months` | Span in months between first and last session |
| `sessions_per_month` | Mean sessions in active months |
| `sessions_in_peak_month` | Maximum sessions in a single month |
| `gini_monthly` | Gini coefficient of the monthly distribution (unevenness of use) |

### 3. Preprocessing
- Missing values (e.g. the Gini index, undefined for a single active month) imputed with the **median** (`SimpleImputer`).
- Features **standardised** (`StandardScaler`) so each contributes equally to the Euclidean distances used by k-means.

### 4. Model selection
- **k-means** fitted for k = 2 to 6, the largest number of clusters still considered interpretable.
- Each solution scored with inertia (elbow), silhouette, Calinski–Harabasz and Davies–Bouldin.
- The final k is selected automatically with a **weighted rank average**: each k is ranked on silhouette, Calinski–Harabasz and Davies–Bouldin, with silhouette and Calinski–Harabasz counting double.

### 5. Cluster description
- Mean feature profile and size of each cluster.
- Descriptive labels (Brief one-off, Long-term, Intensive short-term, Irregular with peaks) assigned after reviewing each cluster's profile (LLM-assisted naming).
- Visualisation in two dimensions with **PCA**, and a bar chart of users per profile.

---

## Author

Xavier Calvet
