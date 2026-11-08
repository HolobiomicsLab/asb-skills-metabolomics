---
name: unsupervised-multivariate-exploratory-analysis
description: Use when a metabolomics matrix (.csv, samples in rows, batch levels in
  column 1) needs an unsupervised PCA/scree check of whether samples cluster by analytical
  batch before selecting a correction model.
when_to_use_negative:
- Input data has already been batch-corrected (e.g. by ber, bagging ber or parametric
  ComBat) and only residual checks are needed
- The dataset has more than ~2000 features, where Visdbnorm is not recommended for
  computational speed
- The goal is supervised differential analysis (e.g. Log2FC > 1.5, adj-p < 0.05 volcano
  plot), not unsupervised batch diagnosis
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3520
tools:
- name: dbnorm
  role: R package that runs the unsupervised PCA/scree diagnostics on the raw data
    via its Visdbnorm function before correction
  repo: https://github.com/NBDZ/dbnorm
- name: factoextra
  role: Plotting library used to render the PCA and scree plots
- name: ggplot2
  role: Plotting library used to render the PCA scores plot colored by batch
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:7318da23b24e76034aac83af509168357e670e87daadb4e73b1bf0c4b4507a64
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NBDZ/dbnorm
  tools:
  - R
  - dbnorm
  - factoextra
  - ggplot2
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# unsupervised-multivariate-exploratory-analysis

## Summary

Exploratory PCA and scree-plot diagnostics on raw metabolomics data (e.g. SKIPOGH, 1079 samples × 239 log2-scaled metabolites across 11 batches) to reveal batch-driven sample clustering before choosing a correction model.

## When to use

Use when a metabolomics matrix (.csv, samples in rows, batch levels in column 1) needs an unsupervised PCA/scree check of whether samples cluster by analytical batch before selecting a correction model.

## When NOT to use

- Input data has already been batch-corrected (e.g. by ber, bagging ber or parametric ComBat) and only residual checks are needed
- The dataset has more than ~2000 features, where Visdbnorm is not recommended for computational speed
- The goal is supervised differential analysis (e.g. Log2FC > 1.5, adj-p < 0.05 volcano plot), not unsupervised batch diagnosis

## Inputs

- log2-scaled metabolomics intensity matrix in .csv format with batch levels in the first column (e.g. SKIPOGH raw data: 1079 samples × 239 metabolites)
- batch level annotations (11 analytical batches)

## Outputs

- PCA scores plot with samples colored by batch level (PDF)
- Scree plot (PDF)
- CSV file of per-sample PC coordinates (one row per sample)

## How to apply

Load the log2-scaled data matrix with batch levels in the first column and run the dbnorm Visdbnorm function with correction disabled so only unsupervised diagnostics are emitted. Extract per-sample PC coordinates from the PCA and inspect the PCA scores plot for clustering by the 11 analytical batch levels, as in Figure 1A of the SKIPOGH study, where raw samples clustered by batch order. Complement the PCA with a scree plot and, optionally, per-feature adjusted-R2 statistics showing >50% of variability explained by batch for most metabolites. Use factoextra/ggplot2 to save the PCA and scree plots as a PDF in the working directory; the diagnosis (batch clustering present) justifies proceeding to batch-effect correction models like ber or parametric ComBat.

## Related tools

- **dbnorm** (R package that runs the unsupervised PCA/scree diagnostics on the raw data via its Visdbnorm function before correction) — https://github.com/NBDZ/dbnorm
- **factoextra** (Plotting library used to render the PCA and scree plots)
- **ggplot2** (Plotting library used to render the PCA scores plot colored by batch)

## Examples

```
library(dbnorm); Visdbnorm(raw_data)  # raw_data: .csv with batch in col 1
```

## Evaluation signals

- The PCA scores plot must reproduce Figure 1A: samples cluster by batch order in raw data
- The PC coordinates CSV contains exactly one row per sample (1079 rows for the SKIPOGH dataset)
- A scree plot accompanies the PCA, both saved as PDF files in the working directory
- Per-feature adjusted-R2 statistics show >50% of variability explained by batch for most metabolites

## Limitations

- Visdbnorm is recommended for datasets with fewer than 2000 features for computational speed
- Raw-data PCA only diagnoses the batch effect; correction (e.g. via lowess, ber or parametric ComBat) must follow
- PCA of raw data can miss subtle within-batch drift that per-feature PDF/violin plots reveal

## Evidence

- [results] PCA reveals batch clustering in raw data: "The Principal Component Analysis (PCA) plot of raw data generated by “dbnorm” showed that the samples were clustered by batch order (Fig. 1A)."
- [readme] dbnorm diagnostics and outputs: "Graphical check such as *PCA* plot and *Scree* plot compiled into a **PDF** (saved in the working directory)"
- [readme] Input format and batching: "The input data must be in **.csv** format with the independent experiments in the rows and the features (variables) in the columns, with the `batch` levels considered in the first column."
- [readme] Feature-count guidance: "This function is suggested for less than 2000 features (variables)."
- [results] Batch variability magnitude: "with > 50% of the variability explained by batch for most metabolites"
