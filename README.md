# A Hybrid CNN+BiLSTM and Genomic LM Architecture


**Author:** Koushik Govardhanam
**Course:** Capstone Project  
**University:** Auburn University

---

## Project Overview

This project predicts log₂ promoter activity across six experimental conditions using 78,677 *Arabidopsis thaliana* 170 bp promoter sequences. A hybrid deep learning architecture fuses a CNN+BiLSTM local-sequence encoder with frozen genomic language model embeddings (AgroNT or GENERator), supplemented by GC content and one-hot species encoding as auxiliary features. Six separate regression heads (one per condition) produce scalar activity predictions.

Two parallel notebooks are provided: one using **AgroNT-1b** as the frozen language model backbone, and one using **GENERator-v2-eukaryote-1.2b**, for direct head-to-head comparison.

---

## Repository Structure

```
.
├── koushik_agront_v2.ipynb     # AgroNT backbone notebook (v2)
├── koushik_generator_v2.ipynb   # GENERator backbone notebook (v2)
├── capstone_manuscript_koushik.docx                  # Written manuscript
├── capstone_presentation.pptx                      # Slides for advisor meeting
└── README.md                                       # This file
```

**Note:** Both notebooks are self-contained and designed to run end-to-end on **Google Colab (GPU runtime)**. All dependencies are installed within the notebook in Cell 1.

---

## Requirements

| Requirement | Details |
|---|---|
| Platform | Google Colab (T4 GPU recommended; A100 for faster embedding) |
| Python | 3.10+ (Colab default) |
| VRAM | ≥ 15 GB (T4 is sufficient) |
| Dataset | `Capstone project data resource-1.xlsx` (uploaded in Cell 2) |

**Python packages installed automatically in Cell 1:**

```
transformers==4.40.0
accelerate
sentencepiece
scipy
```

Standard packages already available on Colab: `torch`, `numpy`, `pandas`, `matplotlib`, `seaborn`, `sklearn`, `requests`.

---

## How to Run

### Step 1: Open in Google Colab

