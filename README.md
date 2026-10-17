# PH-CVT: probabilistic hybrid CNN–Transformer U-Net for brain tumour segmentation

Code for the manuscript *"Multi-cohort evaluation of an uncertainty-aware hybrid convolutional–transformer network for glioma segmentation"* (submitted to *Scientific Reports*).

PH-CVT U-Net is a 2D multimodal (T1, T1ce, T2, FLAIR) residual CNN–Transformer with gated fusion, ASPP, deep supervision, an auxiliary tumour-detection head and Monte Carlo (MC) dropout uncertainty. It is benchmarked against U-Net, ResU-Net, Attention U-Net, TransUNet and Swin-Unet on BraTS 2021. It is then tested zero-shot on two external cohorts: BraTS-Africa and UCSF-PDGM lower-grade glioma.

## Main results (from the notebooks in this repository)

**Internal test, BraTS 2021** (188 held-out patients, full native-resolution volumes, single models, mean of 5 seeds)

| Model | Mean Dice | WT | TC | ET | HD95 (mm) | Params (M) | Latency A100 (ms/slice) |
|---|---|---|---|---|---|---|---|
| PH-CVT | 0.8755 | 0.9162 | 0.8761 | 0.8343 | 9.72 | 17.94 | 10.08 |
| U-Net | 0.8786 | 0.9164 | 0.8824 | 0.8369 | 10.44 | 7.76 | 3.48 |
| ResU-Net | 0.8784 | 0.9164 | 0.8822 | 0.8366 | 9.06 | 8.11 | 4.32 |
| Attention U-Net | 0.8791 | 0.9157 | 0.8857 | 0.8360 | 10.35 | 7.85 | 5.65 |

- **Equivalence:** PH-CVT is statistically equivalent to every CNN baseline within ±0.01 Dice (paired Wilcoxon, Holm-corrected, patient-level bootstrap), although each baseline is slightly higher.
- **Failure detection:** MC-dropout mutual information detects failed segmentations (mean Dice < 0.75) with AUROC 0.897 (95% CI 0.82–0.96).
- **Uncertainty under shift, BraTS-Africa** (94 cases, same scoring rule as internal): AUROC 0.811 (0.70–0.91). The drop from internal is not significant (+0.087, P = 0.16).
- **UCSF-PDGM WHO grade 2–3** (98 cases, zero-shot):
  - Whole-tumour Dice is 0.888 for PH-CVT and 0.881–0.884 for the baselines.
  - PH-CVT has the fewest false enhancing-tumour calls in non-enhancing tumours: 18.4% vs 20.7–23.0%.
- **Detection head** (all 29,140 test slices): ROC-AUC 0.988, sensitivity 0.949, specificity 0.939.
- **Speed:** MC dropout with T = 12 passes batched takes 11.75 ms per slice, against 16.48 ms for a 5-member U-Net ensemble (NVIDIA A100-40GB).

## Repository layout

```
notebooks/
  01_PHCVT_2D_BraTS2021_study.ipynb        training (sessions A/B), FINAL analysis, Parts K (uncertainty, same rule
                                           on both cohorts) and L (all-slice detection, measured MC latency)
  02_UCSF_PDGM_selective_download.ipynb    downloads only the needed UCSF-PDGM v5 files from TCIA (Aspera CLI)
  03_UCSF_PDGM_LGG_curation.ipynb          selects WHO grade 2–3 cases and exports them in BraTS layout
  04_PHCVT_2D_UCSF_LGG_external_eval.ipynb zero-shot evaluation of the frozen models on UCSF-PDGM LGG
data/splits/                               fixed patient-level split used by every model (see README there)
results/                                   summary tables reported in the paper
```

## Data (not redistributed)

| Dataset | Source | Use |
|---|---|---|
| BraTS 2021 Task 1 (1,251 patients) | RSNA-ASNR-MICCAI BraTS 2021, https://www.synapse.org/brats2021 (Kaggle mirror `dschettler8845/brats-2021-task1`) | training, validation, internal test (875/188/188) |
| BraTS-Africa glioma (95 cases; 94 complete) | The Cancer Imaging Archive; Kaggle mirror `shahriyarhasib/brats-africa-dataset` (`95_Glioma`) | external test 1 |
| UCSF-PDGM v5, WHO grade 2–3 (99 curated; 98 evaluated) | The Cancer Imaging Archive, UCSF-PDGM | external test 2 |

No UCSF-PDGM grade 2–3 patient has a BraTS 2021 identifier in the UCSF-PDGM metadata (`BraTS21 ID` column), so there is no patient overlap with the training data.

## How to reproduce (Google Colab)

1. **Secrets.** In Colab Secrets, add your Kaggle API token as `kaggle_API` and enable notebook access. Mount Google Drive when asked. Outputs go to `MyDrive/AI_Brain_Tumor/`.
2. **Train** with `01_PHCVT_2D_BraTS2021_study.ipynb`. Set `SESSION = "A"` in one runtime and `"B"` in another; training resumes after disconnects.
3. **Analyse** with the same notebook: set `SESSION = "FINAL"` and `ANALYSIS_SEEDS = None` (all five seeds), then Run all on one GPU type. This produces every internal table and figure, then Parts K and L.
   - Part K downloads BraTS-Africa from Kaggle automatically.
4. **UCSF-PDGM:** run notebook `02` (paste the TCIA Faspex public link), then `03` to curate grade 2–3, then `04` for the external evaluation.

Every analysis cell is resumable and writes its tables and figures to Drive under `paper_outputs/`.

**Hardware used:** NVIDIA A100 (40/80 GB) for training and timing, NVIDIA T4 for some evaluations. Software: Python 3.13, PyTorch 2.11 (CUDA 13.0).

## Trained weights

The checkpoints (5 seeds × 4 architectures, 3 seeds for TransUNet, Swin-Unet and the ablations) and per-patient result tables are archived at **[Zenodo DOI — to be added]**.

## Citation

See `CITATION.cff`. Please also cite the BraTS 2021, BraTS-Africa and UCSF-PDGM dataset papers.

## License

MIT (see `LICENSE`). The datasets keep their own licences and terms of use.
