# Medellín Road Safety: Injury-Risk Prediction & Hotspot Prioritization

> End-to-end ML project on **700K+ public traffic incidents in Medellín (2008–2025)**: from raw data on AWS S3 to a deployed prediction API and a BI dashboard that tells the city *where and when* to focus prevention.

<!-- Replace the placeholders [ ] with your real results once you have them. Never publish a number you didn't compute. -->

[Live API] · [Dashboard] · [Notebook report]

---

## 1. Business problem

Medellín's Secretaría de Movilidad records hundreds of thousands of traffic incidents, but prevention resources (agents, campaigns, road interventions) are limited. This project answers two questions:

1. **Given an incident's time, place and type, how likely is it to involve injured or fatal victims?**
2. **Which zones and time windows concentrate the highest expected harm, so resources can be prioritized?**

## 2. Key results

| Metric | Value |
|---|---|
| Incidents analyzed | [N] |
| Best model | [model] |
| ROC-AUC (test, temporal split) | [x.xx] |
| PR-AUC vs. baseline | [x.xx] vs. [baseline] |
| Harm captured by top 20 % of zone-hour cells | [xx %] |

**Main insight:** [one sentence a non-technical manager would understand].

## 3. Data

- **Source:** Georeferenced road traffic incident dataset, Medellín 2008–2025 (Secretaría de Movilidad, curated and published on Mendeley Data). Public.
- **Unit of analysis:** incident.
- **Target:** `HAS_VICTIMS` = 1 if severity is *injured* or *fatal*, 0 if *property damage only*.
- Raw data is not versioned; download it manually from Mendeley Data and place it in data/raw/.

## 4. Approach

```
Raw CSV ──► S3 (raw/) ──► cleaning & validation ──► S3 (processed/, Parquet)
                                   │
                                   ▼
               feature engineering (time, space, incident class)
                                   │
                    ┌──────────────┴──────────────┐
                    ▼                             ▼
         Severity classifier              Zone × hour risk score
     (LogReg / RF / XGBoost +           (aggregation + capture curve)
      Optuna, temporal split)
                    │                             │
                    ▼                             ▼
            FastAPI + Docker                 Power BI dashboard
```

- **Data quality:** coordinate validation (points outside Medellín), duplicate incidents, inconsistent severity labels, missing locations.
- **Leakage control:** temporal train/test split (train ≤ 2023, test 2024–2025); no feature derived from post-incident information.
- **Imbalance:** class weights vs. SMOTE inside CV folds only.
- **Explainability:** SHAP values for global and per-prediction explanations.
- **Prescriptive layer:** rank zone-hour cells by expected harm and measure how much of the real harm the top-k % captures vs. random.

## 5. Repository structure

```
medellin-road-safety-ml/
├── README.md
├── pyproject.toml / requirements.txt
├── Makefile                 # make data | make train | make api
├── .env.example             # AWS bucket name, region (no secrets)
├── data/                    # git-ignored
├── notebooks/
│   ├── 01_eda.ipynb
│   ├── 02_data_quality.ipynb
│   ├── 03_modeling.ipynb
│   └── 04_prescriptive.ipynb
├── src/
│   ├── data/                # download.py, s3_io.py, clean.py, validate.py
│   ├── features/            # build_features.py
│   ├── models/              # train.py, evaluate.py, predict.py
│   └── api/                 # main.py (FastAPI), schemas.py
├── sql/                     # analytical queries (DuckDB / SQL)
├── models/                  # serialized model (or link to release)
├── dashboard/               # .pbix or screenshots + link
├── reports/figures/
├── tests/                   # data validation + API tests (pytest)
├── Dockerfile
└── .github/workflows/ci.yml # lint + tests on every push
```

## 6. How to run

```bash
git clone https://github.com/estefaniaospinac/medellin-road-safety-ml.git
cd medellin-road-safety-ml
pip install -r requirements.txt
cp .env.example .env          # set your S3 bucket
make data                     # download, clean, upload to S3
make train                    # train + evaluate, saves model
make api                      # http://localhost:8000/docs
```

**Example request**

```bash
curl -X POST http://localhost:8000/predict \
  -H "Content-Type: application/json" \
  -d '{"comuna": 10, "hour": 18, "weekday": 5, "incident_class": "CHOQUE"}'
```

## 7. Limitations & next steps

- Administrative records under-report minor incidents; severity labels may be inconsistent across years.
- No traffic-volume denominator: risk is relative to recorded incidents, not exposure.
- Next: add weather and traffic-volume data; monitor model drift with a monthly retrain.

## 8. Author

**Estefanía Ospina Cardona** — Mathematician, Specialist in Analytics & Data Science (UdeA).
[LinkedIn](https://www.linkedin.com/in/estefania-ospina-7319b2267/) · [GitHub](https://github.com/estefaniaospinac)
