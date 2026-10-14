# Allocation Planner

Streamlit app for planners to send the allocation optimizer's output to BigQuery.
Follows the `scs-com-allocation` pipeline: `queries_main.py` → `model_main.py` → `results_main.py`. The optimizer itself is not in this app; in demo mode the model step returns a stored solution, and GCS / BigQuery are simulated locally.

**Login:** planner only. Demo account: `yogesh` / `planner123` (turned off once real users are set in Secrets).

| Screen | Script it follows | What it does |
|---|---|---|
| 1 · Data prep | `queries_main.py` | settings, bool/int params, query pipeline, `make_dataset()` → `query_parameters.pkl` |
| 2 · Model run | `model_main.py` | download + unpack inputs, split by class, run optimization (demo: stored solution) → `results_output.pkl` |

Then, following `results_main.py`:

| Screen | What it does |
|---|---|
| 3 · Review & adjust | Load `results_output.pkl` (upload or GCS), a CSV/Excel, or demo output; set run parameters; **adjust before publishing** — include / exclude / add storage types, edit each building's location and capacity (buildings shown by ID + location), exclude departments; review and check |
| 4 · Save results to BigQuery | Choose project / dataset / tables and write mode, preview, check, publish, see publish history and query the tables |

Every adjustment is recorded in `run_log.adjustments_json`.

Tables written (partitioned by month on `target_month`): `allocation_plan`, `capacity_summary`, `warehouse_share`, `run_log`.

## Run locally
```bash
pip install -r requirements.txt
streamlit run app.py
```

## Deploy on Streamlit Community Cloud
1. Push this repo to GitHub.
2. share.streamlit.io → **Create app** → pick the repo, branch `main`, main file `app.py`.
3. (Optional) **Advanced settings → Secrets**: add users and the BigQuery key (see `.streamlit/secrets.toml.example`).
4. Deploy and share the link.

## Users
```bash
python app.py adduser <username> "<Full Name>" <password>
```
Paste the printed block into Secrets.

## Real BigQuery (connected mode)
Without a key the app runs in **demo mode** (a local SQLite file stands in for BigQuery).
To write to real BigQuery: add `[gcp_service_account]` and `[bigquery]` to Secrets and uncomment the
Google lines in `requirements.txt`. The service account needs BigQuery Data Editor + Job User on the
dataset (and Storage Object Viewer to read `results_output.pkl` from GCS).

## Notes
- Only upload `.pkl` files produced by your own pipeline — pickle files can run code when opened.
- Publish history is stored in `data/` on the app server and is reset when Streamlit Cloud restarts the app.