1. Go to [colab.research.google.com](https://colab.research.google.com)
2. Click **File → Upload notebook**
3. Upload either `koushik_agront_v2.ipynb` or `koushik_generator_v2.ipynb`

### Step 2: Enable GPU Runtime

`Runtime → Change runtime type → Hardware accelerator → T4 GPU → Save`

### Step 3: Run Cells in Order

Run all cells **top to bottom** using `Runtime → Run all`, or `Shift+Enter` cell by cell. Do **not** skip cells; each cell depends on outputs from the previous one.

| Cell | What it does |
|---|---|
| **Cell 1** | Installs packages, verifies GPU |
| **Cell 2** | Prompts you to upload the dataset `.xlsx` file |
| **Cell 3** | Loads and merges Supplementary Tables 1 & 2; splits into 6 per-condition DataFrames |
| **Cell 4** | Downloads and loads the frozen language model backbone (AgroNT or GENERator) |
| **Cell 5a** | Defines tokenization for the language model |
| **Cell 5b** | Defines one-hot encoding, GC content scalar, and species one-hot encoding functions |
| **Cell 6** | Precomputes and caches all LM embeddings (~35–40 min on T4; instant on re-run from cache) |
| **Cell 7** | Defines the `HybridPromoterModel` architecture |
| **Cell 8** | Builds per-condition datasets with 80/10/10 train/val/test splits |
| **Cell 9** | Trains 6 hybrid models (max 20 epochs, early stopping patience=5) |
| **Cell 10** | Evaluates all 6 models on the test set; prints R², Pearson r, RMSE |
| **Cell 11** | Generates performance visualizations (scatter plots, training curves) |
| **Cell 12** | In-Silico Mutagenesis (ISM): per-position importance scores |
| **Cell 12b** | Position-zone clustering and cross-condition ISM heatmap |
| **Cell 12c** | TATA-box positional insertion experiment (EUGENe Fig. 2f equivalent) |
| **Cell 12d** | In-silico sequence evolution (EUGENe Fig. 2g equivalent) |
| **Cell 12e** | Nucleotide attribution on highest-activity sequences (EUGENe Fig. 2e equivalent) |
| **Cell 12f** | CNN filter motif visualization + JASPAR CORE Plants matching |
| **Cell 12g** | Statistical JASPAR motif enrichment analysis (Fisher's exact test + Bonferroni) |
| **Cell 13** | Table 6 validation: CPE and TFBS sites |
| **Cell 13b** | Permutation feature importance (species, GC, CNN branch, LM branch) |
| **Cell 14** | Custom sequence prediction interface |
| **Cell 15** | Saves and downloads all output files |

### Step 4: After a Disconnect

If Colab disconnects mid-run, restart from **Cell 1** and re-run top to bottom. The LM embedding cache (`agro_embeddings_cache.pt` or `generator_embeddings_cache.pt`) is saved to Colab's local disk. **If the runtime has not been recycled**, the cache will be found and Cell 6 will load instantly. If the runtime was recycled, Cell 6 will recompute embeddings from scratch (~35–40 min).

---

## Model Architecture (v2)

```
Input (170 bp sequence)
       │
       ├── Branch A: One-hot (4 × 170) ──► CNN+BiLSTM ──► 256-dim
       │
       ├── Branch B: LM tokenization ──► Frozen LM Encoder ──► 1500-dim (AgroNT)
       │                                                      or 512-dim (GENERator)
       ├── GC content scalar ──────────────────────────────►   1-dim
       │
       └── Species one-hot ─────────────────────────────────►  N-dim
                                                               (N = number of unique species)
                                          │
                                   Concatenate all
                                          │
                              Fully-Connected Head (256 → 64 → 1)
                                          │
                              6 separate heads (one per condition)
```

**v2 changes from v1:**
- Added species one-hot encoding as auxiliary input
- Added GC content scalar as auxiliary input
- Fixed critical bug in Table 6 (CNN branch was receiving all-zero tensors)
- Replaced OneCycleLR with ReduceLROnPlateau (compatible with early stopping)
- Added early stopping (patience = 5)
- Added permutation feature importance analysis

---

## Experimental Conditions (6 Prediction Targets)

| # | Condition Label |
|---|---|
| 1 | No enhancer, Dark, Tobacco |
| 2 | No enhancer, Dark, Maize |
| 3 | With enhancer, Dark, Tobacco |
| 4 | With enhancer, Dark, Maize |
| 5 | No enhancer, Light, Tobacco |
| 6 | With enhancer, Light, Tobacco |

---

## Output Files

After running Cell 15, the following files are downloaded from Colab:

| File | Description |
|---|---|
| `performance.csv` | R², Pearson r, RMSE for all 6 conditions |
| `active_regions.csv` | ISM-identified high-importance subsequences |
| `table6_predictions.csv` | Table 6 CPE & TFBS predictions vs ground truth |
| `performance.png` | Bar chart of R² across conditions |
| `scatter.png` | Predicted vs actual scatter plots |
| `training_curves.png` | Train/val loss curves with early stopping markers |
| `motif_density.png` | ISM importance density across 170 bp |
| `cross_condition_importance.png` | Cross-condition ISM heatmap |
| `permutation_importance.png` | Permutation feature importance bar chart |
| `table6_cpe_heatmap.png` | CPE site heatmap |
| `table6_tata_effect.png` | TATA-box effect visualization |
| `tata_insertion_positional.png` | TATA-box positional insertion experiment |
| `attribution_top_sequences.png` | Nucleotide attribution sequence logos |
| `filter_motifs_logo.png` | CNN filter motif logos (JASPAR-matched) |
| `filter_annotation_histogram.png` | Histogram of TF family matches |
| `jaspar_enrichment.csv` | Fisher's exact test enrichment results with Bonferroni-adjusted p-values |
| `model_heads.pt` | Saved model weights for all 6 conditions |

---

## Notes

- The dataset file (`Capstone project data resource-1.xlsx`) is **not included** in this repository. Obtain it from your course materials and upload it when Cell 2 prompts you.
- The frozen LM backbone is downloaded automatically from HuggingFace in Cell 4. Ensure Colab has internet access.
- JASPAR CORE Plants profiles (~500 motifs) are fetched via the JASPAR REST API in Cell 12f. This requires internet access and takes approximately 2 minutes.
- All random seeds are fixed (`random_state=42`) for reproducibility. GPU-level non-determinism may cause minor run-to-run variation in final metrics.
