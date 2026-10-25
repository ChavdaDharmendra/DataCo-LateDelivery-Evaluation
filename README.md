# DataCo Late-Delivery Risk: Leakage-Aware Chronological Evaluation

Code, saved models, predictions and result files for the paper:

> J. Chavda, D. Chavda, K. Gohil, and H. Chauhan, "How Much Does Gradient Boosting Add? A Leakage-Aware Chronological Evaluation of Late-Delivery Risk Prediction on the DataCo Supply Chain Dataset," manuscript under review, 2026.

**Main finding.** After removing outcome-derived fields and using a chronological, order-grouped split, a locked 16-feature XGBoost model reaches independent-test ROC-AUC 0.758. A 96-cell lookup table of training-period late rates by *shipping mode × order hour* reaches the same ROC-AUC (paired difference +0.0007, 95% CI −0.006 to 0.008).

## Repository layout

| Path | Contents |
|---|---|
| `src/reproduce_paper_results.py` | Recomputes every new statistic and all 7 figures in the paper from the dataset and the saved files (no retraining) |
| `src/pipeline_stages1to3/` | Earlier pipeline scripts (preprocessing, ANN experiments, leakage reference) |
| `src/stage4_training/` | Training code for CatBoost, LightGBM, XGBoost, ablation, tuning and threshold analysis (see its README) |
| `results/stage4_classification/` | Saved models (`.json`, `.txt`, `.cbm`), model lock, saved test predictions, all stage-4 reports |
| `results/stage3_leakage_audit/`, `results/ann_v3_*` | ANN models and reports (secondary experiments) |
| `results/stage5_*`, `results/stage6_*` | Evidence, claims, feature-timing and literature audits |
| `figures/` | Figures exactly as used in the manuscript |
| `data/` | Place the dataset here (not redistributed; see `data/README.md`) |
| `SHA256SUMS.txt` | Checksums of every file in this repository |

## Quick start

```bash
pip install -r requirements.txt
# download the dataset into data/ (see data/README.md), then:
python src/reproduce_paper_results.py --data data/DataCoSupplyChainDataset.csv --results results --out reproduced
```

The script first checks the dataset SHA-256 (`fa6d022ed437155e1a2f0378710602848703c8a7f203f7ff5d77805bf8480aa6`), then writes `reproduced/stats.json` and the figures. Expected key values:

| Quantity | Value |
|---|---|
| Test ROC-AUC, locked XGBoost | 0.7582 |
| Test ROC-AUC, mode × hour lookup | 0.7575 |
| Validation ROC-AUC, re-scored baseline / tuned model | 0.7619 / 0.7569 |
| Validation confusion matrix of the shipping rule | [[11587, 1660], [6530, 9099]] |

Runtime is a few minutes on a laptop; the order-cluster bootstrap (1,000 resamples) is the slowest step.

## Protocol summary

* Partition by each order's first timestamp: training < 27 Feb 2017 ≤ validation < 15 Aug 2017 ≤ test.
* Excluded outcome/post-shipment fields: Days for shipping (real), Delivery Status, shipping date, Benefit per order, Order Profit Per Order, Order Item Profit Ratio.
* Model and threshold (0.50) locked and hashed (`stage4_step11_model_lock.json`) before the test partition was scored.

## Known limitations of this package

* ANN profit-regression metrics are taken from archived reports; the row-level predictions were not recovered.
* Some earlier scripts contain absolute Colab paths (`/mnt/data`, `/content/drive`); edit them before rerunning.

## Licence and citation

Code is released under the MIT Licence (`LICENSE`). The dataset belongs to its authors (Constante, Silva, Pereira, Mendeley Data, doi:10.17632/8gx2fvg2k6.5) and must be cited separately. Please cite this work using `CITATION.cff`.
