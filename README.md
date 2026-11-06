# FLIC Food Leftover Mass Estimation: Reproduction and Research Extensions

Reproduction of the digital-density baseline from the FLIC dataset paper, plus five extensions: density shift, enrollment strategy, a paired before/after estimator, automatic course classification and conformal prediction intervals.

> **Base paper:** Piccoli, F. et al. *FLIC: A Real-World Dataset for Visual Estimation of Food Leftovers in Canteens.* Applied Sciences 2026, 16, 5465.
> The FLIC dataset belongs to its authors and is **not included** in this repository.

## Abstract

Food leftovers in collective catering are difficult to quantify automatically because the remaining food is visually heterogeneous, small in mass, and changes appearance after consumption. FLIC is a real-world dataset of 401 paired full and leftover tray acquisitions collected over 22 days, with pixel-precise semantic annotations and physically measured food mass.

This project first reproduces the published digital-density baseline, which estimates mass from the relationship between food-pixel area and measured mass. The notebook then investigates five research questions: density shifts between full and leftover states, enrollment using a few measured leftover plates, a paired before/after estimator, automatic broad course classification, and calibrated prediction intervals.

The reproduction is close to the published results for several settings. Density shift is associated with baseline error, especially for heterogeneous foods such as chicken upper thigh and pasta with vegetables. The paired estimator does not consistently improve on the baseline, and automatic course classification improves with colour features but adds downstream mass error. Conformal intervals reach roughly the intended 90% coverage overall, although coverage varies across food courses.

## Research questions

| ID | Question | Method |
|---|---|---|
| RQ0 | Can the published baseline be reproduced? | Digital density, `mass = delta × area`, 100 random enrollment runs |
| RQ1 | How much does density change between full and leftover plates? | Separate `delta` per state, shift ratio and implied bias |
| RQ2 | Should enrollment include a few leftover weighings? | Full-only, leftover-only and mixed enrollment (F = 3, 10) |
| RQ3 | Does paired before/after imagery help? | Fraction-remaining estimator with shrinkage α, tuned on development days |
| RQ4 | Can the human course-selection step be automated? | Random Forest on position, shape and colour, GroupKFold by day |
| RQ5 | Can the estimate come with reliable intervals? | Split conformal prediction, 90% nominal coverage |

## Data

| Item | Value |
|---|---|
| Acquisitions | 401 full + 401 leftover trays (802 images, 640 × 480 RGB) |
| Collection | 22 days, June–July 2024, canteen of a children's summer camp in Italy |
| Annotations | Pixel-precise semantic masks (36 classes) |
| Ground truth | Physically measured food mass |
| Items in the reproduced analysis set | 1,160 (the paper reports 1,166) |
| Mask-area cross-check | 2,696 tray-course-state areas compared, 99.96% exact agreement |

## Results

**RQ0: Baseline reproduction (MAE-1, grams)**

| Setting | Notebook | Paper | Relative gap |
|---|---|---|---|
| No classification, all | 16.662 | 14.300 | +16.516% |
| Broad, all | 12.336 | 13.040 | −5.402% |
| Broad, first | 13.001 | 12.000 | +8.346% |
| Broad, second | 14.377 | 20.810 | −30.911% |
| Broad, side | 18.899 | 16.190 | +16.731% |
| Broad, bread | 2.864 | 3.430 | −16.495% |
| Fine, all | 11.849 | 11.470 | +3.304% |
| Fine, first | 13.098 | 12.380 | +5.800% |
| Fine, second | 11.929 | 13.600 | −12.289% |

**RQ1: Density shift (representative dishes)**

| Dish | n | delta full | delta leftover | shift |
|---|---|---|---|---|
| Gratin-style plaice fillets | 23 | 0.011 | 0.000 | 0.035 |
| Pasta Bolognese style | 45 | 0.012 | 0.004 | 0.291 |
| Chicken upper thigh | 21 | 0.017 | 0.007 | 0.385 |
| Pasta with vegetables | 20 | 0.014 | 0.006 | 0.402 |

Absolute implied bias is positively associated with baseline MAE-1 (Spearman ρ = 0.73, n = 32 dishes). This correlation is partly mechanical, because both quantities depend on how much food is left.

**RQ3: Paired estimator on held-out days**
- No consistent improvement at broad level (mean gain between −0.04 and −0.18 g).
- Fine-level differences are tiny, and the selected α was 0.75 or 1.0, so the method often reduces to the baseline.

**RQ4: Automatic course classification (1,924 regions)**

| Features | Accuracy | Macro-F1 |
|---|---|---|
| Position + shape | 0.638 | 0.641 |
| Position + shape + colour | 0.779 | 0.775 |

Downstream MAE: item-oracle 9.972 g, region-oracle 10.195 g, pooled no-classification 12.840 g, **automatic classification 16.975 g**. Classification errors propagate into mass estimation.

**RQ5: Conformal prediction intervals (90% target)**

| Variant | Mean coverage | Mean width (g) |
|---|---|---|
| Global, absolute | 0.892 | 40.659 |
| Global, normalized | 0.901 | 46.861 |
| Per-course, normalized | 0.892 | 44.440 |

Coverage varies by course, so calibration should be monitored rather than assumed.

## Limitations

- FLIC comes from a single site with a fixed menu, which limits generalisation.
- The dataset is small compared with large generic food datasets.
- Mixed dishes cause region-merging and attribution problems.
- The notebook is a reconstruction of the baseline, so small gaps from the published numbers remain (random enrollment and data-mapping details).

## How to run

The notebook is written for Google Colab.

1. Download the FLIC files from the dataset release linked in the base paper: `FLIC_grams.csv`, `FLIC_menu.csv` and the four zip files.
2. Upload them to a Google Drive folder, for example `MyDrive/FLIC`.
3. Open `flic_leftover_mass_estimation.ipynb` in Colab and edit the configuration cell:
   ```python
   DATA_MODE = "real"
   REAL_ROOT = Path("/content/drive/MyDrive/FLIC")
   ROUTE = "FLIC"
   ```
4. Run all cells (**Runtime → Run all**). Result tables are saved to `flic_outputs/`.

**No dataset? Run the self-test.** Set `DATA_MODE = "synthetic"` to run the full pipeline on generated fake canteen data. These numbers only check that the code works and are not reportable results.

Local run:
```bash
pip install -r requirements.txt
jupyter notebook flic_leftover_mass_estimation.ipynb
```

## Repository contents

| File | Description |
|---|---|
| `flic_leftover_mass_estimation.ipynb` | Full notebook with outputs: data checks, baseline, RQ1–RQ5 |
| `FLIC_project_report.pdf` | Project report with methodology, results and discussion |
| `requirements.txt` | Python dependencies |

## Reference

Piccoli, F.; Callegaro, D.; Marelli, D.; Buzzelli, M.; Franchini, C.; Stella, L.; Bianco, S.; Ciocca, G.; Schettini, R.; Scazzina, F. *FLIC: A Real-World Dataset for Visual Estimation of Food Leftovers in Canteens.* Applied Sciences 2026, 16, 5465.

## Author

Krithi Shiva · M.Sc. Data Science, SASTRA University
[GitHub](https://github.com/KrithiShiva) · [LinkedIn](https://linkedin.com/in/krithishiva)
