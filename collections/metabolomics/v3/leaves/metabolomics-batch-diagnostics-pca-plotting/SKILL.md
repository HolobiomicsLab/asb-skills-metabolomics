---
name: metabolomics-batch-diagnostics-pca-plotting
description: Use when raw targeted or untargeted LC-MS metabolomics data must be checked
  for analytical batch effects before choosing a correction model, e.g. samples appear
  grouped by run order.
when_to_use_negative:
- Data have already been batch-corrected and you need to compare correction models
  via adjusted-R2 scoring rather than raw-data diagnostics
- Dataset has more than ~2000 features, where dbnorm functions are not suggested for
  computational speed
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: dbnorm
  role: R package providing the Visodbnorm/Visdbnorm function that computes the PCA,
    scree plot, and batch-colored scores plot on the raw data matrix
  repo: https://github.com/NBDZ/dbnorm
- name: R
  role: Environment in which dbnorm is installed and executed
- name: factoextra
  role: Used to produce the PCA visualization of sample clustering by batch
- name: ggplot2
  role: Underlying plotting engine for the PCA and scree PDF outputs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:9c600ddaf918dd2f0281eca37703e77a971bbc4d94e6663070ce077ff4f83a54
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# metabolomics-batch-diagnostics-pca-plotting

## Summary

An unsupervised diagnostic step from the dbnorm R package that runs PCA on raw log2-scaled metabolomics data and plots sample scores colored by analytical batch to expose across-batch signal drift. In the SKIPOGH dataset (1079 samples, 239 metabolites, 11 batches) it revealed samples clustered by batch order.

## When to use

Use when raw targeted or untargeted LC-MS metabolomics data must be checked for analytical batch effects before choosing a correction model, e.g. samples appear grouped by run order.

## When NOT to use

- Data have already been batch-corrected and you need to compare correction models via adjusted-R2 scoring rather than raw-data diagnostics
- Dataset has more than ~2000 features, where dbnorm functions are not suggested for computational speed

## Inputs

- .csv data matrix with samples in rows, metabolite features in columns, and batch levels in the first column
- log2-normalized/scaled metabolomics intensities (dbnorm requires log2 scaling to account for high-abundance features)

## Outputs

- PCA scores plot (PDF) with samples colored by batch level
- Scree plot (PDF)
- CSV of per-sample PC coordinates (one row per sample)

## How to apply

Load the data as a .csv with samples in rows, metabolites in columns, and batch levels in the first column (README specifies the input format for dbnorm). Run the dbnorm Visodbnorm function with correction disabled so only the unsupervised diagnostics are produced: it computes a PCA of the raw data, extracts per-sample PC coordinates, and generates the scores plot plus a scree plot as a PDF saved in the working directory. Color samples by the batch levels (e.g. the 11 analytical batches of SKIPOGH) to visually assess drift; the article shows that PCA of raw SKIPOGH data clustered samples by batch order, confirming a strong batch effect. Use factoextra/ggplot2 for plotting. Correct application means the emitted PCA plot reproduces batch-ordered clustering and the PC coordinates file has one row per sample.

## Related tools

- **dbnorm** (R package providing the Visodbnorm/Visdbnorm function that computes the PCA, scree plot, and batch-colored scores plot on the raw data matrix) — https://github.com/NBDZ/dbnorm
- **R** (Environment in which dbnorm is installed and executed)
- **factoextra** (Used to produce the PCA visualization of sample clustering by batch)
- **ggplot2** (Underlying plotting engine for the PCA and scree PDF outputs)

## Examples

```
data <- read.csv("path/to/mydata.csv", sep=",", header=T, row.names=1); library(dbnorm); Visodbnorm(data)
```

## Evaluation signals

- Emitted PCA scores plot reproduces the paper's Figure 1A pattern: raw SKIPOGH samples cluster by batch order (11 batch levels)
- PC coordinates CSV contains exactly one row per sample (1079 rows for SKIPOGH)
- PCA and scree plot are compiled into a PDF saved in the working directory per dbnorm's documented output behavior
- Input matrix respects the dbnorm format: batch levels in first column, features in columns, log2-scaled values

## Limitations

- dbnorm function names and outputs vary by version; some cards refer to Visdbnorm while the README names Visodbnorm
- The README states these dbnorm functions are suggested for fewer than 2000 features for computational speed
- Requires log2-scaled data; the README notes raw data must be normalized and scaled on log2 before upload to account for high-abundance features
- dbnorm-specific details such as the exact version used in the paper are not documented in the article

## Evidence

- [results] The Principal Component Analysis (PCA) plot of raw data generated by “dbnorm” showed that the samples were clustered by batch order (Fig. 1A).: "The Principal Component Analysis (PCA) plot of raw data generated by “dbnorm” showed that the samples were clustered by batch order"
- [readme] The input data must be in .csv format with the independent experiments in the rows and the features (variables) in the columns, with the batch levels considered in the first column.: "The input data must be in .csv format with the independent experiments in the rows and the features (variables) in the columns"
- [readme] Data to be uploaded must be normalized and scaled on the log2 to account for the high abundance features (variables) by which technical heterogeneity might be overlooked.: "Data to be uploaded must be normalized and scaled on the log2 to account for the high abundance features (variables)"
- [readme] Graphical check such as PCA plot and Scree plot compiled into a PDF (saved in the working directory): "Graphical check such as *PCA* plot and *Scree* plot compiled into a **PDF** (saved in the working directory)"
- [readme] This function is suggested for less than 2000 features (variables).: "This function is suggested for less than 2000 features (variables)."
