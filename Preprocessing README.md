# BAM Preprocessing: Reproducibility Notes

## Overview

This folder contains the preprocessing pipeline for the Bicultural Acculturation Measure (BAM) analysis using the Latino National Survey (LNS) 2006 dataset.

## File Structure

```
preprocessing/
├── bam_preprocessing.qmd    # Documentation of preprocessing steps
├── README.md                # This file
└── data/
    └── processed/
        └── lns_with_clusters_k4.rda  # Canonical preprocessed data (USE THIS)
```

## Important: Reproducibility Limitations

### Why Exact Replication Is Not Possible

The preprocessing pipeline includes **Multiple Imputation by Chained Equations (MICE)** using Predictive Mean Matching (PMM). This method is inherently stochastic—even with identical seeds, results vary due to:

1. **R version differences** — Random number generation algorithms change between versions
2. **Package version differences** — The `mice` package implementation evolves
3. **Platform differences** — ARM vs x86, macOS vs Linux vs Windows

### What This Means

| If you run... | You will get... |
|---------------|-----------------|
| `bam_preprocessing.qmd` | Different imputed values than the original |
| GMM on freshly imputed data | Different cluster assignments |
| GMM on preserved `lns_with_clusters_k4.rda` | **Identical** cluster assignments (ARI = 1.0) |

## The Solution: Preserved Data Artifact

The file `lns_with_clusters_k4.rda` contains:

- **The exact imputed values** from the original September 2023 analysis
- **The original cluster assignments** (`cluster_assignment_k4` column)
- **All 27 variables** needed for downstream analysis

This file is the **reproducibility artifact**. Using it guarantees:

✓ Exact cluster sizes: 433 / 705 / 378 / 3,269  
✓ ARI = 1.0 when re-fitting GMM with same parameters  
✓ Identical downstream analyses  

## How to Use

### For Replication (Recommended)

```r
# Load the preserved dataset
load("data/processed/lns_with_clusters_k4.rda")

# The clusters are already assigned
table(lns_subset$cluster_assignment_k4)
#   1    2    3    4 
# 433  705  378 3269 

# Use these for all downstream analyses
```

### For Understanding the Pipeline

Review `bam_preprocessing.qmd` to understand:
- How variables were recoded
- Which variables were imputed
- The MICE parameters used (m=5, maxit=20, method='pmm', seed=500)

**Do not re-run the preprocessing** expecting identical results.

## GMM Clustering Parameters

For reference, the GMM was fit with:

```r
set.seed(2500)
gmm <- Mclust(X_scaled, G = 4, modelNames = "EEV")
```

Where:
- `X_scaled` = scaled matrix of VARS5 (AMERICAN, CULTURAL_IDENTITY, KEEPSPAN, DISTINCT, LEARNENG)
- `G = 4` = four clusters
- `modelNames = "EEV"` = ellipsoidal, equal volume and shape

## Cluster Interpretations

| Cluster | Label | n | Characteristics |
|---------|-------|---|-----------------|
| 1 | Culture Affirming | 433 | High cultural identity, moderate American identity, lower English emphasis |
| 2 | Assimilationist | 705 | High American identity, lower cultural maintenance, high English emphasis |
| 3 | Demicultural | 378 | Moderate identities, lowest Spanish maintenance, high distinctiveness |
| 4 | Bicultural | 3,269 | High on both American and cultural identity, high language maintenance |

## Data Source

Latino National Survey, 2006  
ICPSR Study 20862  
https://www.icpsr.umich.edu/web/ICPSR/studies/20862

## Original Analysis

- **Date**: September 2023
- **R Version**: ~4.1.x
- **Platform**: macOS

## Contact

Jessala A. Grijalva  
[Your institution/email if desired]

---

*This README accompanies the BAM preprocessing documentation for the PGI/GitHub submission.*
