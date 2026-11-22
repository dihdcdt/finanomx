# Data, Artifacts and Licensing Notes

FinAnomX is **artifacts-first**: every stage of the pipeline writes its outputs to disk, and the dashboard reads only those files. This repository ships the **code** and a small set of **aggregate result tables**. It does not ship raw market data, scraped news, generated per-day artifacts or trained model files. This document explains why, and how to regenerate each one.

## 1. Data sources

| Source | Used for | Access | Redistribution note |
|---|---|---|---|
| Yahoo Finance (via `yfinance`) | Daily OHLCV for TSLA, AMZN, NVDA, MSFT (2018–2024) | No key | Yahoo's terms restrict redistribution of its data. **Not included.** |
| FRED, Federal Reserve Bank of St. Louis (via `fredapi`) | VIXCLS, DGS10, FEDFUNDS, CPIAUCSL | Free API key (`FRED_API_KEY`) | Some series are owned by third parties (e.g., VIX by Cboe). **Not included.** |
| SEC EDGAR submissions API | Earnings-date proxies (8-K Item 2.02) | `SEC_USER_AGENT` header required by SEC fair-access policy | Public data. The derived date list is included. |
| FRED release calendar / Federal Reserve | CPI release and FOMC dates | Same as FRED | Derived date list is included. |
| GDELT DOC API + publisher websites | News articles around detected anomaly peaks | No key | Article text is copyrighted by the publishers. **Not included.** |

## 2. What is included

| Path | Content | Why it is safe to include |
|---|---|---|
| `data/external/event_seed_earnings.csv` | Ticker, earnings-event date, SEC source URL, note (40 rows) | Public calendar facts |
| `data/external/event_seed_macro.csv` | CPI/FOMC event dates and source (42 rows) | Public calendar facts |
| `results/model_comparison*.csv`, `results/eval_detection*.csv` | Detection metrics (AUROC, AUPRC, P/R/F1, P@K, confusion counts) | Aggregate numbers only |
| `results/eval_explain.csv` | Explanation-layer summary metrics | Aggregate numbers only |
| `results/xai_feature.csv`, `results/xai_temporal.json` | Feature and time-step attributions for the 25 episode peaks | Derived indicator values for 25 dates; no OHLCV series |
| `results/xai_faithfulness.csv`, `results/xai_stability.csv` | Deletion-test and date-jitter results | Derived scores only |
| `results/event_windows.csv` | Event windows built from the seed files | Public calendar facts |
| `results/event_counterfactual.csv` | Retrieval-query ablation results for the 25 peaks: generated queries, **links** to the top-ranked article, scores | Contains URLs only, no article text |
| `results/sample_anomalies.csv` | 62 rows (flagged days and episode peaks, 2023–2024): ticker, date, split, fused score, adaptive threshold/flag, episode fields, subtype | No prices, volumes or text |

`results/` is a **read-only snapshot** of the experiment reported for the course. The pipeline itself writes to `data/results/`, which is git-ignored.

## 3. What is excluded, and how to regenerate it

| Excluded artifact | Reason | Regenerate with | Output |
|---|---|---|---|
| `data/raw/raw_merged.parquet` | Yahoo/FRED data | `python src/01_ingest_raw.py` | `data/raw/` |
| `data/processed/features.parquet`, `scaled.parquet` | Derived from Yahoo/FRED data | `python src/02_process_data.py` | `data/processed/` |
| `models/scalers/*.pkl` | Contain statistics of the Yahoo/FRED data; pickle files are unsafe to load from untrusted sources | `python src/02_process_data.py` | `models/scalers/` |
| `data/external/*.csv` (refresh) | Included, but can be rebuilt | `python src/03_build_event_windows.py` | `data/external/`, `data/results/event_windows.csv` |
| Chronos-2 / baseline scores | Derived per-day data | `python src/04_baselines.py` | `data/results/` |
| `models/dl/patchtst_forecaster.pth`, `tranad_detector.pth` | Trained on Yahoo/FRED-derived features; redistribution status of derived weights not confirmed | `python src/05_train_models.py` | `models/dl/` |
| `models/baselines/*.pkl` (iForest, OCSVM, PCA, DLinear) | Pickles of models fit on the same data (~12 MB) | Not produced by the documented 00–11 run order; these come from an earlier baseline stage | — |
| `data/results/anomalies_base.csv` | Per-day prices and scores | `python src/06_infer_anomalies.py` | `data/results/` |
| XAI outputs | Included as a snapshot | `python src/07_xai_numeric.py` | `data/results/xai_*` |
| `data/results/news_cache/`, `news_enriched.parquet` | **Full scraped article text** (copyrighted) | `python src/08_news_on_demand.py` | `data/results/` |
| `data/results/event_attribution.csv` | Contains article titles and evidence excerpts | `python src/09_event_grounding.py` | `data/results/` |
| Evaluation tables | Included as a snapshot | `python src/10_evaluate.py` | `data/results/` |
| `data/results/anomalies.csv`, `chat_evidence.jsonl` | Contain OHLCV values and article summaries and excerpts | `python src/11_pack_for_web.py` | `data/results/` |
| Upstream TranAD benchmark datasets (~648 MB) and upstream images | Not used by FinAnomX; third-party data | See `external/*/THIRD_PARTY.md` | — |

Re-running the pipeline downloads current data from live services. Results may therefore differ slightly from the snapshot in `results/`, for example if Yahoo revises historical data or GDELT returns different articles.

## 4. Environment variables

Copy `.env.example` to `.env` and fill in your own values. **Never commit `.env`.**

| Variable | Required | Purpose |
|---|---|---|
| `FRED_API_KEY` | Yes (step 01) | FRED API access. Get a free key from the FRED website. |
| `SEC_USER_AGENT` | Yes (step 03) | SEC requires a descriptive User-Agent with contact info, e.g. `FinAnomX <name> <email>` |
| `OLLAMA_BASE_URL` | Yes for step 09 | Local Ollama server (default `http://localhost:11434`) |
| `OLLAMA_EMBED_MODEL` | Yes for step 09 | Embedding model for evidence ranking (default `nomic-embed-text:latest`) |
| `OLLAMA_LLM_MODEL` | Optional | Local LLM for chatbot phrasing (default `deepseek-r1:8b`); without it, the chatbot uses deterministic answers |
| `PATCHTST_REPO_DIR`, `TRANAD_REPO_DIR` | Optional | Use a full upstream clone instead of the vendored subset in `external/` |
| `DEVICE`, `N_JOBS`, `STEP05_*`, `PATCHTST_*`, `TRANAD_*`, `CHRONOS_*` | Optional | Runtime and hyperparameter overrides (see `config.py`) |
