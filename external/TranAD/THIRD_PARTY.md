# Third-party code: TranAD

**This directory contains third-party code. It is not original work of the FinAnomX project.**

| Item | Value |
|---|---|
| Upstream repository | https://github.com/imperial-qore/TranAD |
| Pinned commit | `7ffb98d0c18189cc3d9ab732b4cb0278200a0af0` (2023-09-13, default-branch HEAD at time of verification; upstream has no tags) |
| License | BSD 3-Clause License, Copyright (c) 2022, Shreshth Tuli (see [`LICENSE`](LICENSE), copied verbatim) |
| Paper | *TranAD: Deep Transformer Networks for Anomaly Detection in Multivariate Time Series Data*, VLDB 2022 (Tuli, Casale, Jennings) |

## Files included and verification status

Only the minimal set of files imported by the FinAnomX pipeline is included. `src/05_train_models.py`, `src/06_infer_anomalies.py` and `src/07_xai_numeric.py` load `src.models` and `src.dlutils` dynamically, and those import `src.constants`, `src.folderconstants` and `src.parser`. Each file was compared against the pinned commit using its git blob SHA-1.

| File | Status |
|---|---|
| `LICENSE` | identical |
| `src/models.py` | identical |
| `src/dlutils.py` | identical |
| `src/constants.py` | identical |
| `src/folderconstants.py` | identical |
| `src/parser.py` | identical |

**No modifications have been made to these files.** The FinAnomX-specific adaptation (windowing, feature set, training and inference) lives in this repository's `src/` directory. `src/models.py` imports `dgl`, so FinAnomX installs a minimal stub for `dgl` at runtime and only uses the `TranAD` model class.

The upstream benchmark datasets (`data/`, ~648 MB), results and scripts are intentionally not redistributed. To obtain the full repository:

```bash
git clone https://github.com/imperial-qore/TranAD.git
git -C TranAD checkout 7ffb98d0c18189cc3d9ab732b4cb0278200a0af0
# then point FinAnomX at it:  TRANAD_REPO_DIR=/path/to/TranAD
```
