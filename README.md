# Flu Surge Outlook

**A flu forecasting dashboard that turns weekly surveillance data into one-to-four-week activity outlooks.**

[Explore the live demo](https://flu-surge-outlook.netlify.app/) · [Results](#what-we-found) · [Run locally](#run-locally)

Flu Surge Outlook helps users explore how influenza activity is changing, compare forecasts from five models, and understand the probability of reaching a higher activity level. It combines a public-facing dashboard with reproducible data preparation, model evaluation, and exploratory analysis.

This is Flora Jeong’s portfolio fork of a collaborative project, originally titled **Seeing the Surge Coming**. The [original repository](https://github.com/CocoChengtw/flu-forecast) contains the team’s shared work. The linked demo is the team deployment; edits to this fork do not update it.

## Explore the product

- **Activity outlook:** see the current signal level and the probability of crossing a high-activity threshold within the next four weeks.
- **Model comparison:** compare seasonal naïve, ETS, ARIMA, Prophet, and Chronos forecasts alongside their uncertainty and historical performance.
- **Seasonal context:** explore how the current trajectory compares with previous flu seasons.
- **Geographic context:** inspect state-level activity on the dashboard’s map.
- **Plain-language explanations:** use the assistant to ask about the displayed data and forecasts.

The dashboard opens with a historical case study, including an option to reveal what happened next. A separate latest-week view shows the newest included observations, with forecast availability depending on the model. A hosted demo does not imply an automatically refreshed surveillance feed.

## What the forecasts measure

The target is national weekly **ILI+**, a proxy for influenza-related outpatient activity:

**ILI+ = weighted percentage of visits for influenza-like illness × influenza test positivity / 100**

ILI describes the share of outpatient visits involving flu-like symptoms. Test positivity adds information about influenza specifically. Combining them accounts for both symptom-related visit volume and influenza positivity, while retaining limitations from care-seeking and testing patterns. ILI+ is not a direct count of infections or population prevalence.

The project uses 523 consecutive national weekly observations from October 2016 through October 2026, sourced from CDC FluView through the [Delphi Epidata API](https://cmu-delphi.github.io/delphi-epidata/). HHS Region 9 data are also included; reported model comparisons are national.

| Signal | ILI+ threshold | Interpretation |
| --- | ---: | --- |
| Early rise | 0.0282 | Departure from low off-season activity |
| High activity | 0.6939 | A relatively high level of historical in-season activity |

The early-rise threshold uses the off-season mean plus two sample standard deviations. The high-activity threshold uses the 75th percentile of in-season observations from the three seasons before the main 2025–26 evaluation period. These are research definitions, not validated staffing or bed-capacity triggers.

When activity is below the threshold, the main event output estimates the probability that at least one of the next four weeks exceeds it. If activity is already above, the product displays that status. Dashboard values expressed per 1,000 visits use ILI+ × 10, so 0.6939 corresponds to approximately 6.94 on that scale.

## What we found

Models are evaluated at weekly rolling forecast origins, using only observations through each origin, with horizons of one to four weeks and 500 simulated paths. The main evaluation season is 2025–26, with 2024–25 as an additional check.

- **ARIMA was consistent across both seasons:** distributional forecast error (CRPS) was approximately 27% and 42% lower than the seasonal naïve baseline.
- **Chronos produced competitive point forecasts, but underestimated uncertainty:** its nominal 90% intervals contained observations only about 60–61% of the time.
- **Prophet’s performance varied by season:** it achieved the lowest CRPS in 2025–26 but performed worse than the baseline in 2024–25.
- **Wastewater tracked influenza-A ILI+ closely without a consistent lead:** in four complete seasons, the strongest level correlations occurred at the same week or with wastewater one week later. Weekly changes aligned best at zero lag in every season.

[Full model metrics](results/metrics_summary.csv) · [Wastewater analysis](notes/wastewater_lead_exploration.md)

![Forecast comparison with observed influenza activity](results/figures/forecast_vs_actual.png)

### Interpreting the results

This is a research prototype for exploring surveillance signals and forecast uncertainty. Results use revised historical data rather than an as-of-publication reconstruction. Threshold calibration includes 2024–25, so that season’s threshold-based evaluation is not fully independent. The unusually low 2020–21 period is imputed for modeling, while threshold calculations retain raw observations. Wastewater correlations are descriptive and do not establish operational advance warning or incremental forecast value.

The forecasting models, evaluation framework, dashboard, assistant, and presentation were collaborative work by **Coco Cheng, Flora Jeong, Christina, Frank, and Mark**. This fork preserves the original Git history and attribution.

## Run locally

### Dashboard

The checked-in dashboard data lets you explore the interface without rerunning the models. Install Node.js and npm, then:

```bash
git clone https://github.com/jyjeong027/flu-forecast.git
cd flu-forecast/app
npm install
npm run dev
```

See [dashboard documentation](app/README.md) for the optional AI assistant, data export, and deployment setup. Live assistant answers require an API key; keep credentials outside version control.

### Analysis

From the repository root, use Python 3.11:

```bash
python3.11 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python -m src.audit_threshold
python -m src.eda
python -m src.explore_high_activity_threshold
python -m src.explore_wastewater_lead
```

On Windows, activate with `.venv\Scripts\activate`.

## Repository guide

| Location | Contents |
| --- | --- |
| [app/](app/) | React + TypeScript dashboard and assistant integration |
| [src/](src/) | Data preparation, forecasting, evaluation, and exploration |
| [notebooks/](notebooks/) | Exploratory analysis |
| [data/processed/](data/processed/) | Weekly surveillance and comparison datasets |
| [results/](results/) | Forecasts, metrics, figures, and analysis outputs |
| [notes/](notes/) | Methods, decisions, and limitations |
| [tests/](tests/) | Focused checks for analysis and model behavior |

**Stack:** Python, pandas, NumPy, statsmodels, Prophet, Chronos, React, TypeScript, Vite, and Netlify.

## Attribution and license

Based on the [original team repository](https://github.com/CocoChengtw/flu-forecast). Source data include CDC FluView/ILINet and clinical laboratory surveillance via Delphi, and CDC NWSS influenza-A wastewater measurements. See the analysis notes for dataset-specific processing and limitations.

Code is distributed under the existing [Apache 2.0 license](LICENSE). Source datasets remain subject to their providers’ terms.
