# Tumor Classification from Gene Expression (TCGA Pan-Cancer)

> **TL;DR** Multiclass classifier predicting 32 tumor types from RNA-Seq expression profiles. PCA latent factors plus multinomial logistic regression reach **94.7% accuracy and 0.926 macro-F1** on a 1,678-sample held-out test set.

**Author:** Ryan Lindberg · UC San Diego, Halıcıoğlu Data Science Institute · [rylindberg@ucsd.edu](mailto:rylindberg@ucsd.edu)

## Overview

|  |  |
|---|---|
| Dataset | [TCGA Pan-Cancer expression data](https://figshare.com/articles/dataset/TCGA_Pan-Cancer_expression_and_mutation_data_for_Project_Cognoma/3487685) (Project Cognoma curation) |
| Samples | 8,388 tumor samples |
| Features | 16,261 genes per sample (RNA-Seq, log-transformed counts) |
| Classes | 32 tumor types (BRCA, GBM, LUAD, READ, COAD, PRAD, etc.) |
| Task | Multiclass classification of tumor type from a single expression vector |
| Train/test | Stratified 80/20 split — 1,678 test samples |
| Validation | K-fold cross-validation with grid search over PCA components and L2 regularization strength |

## Method

1. **Standardize** the gene expression matrix per gene (zero mean, unit variance).
2. **PCA** to compress 16,261 genes into a small number of latent factors (k chosen via CV).
3. **Multinomial logistic regression** on the latent factors, trained with L2-regularized cross-entropy.

Compared against two baselines:

- **Majority class** — always predicts the most frequent tumor type. Establishes the performance floor under heavy class imbalance.
- **Logistic regression on a high-variance gene subset** — simple linear classifier on hand-selected features. Tests whether dimensionality reduction actually adds value over crude feature selection.

## Results

| Model | Accuracy | Macro-F1 |
|---|---:|---:|
| Majority class | 0.094 | 0.005 |
| Logistic regression on high-variance gene subset | 0.926 | 0.887 |
| **PCA + multinomial logistic regression** (this work) | **0.947** | **0.926** |

Several tumor types are essentially solved by this pipeline (GBM, LGG, OV, PCPG, TGCT, THCA, THYM, UVM all reach F1 ≈ 1.0). Anatomically related cancers (COAD vs READ, ESCA, UCS) remain harder to distinguish, with per-class F1 in the 0.5–0.8 range. The simpler baseline lands close behind the main model, so the value of PCA-based latent factors over a hand-picked subset is real but modest at this scale.

## Caveats

- Performance reflects within-TCGA evaluation; external validation on independent cohorts and sequencing pipelines is required before any clinical use.
- PCA latent factors are linear combinations of thousands of genes, which limits per-prediction interpretability.
- The dataset has selection bias toward TCGA-style patient demographics, geography, and sequencing protocols.

## Reproducing

```bash
git clone https://github.com/lindbergryan04/gene-expression-prediction.git
cd gene-expression-prediction
jupyter notebook main.ipynb
```

Data files are tracked in `data/`:

- `expressionmatrix.tsv.bz2` — gene expression matrix (16,261 genes × 8,388 samples)
- `samples.tsv` — clinical metadata (`sample_id`, `acronym`, `disease`, `age_diagnosed`, `gender`)

Dependencies: Python 3.10+, `pandas`, `numpy`, `scikit-learn`, `matplotlib`, `seaborn`.

## Related work

Cancer-type classification from gene expression is a well-studied benchmark task. For comparison:

- [Li et al. (2017)](https://pubmed.ncbi.nlm.nih.gov/28673244) — KNN with genetic-algorithm gene selection on 32 tumor types from TCGA, achieving >90% accuracy and noting that anatomically-related tumors (e.g., READ vs COAD) are intrinsically hard to separate.
- [Divate et al. (2022)](https://pmc.ncbi.nlm.nih.gov/articles/PMC8909043) — deep neural network on pan-cancer expression, >97% accuracy.
- [Yap et al. (2020)](https://arxiv.org/abs/1909.04169) — explainable CNNs with biomarker attribution.

This project deliberately uses simple methods (PCA + linear classifier) to see how much of the discriminative signal can be captured by a teaching-grade pipeline. The answer turns out to be most of it.
