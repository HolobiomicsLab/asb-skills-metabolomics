---
name: r-package-workflow-execution
description: Use when reproducing a published R-package analysis from raw tabular
  data (e.g. log2-scaled metabolomics intensities with batch levels in the first column)
  into diagnostic plots (PCA, scree) plus per-sample coordinate CSVs.
when_to_use_negative:
- Data has already been batch-corrected — the Figure 1A raw-data clustering signal
  (samples clustered by batch order) is specifically a raw-data diagnostic; applying
  this skill to corrected data would not reproduce it.
- Input has >2000 features (variables) — the README recommends the dbnorm unsupervised-plotting
  functions only for datasets with fewer than 2000 features for computational speed.
- Missing-value imputation or batch-effect correction is the goal (e.g. emvd/emvf
  imputation or lowess/ber/ComBat correction) — those are separate skills; this skill
  stops at raw explorative diagnostics.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: dbnorm
  role: R package providing the explorative PCA/scree visualization pipeline used
    to detect across-batch drift in the raw SKIPOGH data
  repo: https://github.com/NBDZ/dbnorm
- name: R
  role: Runtime environment in which dbnorm and its dependencies (factoextra, ggplot2)
    are executed to generate the PDF plots and CSV outputs
- name: factoextra
  role: Dependency used within dbnorm for PCA visualization rendering in the Figure
    1A reproduction
- name: ggplot2
  role: Dependency used within dbnorm for graphical inferences of the PCA scores and
    scree plots
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:a3c7bb484c786cd5281095b836bdc8bc457f42aea2e317b6ae0c9b31d10b7e4d
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
yes_rate: 0.6
n_claims: 10
---

# r-package-workflow-execution

## Summary

Execute a published R-package workflow end-to-end (here: dbnorm) to reproduce a paper analysis — in this case the PCA of raw SKIPOGH plasma metabolomics data (1079 samples × 239 metabolites, 11 batches) that reveals clustering by analytical batch order, i.e. Figure 1A of Bararpour et al. 2021.

## When to use

Use when reproducing a published R-package analysis from raw tabular data (e.g. log2-scaled metabolomics intensities with batch levels in the first column) into diagnostic plots (PCA, scree) plus per-sample coordinate CSVs.

## When NOT to use

- Data has already been batch-corrected — the Figure 1A raw-data clustering signal (samples clustered by batch order) is specifically a raw-data diagnostic; applying this skill to corrected data would not reproduce it.
- Input has >2000 features (variables) — the README recommends the dbnorm unsupervised-plotting functions only for datasets with fewer than 2000 features for computational speed.
- Missing-value imputation or batch-effect correction is the goal (e.g. emvd/emvf imputation or lowess/ber/ComBat correction) — those are separate skills; this skill stops at raw explorative diagnostics.

## Inputs

- Raw SKIPOGH log2-scaled intensities matrix: .csv with 1079 samples in rows, 239 metabolites in columns, batch levels in the first column (11 analytical batches)
- R environment with dbnorm installed plus dependencies (factoextra, ggplot2, sva, ber, limma, pcaMethods, etc. per README)

## Outputs

- PCA scores plot (samples colored by 11 batch levels) reproducing the raw-data panel of Figure 1A, saved as PDF in the working directory
- Accompanying Scree plot PDF in the working directory
- Per-sample PC coordinates exported as CSV (one row per sample) written to the Rtmpe temporary folder

## How to apply

Load the raw SKIPOGH log2-scaled intensities CSV with 1079 samples in rows and 239 metabolites in columns, batch levels in the first column, then call the dbnorm Visdbnorm function with the correction step disabled so only the raw-data unsupervised diagnostics are produced. Visodbnorm runs PCA on the raw data and compiles PCA and Scree plots into a PDF in the working directory using factoextra/ggplot2, while writing three .csv files (including per-sample PC coordinates) into a temporary 'Rtmpe' folder. For a true raw-only reproduction of Figure 1A, apply dbnorm only for explorative analysis and avoid emvd/emvf missing-value imputation or any of the batch-correction models (lowess, ber, parametric/nonparametric ComBat) — the paper found >50% of variability was explained by batch for most metabolites in this raw state. Validate that the PCA plot reproduces the paper's observation that samples cluster by batch order, and that the PC-coordinate CSV contains one row per sample.

## Related tools

- **dbnorm** (R package providing the explorative PCA/scree visualization pipeline used to detect across-batch drift in the raw SKIPOGH data) — https://github.com/NBDZ/dbnorm
- **R** (Runtime environment in which dbnorm and its dependencies (factoextra, ggplot2) are executed to generate the PDF plots and CSV outputs)
- **factoextra** (Dependency used within dbnorm for PCA visualization rendering in the Figure 1A reproduction)
- **ggplot2** (Dependency used within dbnorm for graphical inferences of the PCA scores and scree plots)

## Examples

```
Rscript -e 'library(dbnorm); data <- read.csv("skipogh_raw.csv", header=T, row.names=1); Visdbnorm(data)'
```

## Evaluation signals

- The emitted PCA plot must reproduce Figure 1A: samples clustered by the 11 batch order levels (i.e. the raw-data batch-effect signal)
- The PC-coordinate CSV must contain one row per sample (1079 rows for the SKIPOGH run)
- A Scree plot PDF must also be present in the working directory alongside the PCA PDF
- Corrected-data .csv files (lowess/ber/ComBat) should NOT be produced for a raw-only reproduction — only raw diagnostics
- Input CSV must have 239 metabolite columns plus a leading batch column, matching the raw SKIPOGH matrix shape

## Limitations

- dbnorm version for the published workflow is not specified in the article — behavior may differ across package versions; README states V-0.2.2.
- README requires all data be 'normalized and scaled on the log2' with batch levels in the first column; deviations in scaling or column order break the function's assumptions.
- Visodbnorm writes outputs into a temporary Rtmpe folder whose location varies by OS ('C:\Users\%USERNAME%\AppData\Local\Tem'), which must be located to retrieve the CSV files.
- The README notes the batch-correction/visualization functions are suggested for less than 2000 features; running on much larger datasets may be slow.
- Installation requires non-CRAN archive versions of ber, NormalizeMets, and metabolomics via devtools::install_version, which may fail on modern R versions.

## Evidence

- [results] PCA of raw SKIPOGH data showed samples clustered by batch order: "The Principal Component Analysis (PCA) plot of raw data generated by “dbnorm” showed that the samples were clustered by batch order (Fig. 1A)."
- [readme] Visodbnorm PCA/scree output description: "Graphical check such as *PCA* plot and *Scree* plot compiled into a **PDF** (saved in the working directory) and three **.csv** files (saved in a folder, intiate with *Rtmpe*"
- [readme] Input format requirement: "Data to be uploaded must be normalized and scaled on the log2 to account for the high abundance features (variables) by which technical heterogeneity might be overlooked. The input data must be in"
- [results] >50% batch variability finding: "with > 50% of the variability explained by batch for most metabolites (Fig. 1B and Supplementary Table 1)"
- [readme] Feature-count limit caveat: "This function is suggested for less than 2000 features (variables)."
