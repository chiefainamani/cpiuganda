# Uganda CPI Intelligence System

An interactive system for exploring Uganda's Consumer Price Index (CPI) using monthly data published by the Uganda Bureau of Statistics (UBOS). It combines an API-backed dashboard with statistical analysis and short-term forecasting.

**Authors:** Ainamani Dickson and Lumu Richard

## Open the dashboard

Use the hosted dashboard; no installation or local server is needed:

**[Open the Uganda CPI Intelligence dashboard](https://chiefainamani.github.io/ugandacpi/)**

The dashboard is hosted on GitHub Pages and retrieves its data from a separately hosted API. The API root reports the coverage of the data currently deployed, so the dashboard always reflects the latest release that has been processed and published to the API.

## What the system provides

- **Overview:** Headline CPI and inflation trends, summary statistics, and anomaly indicators.
- **Inflation drivers:** Category contributions and cumulative price growth.
- **Household cost burden:** Estimate how a user-defined basket's cost changes between selected months.
- **Essential vs discretionary analysis:** Compare inflation for the two groups using a Welch t-test.
- **Forecasting:** View short-term CPI forecasts, prediction intervals, validation metrics, and model-comparison evidence.

The dashboard requests current values from the API rather than embedding monthly figures in its interface. This allows new releases to be displayed when the data pipeline and API deployment have been updated.

## How it works

1. The ingestion pipeline reads and validates a UBOS CPI Excel workbook.
2. Analysis and forecasting modules prepare CPI series, category contributions, household cost comparisons, anomaly indicators, and forecast evidence.
3. A Flask API serves the results as JSON.
4. The browser dashboard requests the API data and renders interactive charts and tables.

The public dashboard and API are deployed separately:

- **Dashboard:** [chiefainamani.github.io/ugandacpi](https://chiefainamani.github.io/ugandacpi/)
- **API base URL:** [ugandacpi.up.railway.app/api/v1](https://ugandacpi.up.railway.app/api/v1)

## Project structure

```text
.
├── api/                  Flask API
├── data/
│   └── raw/              UBOS Excel workbooks
├── forecasting/          Forecast models and model selection
├── outputs/              Generated analytical and model-selection outputs
├── pipeline/             Data ingestion, configuration, and analysis
├── tests/                Automated tests
├── index.html            Dashboard entry point
├── main.py               Application entry point
├── pyproject.toml        Python project and dependency configuration
└── uv.lock               Locked Python dependencies
```

## API

The API returns JSON. Its root endpoint, [`/`](https://ugandacpi.up.railway.app/api/v1/), provides API metadata and the deployed dataset's coverage. All endpoints use the `/api/v1` prefix and accept `GET` requests.

| Endpoint | Description |
|---|---|
| `/summary` | Dataset coverage and headline summary statistics |
| `/headline` | Headline CPI series and inflation rates |
| `/categories` | CPI series for all categories |
| `/categories/<name>` | CPI series and weight for one category |
| `/weights` | Expenditure weights |
| `/drivers?period=` | Category contributions for a selected period |
| `/growth?base=` | Cumulative growth from a selected base period |
| `/burden?basket=&base=&target=` | Household basket cost comparison |
| `/essential-vs-discretionary` | Comparison of inflation between the two groups |
| `/anomalies?threshold=` | Anomaly indicators |
| `/forecast?horizon=` | Forecast from the selected model |
| `/forecast/validation` | Selected model's validation results |
| `/forecast/models` | Model-selection evidence and comparison |
| `/forecast/ols` | OLS benchmark forecast |
| `/forecast/ols/validation` | OLS benchmark validation results |
| `/forecast/sarima` | SARIMA benchmark forecast |
| `/forecast/sarima/validation` | SARIMA benchmark validation results |
| `/forecast/comparison` | Forecast comparison across models |

Example request to the public API:

```bash
curl "https://ugandacpi.up.railway.app/api/v1/summary"
```

## Development

These instructions are for contributors; dashboard visitors should use the hosted link above.

### Requirements

- Python 3.13 or later
- [uv](https://docs.astral.sh/uv/)
- Git

### Run locally

```bash
git clone https://github.com/chiefainamani/ugandacpi.git
cd ugandacpi
uv sync
uv run python -m api.app
```

The API starts on `http://localhost:5000`. Open the root `index.html` in a browser to use the dashboard with the local API. The repository includes UBOS workbooks under `data/raw/`; the selected workbook and dataset settings are maintained in `pipeline/config.py`.

### Run tests

```bash
uv run python -m pytest tests/ -v
```

## Updating the data

When UBOS publishes a new CPI workbook, place it in `data/raw/`, update the configured workbook and dataset metadata in `pipeline/config.py`, and check category names and weights for revisions. Restart the API to ingest the workbook and refresh model-selection results for the new data endpoint. Verify that `/api/v1/summary` reports the expected latest period. To publish the new release to the public dashboard, deploy the refreshed API and verify its `/api/v1/` metadata reports the expected coverage.

## Data source

CPI data is published by the [Uganda Bureau of Statistics (UBOS)](https://www.ubos.org/). The dashboard presents analysis of that source data and should not be treated as an official UBOS publication.
