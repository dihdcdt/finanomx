# Third-party code: PatchTST

**This directory contains third-party code. It is not original work of the FinAnomX project.**

| Item | Value |
|---|---|
| Upstream repository | https://github.com/yuqinie98/PatchTST |
| Pinned commit | `204c21efe0b39603ad6e2ca640ef5896646ab1a9` (2023-08-11, default-branch HEAD at time of verification; upstream has no tags) |
| License | Apache License 2.0 (see [`LICENSE`](LICENSE), copied verbatim) |
| Authors | Yuqi Nie, Nam H. Nguyen, Phanwadee Sinthong, Jayant Kalagnanam |
| Paper | *A Time Series is Worth 64 Words: Long-term Forecasting with Transformers*, ICLR 2023 |

## Files included and verification status

Only the minimal set of files imported by the FinAnomX pipeline (`src/05_train_models.py`, `src/06_infer_anomalies.py` and `src/07_xai_numeric.py` load `models.PatchTST` dynamically) is included. Each file was compared against the pinned commit using its git blob SHA-1.

| File | Status |
|---|---|
| `LICENSE` | identical |
| `PatchTST_supervised/models/PatchTST.py` | identical |
| `PatchTST_supervised/layers/PatchTST_backbone.py` | identical |
| `PatchTST_supervised/layers/PatchTST_layers.py` | identical |
| `PatchTST_supervised/layers/RevIN.py` | identical |

**No modifications have been made to these files.** The FinAnomX-specific adaptation (data windowing, training loop, calibration and inference) lives in this repository's `src/` directory. It uses the upstream model unchanged.

All other upstream content (self-supervised code, other baseline models, scripts, images) is intentionally not redistributed. To obtain the full repository:

```bash
git clone https://github.com/yuqinie98/PatchTST.git
git -C PatchTST checkout 204c21efe0b39603ad6e2ca640ef5896646ab1a9
# then point FinAnomX at it:  PATCHTST_REPO_DIR=/path/to/PatchTST
```
