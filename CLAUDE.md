# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Malackathon 2026 (Universidad de Málaga) entry for the Mercedes-Benz Group Services Madrid challenge **"Clustering the Star"**: segment Mercedes-Benz customers with unsupervised learning (K-Means required) to drive personalized marketing (vehicle offers, maintenance plans, loyalty campaigns, targeted communications). Judging rewards **business interpretation and commercial usefulness** of the segments, not just technical metrics.

The full brief and data dictionary are in `Reto Malackathon _MercedesBenz_2026.pdf` (Spanish). Stack: Python with pandas + scikit-learn, delivered as Jupyter notebooks in `notebooks/` (one per challenge, e.g. `01_eda.ipynb`). Notebooks read the CSV via the relative path `../datos_reto_mercedes_benz_malackathon.csv`. The system `python3` has no pandas; use `uv`:

```bash
# Execute a notebook in place (run from notebooks/ so the relative data path resolves)
cd notebooks && uv run --with-requirements ../requirements.txt --with nbconvert --with ipykernel \
  jupyter nbconvert --to notebook --execute --inplace 01_eda.ipynb
```

Read the CSV with `pd.read_csv(path, index_col=0, dtype={"zipcode": str})` — without the dtype, zipcodes mix int/str and look inconsistent per client. Valid file SHA-256: `fb58cecbb924f358967fb5c95259b5f726901b6a0e0a3cc38b849665f48c1dde` (302,850 rows); an earlier Drive download was silently truncated. Plans live in `docs/`.

## Deliverables (from the brief)

Basic challenges — **all four must be fully completed** before any advanced challenge is evaluated:

1. **EDA & data quality** — descriptive stats, null/inconsistency analysis with justified treatment, key distributions, justified exclusion of ID/metadata columns. Deliver an executed notebook + short summary (main customer traits; quality issues found and how they were resolved).
2. **Feature engineering** — justified feature selection (think like a business analyst, don't use all columns), at least one derived feature (e.g. vehicle age from `date_of_first_registration`), proper categorical encoding, numeric imputation + scaling, all wrapped in a single **pipeline**. Deliver notebook + feature list with justification + description of derived features.
3. **K-Means** — run over a range of k, elbow method, Silhouette score per k, final model with chosen k, cluster size distribution. Deliver notebook with elbow/silhouette plots and justification of k.
4. **Segment interpretation** — business-language profile and descriptive name per cluster (e.g. "Premium Tradicional", "Joven Digital"), with profiling tables and charts.

Advanced challenges:

1. **Algorithm comparison** — at least one other algorithm (DBSCAN, GMM, Agglomerative) with the same k and same prepared data; compare Silhouette, analyze whether segments change, comparison chart, reasoned conclusion.
2. **PCA** — explained variance per component and cumulative, 2D (optionally 3D) scatter colored by cluster, top loadings per component, assessment of visual separability.
3. **Jackknife stability** — N subsamples (e.g. 20) each dropping 10–20% of data, retrain K-Means with the same k, measure assignment agreement for clients present in multiple runs, per-cluster stability analysis, final recommendation.

## Data

`datos_reto_mercedes_benz_malackathon.csv` (95.5 MB, 302,850 rows, committed to git). Notes that affect every notebook:

- The first column is an unnamed pandas index — read with `index_col=0`.
- Grain is **client–vehicle**, not client: a `client_id` can appear on several rows (one per vehicle, current or past via `is_current_vehicle`). Customer segmentation needs an explicit decision to aggregate to one row per client (or filter to current vehicles) — make and document this choice before clustering.
- `client_id` / `vehicle_id` are hashed identifiers; `zipcode` is an identifier too — exclude from features.
- Column groups: demographics (`age_range`, `gender_derived` — inferred, not declared), vehicle (`model_series`, `vehicle_type`, `model_class`, `product_group`, `engine_concept`, `fuel_type`, `date_of_first_registration`), brand relationship (`role`, `role_type` B2B/B2C, `months_as_mb_customer`, `currently_connected_vehicles`, `finance_leasing_status`, `workshop_visits_warranty_goodwill`), consents (`consent_*_communication_channel`), zip-code socio-economics (`purchasing_power_*`, `population`, `number_of_households`), and connected-car telemetry (`trip_distance_*`, `operating_hours_total`, `fuel_consumption_average`, `speed_*`).
- Telemetry columns are sparse (empty for many rows) and `engine_concept` / `fuel_type` have many blanks; `trip_distance_mode` and `speed_mode` are typed as strings in the dictionary.
- Socio-economic columns are zip-code level, so they are shared by all clients in the same zip and are partly redundant with each other (`purchasing_power_millions_eur` ≈ per-household × households).

## Course notes (reference material)

The team's university notes (Spanish/English PDFs) are committed as reference. Prefer the techniques they teach so the work matches what judges/mentors expect from UMA students:

- `EDA.pdf` — **data preprocessing** notes: missing values (drop rows only if <5% and not biased; drop columns that are mostly empty; mean/median/mode, regression, KNN with Gower distance, EM imputation), outliers (Z-score >3, IQR 1.5 rule, boxplots; remove only if error and <5%, otherwise impute or log/sqrt transform), redundancy detection (Chi-square for nominal, Pearson/Spearman |r| > 0.8 for numeric), scaling (Min-Max, Z-score, decimal), transforms (log, power, Box-Cox), encoding (one-hot, label, ordinal), feature/record selection (filter/wrapper/embedded, stratified sampling), PCA, and discretization. Embedded images are missing ("No se encontró … .png").
- `Apuntes Clustering/Tema 1 Intro/` — introduction, dimensionality reduction and PCA (relevant to advanced challenge 2).
- `Apuntes Clustering/Tema 2 Clustering/` — T2-1 clustering intro, T2-2 K-Means, T2-3 Mean-Shift, T2-4 DBSCAN, T2-5 hierarchical, T2-6 self-organizing maps (plus `SOM( Ejemplos).pptx`), T2-7 cluster validation (elbow/silhouette, relevant to challenges 3 and advanced 1).
- `Apuntes Python/` — introductory slides by Ezequiel López-Rubio (UMA) on `Python.pdf`, `Pandas.pdf`, `Matplotlib.pdf` and `Sklearn.pdf` (estimator API, validation, models, metrics). They assume Jupyter/Google Colab (`%matplotlib inline`).
- `Apuntes Python/Numpy.pdf` is **truncated/corrupted** (missing EOF, unreadable by `pdftotext`); ask the user for a fresh copy if it's needed.

Text can be extracted with `pdftotext -layout <file> -` (poppler is installed).

## Conventions

- Deliverable artifacts (notebooks, summaries, segment names) are evaluated by Spanish-speaking judges; the brief and data dictionary are in Spanish.
- Teams are of 4 people.
