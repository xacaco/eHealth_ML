# Usage Analytics of a Teleconsultation Mental-Health Program: Demand Forecasting and User Profiling

A user experience (UX) research project on a nationwide digital health tool: **Santé Psy Étudiant**, the French publicly funded program that gives higher-education students free psychological sessions (in person or by teleconsultation) with registered psychologists.

The project uses a large, pseudonymized log of program sessions to study how the service is used from two complementary angles:

1. **Usage over time**: describing and forecasting daily and weekly session demand (time-series analysis).
2. **Usage patterns across users**: grouping students according to how they engage with the service (unsupervised clustering).

Both analyses rely only on behavioural usage data (who had a session with whom, and when). No clinical content is involved.

---

## Repository contents

| File | Purpose |
| --- | --- |
| `PsyEt_TimeSeries.ipynb` | Data exploration, seasonal-trend decomposition, Prophet forecasting, hyperparameter tuning and validation |
| `PsyEt_Cluster.ipynb` | Feature standardization, k-means clustering, cluster validity indices, cluster summaries and plots |
| `SantéPsy_manuscript.docx` | Manuscript describing the study (background, methods, discussion) |


> The data were provided by the French Ministry of Higher Education and Research and are **not included** in this repository. Psychologists and students are identified only by pseudonymized IDs, with no correspondence table available to the researchers.

---

## Data

- **Source:** sessions registered on the program platform from 1 March 2021, with the analysis cut-off set at 27 April 2025.
- **Raw volume:** 442,973 sessions.
- **Session-level cleaning:** sessions registered more than once for the same student on the same day were treated as registration errors and removed, leaving 440,729 sessions for forecasting.
- **Variable screening:** the additional variables in the source files (e.g. psychologist department, diploma, student gender, birth date) were assessed for missing values, duplicates, free-text recodability and plausibility of values. Only variables of sufficient quality and relevance were retained (see supplementary Table S1 in the manuscript).
- **Clustering sample:** students with more than 12 sessions in a year were excluded (above the subsidized maximum), and only students whose first consultation was at least one year before the cut-off were kept, so that recent joiners do not inflate low-engagement profiles. This left 47,989 students.

---

## Part 1: Time-series analysis (`PsyEt_TimeSeries.ipynb`)

### 1. Loading and aggregation
- Session dates parsed to datetime; the most recent weeks are trimmed so that incomplete periods are not analysed.
- Sessions aggregated to **daily**, **weekly** and **monthly** counts.
- The first appearance of each psychologist and each student was used to derive series of **new psychologists** and **new students** per day and per week.

### 2. Exploratory visualization
- Bar charts of sessions per day, week and month.
- Daily sessions split by weekday, Saturday and Sunday.
- A combined weekly view of sessions, new students and new psychologists.
- Publication-style versions of the main figures exported as high-resolution JPGs.

### 3. Seasonal-trend decomposition
- Weekly session counts decomposed with **STL** (period = 52 weeks), with additive and multiplicative variants (a log-transformed robust STL and `seasonal_decompose` in multiplicative mode).
- One yearly seasonal cycle extracted and overlaid with the **academic calendar** (university start, holiday periods, exam periods) and **French national holidays**.

### 4. Forecasting with Prophet
- **Model:** [Prophet](https://facebook.github.io/prophet/), a generalized additive model with piecewise-linear trend, Fourier-based seasonality and holiday effects. Weekly and yearly seasonality and the built-in **French holiday calendar** were included.
- **Baseline model:** multiplicative seasonality on the raw daily counts.
- **Final model:** daily counts transformed with `log1p` to prevent negative predictions, additive seasonality, linear growth.
- **Hyperparameter tuning:** grid search over `seasonality_prior_scale` {0.01, 0.1, 1, 10}, `changepoint_prior_scale` {0.001, 0.01, 0.1, 0.5} and `holidays_prior_scale` {0.01, 0.1, 1, 10}, each combination evaluated with Prophet's rolling-origin cross-validation and compared by MAPE.
- **Validation:** time-series cross-validation with Prophet's diagnostics (`cross_validation`, `performance_metrics`) after back-transforming predictions to the original scale. The notebook also includes a bias-correction step for log-scale forecasts.
- **Forecast horizon:** the final model generates forecasts one year ahead; validation is run at multiple horizons.

### 5. Capacity, saturation and cost indicators
- **Daily carrying capacity** defined as the number of active psychologists multiplied by the mean number of sessions per psychologist per day, with per-psychologist session statistics (min, max, mean per day) computed from the session log.
- A **saturation date** is defined as the date on which forecasted sessions reach carrying capacity, assuming no new psychologists join.
- **Expected expenditure** estimated from forecasted sessions and the per-session reimbursement rate.

---

## Part 2: Usage clustering (`PsyEt_Cluster.ipynb`)

### 1. Feature engineering
For each student, the session history was aggregated into monthly engagement features:

| Feature | Description |
| --- | --- |
| `active_months` | Number of distinct months with at least one session |
| `total_months` | Span of months between first and last session |
| `sessions_per_month` | Mean number of sessions in months where the student was active |
| `sessions_in_peak_month` | Maximum number of sessions in a single month |
| `gini_monthly` | Gini coefficient of the monthly session distribution (inequality of use across active months) |

### 2. Preprocessing
- Missing values imputed with the **median** (`SimpleImputer`).
- Features **standardized** (`StandardScaler`).

### 3. Model selection
- **k-means** (`random_state=42`) fitted for k = 2 to 8, a range chosen by the research team as the maximum still considered interpretable.
- Each solution evaluated with five indices:
  - Inertia (elbow plot)
  - Silhouette coefficient
  - Calinski–Harabasz index
  - Davies–Bouldin index
  - Dunn index (custom implementation)
- The final number of clusters was chosen by weighing the indices together, giving priority to the silhouette and Calinski–Harabasz scores.

### 4. Cluster description
- Final model fitted with **k = 6**.
- Mean of each feature computed per cluster; cluster sizes counted.
- Clusters visualized in two dimensions with **PCA** and in a bar chart of students per cluster.
- Descriptive labels for each cluster were generated from its aggregated feature values using an LLM (Le Chat, Mistral AI) prompted with the context of the program, then used as names in the notebook plots.

---

## Author

Xavier Calvet Colomé

## Authors

Xavier Calvet, Orgest Beqiri, Jean-Luc Martinot, Lise Haddouk
