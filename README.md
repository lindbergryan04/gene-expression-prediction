# Tumor Classification from Gene Expression

This project investigates how well we can predict tumor type directly from high-dimensional gene expression profiles. Each sample is a cancerous tumor, represented by an RNA-Seq expression vector with ~16k genes, and labeled with a tumor-type ('glioblastoma multiforme', 'lung adenocarcinoma', ' mesothelioma'). 

For the main model, we first learn a low-dimensional latent representation of tumors using a latent factor model (PCA) and then train logistic regression on these latent factors. This allows us to compress the original ~16k-dimensional gene space into a much smaller number of latent dimensions while preserving information relevant for tumor-type discrimination. As a baseline, we train multinomial logistic regression directly on gene expression (after variance-based gene filtering and standardization). 

On a held-out test set, the latent-factor + logistic regression model achieves high classification performance, with test accuracy of 94.7% and a macro-F1 of 93%, substantially outperforming trivial baselines such as majority-class prediction. Many tumor types are nearly perfectly separated in the learned latent space, while a few related or data-poor cancer types remain challenging and exhibit more confusion. Overall, the results show that a relatively simple pipeline involving latent factor representation learning plus a linear classifier is sufficient to capture much of the structure in gene expression data and to recover tumor-type labels with high accuracy, while remaining interpretable and computationally efficient.

Data Source:

Data can be found at https://figshare.com/articles/dataset/TCGA_Pan-Cancer_expression_and_mutation_data_for_Project_Cognoma/3487685

Files from figshare used in this project: expression-matrix.tsv.bz2, samples.tsv
