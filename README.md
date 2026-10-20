# PH-CVT: probabilistic hybrid CNN–Transformer U-Net for brain tumour segmentation

Code for the manuscript *"Hybrid Convolutional–Transformer Glioma Segmentation: Architectural Equivalence and Uncertainty Reliability Under External Distribution Shift"* (submitted to *Scientific Reports*).

PH-CVT U-Net is a 2D multimodal (T1, T1ce, T2, FLAIR) residual CNN–Transformer with gated fusion, ASPP, deep supervision, an auxiliary tumour-detection head and Monte Carlo (MC) dropout uncertainty. It is benchmarked against U-Net, ResU-Net, Attention U-Net, TransUNet and Swin-Unet on BraTS 2021. It is then tested zero-shot on two external cohorts: BraTS-Africa and UCSF-PDGM lower-grade glioma.

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.23434518.svg)](https://doi.org/10.5281/zenodo.23434518)

## Main results (from the notebooks in this repository)

**Internal test, BraTS 2021** (188 held-out patients, full native-resolution volumes, single models, mean of 5 seeds)

| Model | Mean Dice | WT | TC | ET | HD95 (mm) | Params (M) | Latency A100 (ms/slice) |
|---|---|---|---|---|---|---|---|
| PH-CVT | 0.8755 | 0.9162 | 0.8761 | 0.8343 | 9.72 | 17.94 | 10.08 |
| U-Net | 0.8786 | 0.9164 | 0.8824 | 0.8369 | 10.44 | 7.76 | 3.48 |
| ResU-Net | 0.8784 | 0.9164 | 0.8822 | 0.8366 | 9.06 | 8.11 | 4.32 |
| Attention U-Net | 0.8791 | 0.9157 | 0.8857 | 0.8360 | 10.35 | 7.85 | 5.65 |

- **Equivalence:** PH-CVT is statistically equivalent to every CNN baseline within ±0.01 Dice (paired patient-level bootstrap, 10,000 resamples, Holm-corrected), although each baseline is slightly higher.
- **Failure detection:** MC-dropout mutual information detects failed segmentations (mean Dice < 0.75) with AUROC 0.862 ± 0.033 (mean ± SD over 5 seeds).
- **Uncertainty under shift, BraTS-Africa** (94 cases, same scoring rule as internal): AUROC 0.804 ± 0.033 (5 seeds). The per-seed drop from internal is 0.01–0.09, and every seed's 95% CI includes zero.
- **UCSF-PDGM WHO grade 2–3** (98 cases, zero-shot):
  - Whole-tumour Dice is 0.888 for PH-CVT and 0.881–0.884 for the baselines.
  - PH-CVT has the fewest false enhancing-tumour calls in non-enhancing tumours: 18.4% vs 20.7–23.0%.
- **Detection head** (all 29,140 test slices): ROC-AUC 0.988, sensitivity 0.949, specificity 0.939.
- **Speed:** MC dropout with T = 12 passes batched takes 11.75 ms per slice, against 16.48 ms for a 5-member U-Net ensemble (NVIDIA A100-40GB).

## Repository layout

```
notebooks/
  01_PHCVT_2D_BraTS2021_study.ipynb       main study. SESSION = "A"/"B": training of seeds 31415/27182;
                                           "FINAL": all internal tables and figures, Parts K (uncertainty, same rule
                                           on both cohorts) and L (all-slice detection, MC latency);
                                           "AFRICA5": Part M (BraTS-Africa, five seeds, external ablation M4)
  02_UCSF_PDGM_selective_download.ipynb   downloads only the needed UCSF-PDGM v5 files from TCIA
  03_UCSF_PDGM_LGG_curation.ipynb         selects WHO grade 2-3 cases and exports them in BraTS layout
  04_PHCVT_2D_UCSF_LGG_external_eval.ipynb zero-shot evaluation on UCSF-PDGM grade 2-3
  05_PHCVT_2D_PartN_additional_analyses.ipynb  SESSION = "UPDATE": bootstrap P values, full-volume ablation,
                                           MC dropout for all five seeds and three cohorts, Suppl. Fig. S2
  06_PHCVT_2D_original_training_seeds42-123-2026_GradCAMpp.ipynb  original training of every model on seeds
                                           42/123/2026 and Grad-CAM++ at three network depths (cell 13b v3)
  07_alert_rate_sensitivity_cpu.ipynb     alert-rate sensitivity of MC-dropout failure detection (CPU)
scripts/flops_count.py                    GFLOPs per 160 x 160 slice
data/splits/patient_level_split_1251.csv  fixed patient-level split (875 / 188 / 188) used by every model
results/                                  result tables reported in the paper (see results/README.md)
```

## Where each result in the paper comes from

| Paper item | Notebook (session / part) | Result file |
|---|---|---|
| Table 1 (selected-slice, 3-seed models), Suppl. Table S1, Suppl. Fig. S1 | 06, then 01 `FINAL` | `results/table3a_paired_selected_slice.csv` |
| Table 1 (full volume, 5 seeds, ensembles, latency), Table 2, Fig. 2, Suppl. Fig. S3 | 01 `FINAL` | `results/internal_full_volume_patient_rows.csv`, `results/table3b_paired_full_volume.csv` |
| Table 2 bootstrap P values, Suppl. Table S8 | 05 (N6) | `results/partN/N6_table2_bootstrap_10000.csv` |
| Suppl. Table S1 full-volume ablation | 05 (N3) | `results/partN/N3_*.csv` |
| Table 3, Fig. 3, Suppl. Table S7 (BraTS-Africa) | 01 `AFRICA5` (M1, M2) | `results/brats_africa/` |
| External ablation without transformer and fusion | 01 `AFRICA5` (M4) | `results/brats_africa/PHCVT_BraTSAfrica_ablation_no_transformer_paired.csv` |
| Fig. 4, Suppl. Tables S3, S4, Fig. S4 (MC dropout) | 01 `FINAL` (Part 4, Part K) | `results/brats_africa_phcvt_s42_roi_uncertainty.csv` |
| Suppl. Table S5 (alert rate) | 07 | `results/PHCVT_K4_alert_rate_sensitivity.csv` |
| Suppl. Table S9 (all five seeds) | 05 (N4, N5) | `results/partN/N4_*.csv`, `N5_*.csv` |
| Suppl. Note 1, detection head, MC latency | 01 `FINAL` (Part L), 05 (N2) | `results/partN/N2_small_tumour_slices.csv` |
| Suppl. Table S2 (UCSF-PDGM) | 02, 03, 04 | `results/ucsf_pdgm_lgg_patient_rows.csv` |
| Suppl. Note 2, Table S6 (Grad-CAM++) | 06 (cell 13b v3) | written to `extended_studies/gradcam_pp_v3/` on Drive |
| Suppl. Fig. S2 | 05 (N7) | figure |
| GFLOPs column of Table 1 | `scripts/flops_count.py` | `results/PHCVT_2D_flops_per_slice.csv` |

Part 5 of notebook 01 (OpenNeuro ds007045) is not used in the paper.

## Data (not redistributed)

| Dataset | Source | Use |
|---|---|---|
| BraTS 2021 Task 1 (1,251 patients) | RSNA-ASNR-MICCAI BraTS 2021, https://www.synapse.org/brats2021 (Kaggle mirror `dschettler8845/brats-2021-task1`) | training, validation, internal test (875/188/188) |
| BraTS-Africa glioma (95 cases; 94 complete) | The Cancer Imaging Archive; Kaggle mirror `shahriyarhasib/brats-africa-dataset` (`95_Glioma`) | external test 1 |
| UCSF-PDGM v5, WHO grade 2–3 (99 curated; 98 evaluated) | The Cancer Imaging Archive, UCSF-PDGM | external test 2 |

No UCSF-PDGM grade 2–3 patient has a BraTS 2021 identifier in the UCSF-PDGM metadata (`BraTS21 ID` column), so there is no patient overlap with the training data.

## How to reproduce (Google Colab)

1. **Secrets.** In Colab Secrets, add your Kaggle API token as `kaggle_API` and enable notebook access. Mount Google Drive when asked. Outputs go to `MyDrive/AI_Brain_Tumor/`.
2. **Train.** Seeds 42, 123 and 2026 of every model and ablation were trained with notebook `06`. Seeds 31415 and 27182 of PH-CVT, PH-CVT-Lite and the three CNNs were trained with notebook `01`: set `SESSION = "A"` in one runtime and `"B"` in another; training resumes after disconnects.
3. **Analyse** with the same notebook: set `SESSION = "FINAL"` and `ANALYSIS_SEEDS = None` (all five seeds), then Run all on one GPU type. This produces every internal table and figure, then Parts K and L.
   - Part K downloads BraTS-Africa from Kaggle automatically.
   - For the five-seed BraTS-Africa results (Table 3) set `SESSION = "AFRICA5"` and Run all; it stops by itself after cell M4.
4. **UCSF-PDGM:** run notebook `02` (paste the TCIA Faspex public link), then `03` to curate grade 2–3, then `04` for the external evaluation.
5. **Additional analyses:** run notebook `05` with `SESSION = "UPDATE"` (inference and statistics only), and `07` on a CPU runtime.

Every analysis cell is resumable and writes its tables and figures to Drive under `paper_outputs/`.

**Hardware used:** NVIDIA A100 (40/80 GB) for training and timing, NVIDIA T4 for some evaluations. Software: Python 3.13, PyTorch 2.11 (CUDA 13.0).

## Trained weights

The checkpoints (5 seeds × 4 architectures, 3 seeds for TransUNet, Swin-Unet and the ablations) and per-patient result tables are archived at Zenodo: [https://doi.org/10.5281/zenodo.23434518](https://doi.org/10.5281/zenodo.23434518).

## Citation

If you use this code, please cite the article:

> Hameed N, Khan TF, Hussain M, Saeed S, Rauf A. Hybrid Convolutional–Transformer Glioma Segmentation: Architectural Equivalence and Uncertainty Reliability Under External Distribution Shift. *Scientific Reports* (2026).

and the archived code and weights: https://doi.org/10.5281/zenodo.23434518. Machine-readable metadata is in `CITATION.cff`.

Please also cite the datasets:

- **BraTS 2021:** Baid U, et al. The RSNA-ASNR-MICCAI BraTS 2021 benchmark on brain tumor segmentation and radiogenomic classification. arXiv:2107.02314 (2021).
- **BraTS-Africa:** Adewole M, et al. The Brain Tumor Segmentation (BraTS) Challenge 2023: Glioma segmentation in Sub-Saharan Africa patient population (BraTS-Africa). arXiv:2305.19369 (2023).
- **UCSF-PDGM:** Calabrese E, et al. The University of California San Francisco Preoperative Diffuse Glioma MRI dataset. *Radiology: Artificial Intelligence* 4(6):e220058 (2022). https://doi.org/10.1148/ryai.220058

## License

MIT (see `LICENSE`). The datasets keep their own licences and terms of use.
