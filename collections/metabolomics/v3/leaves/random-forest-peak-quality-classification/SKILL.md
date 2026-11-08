---
name: random-forest-peak-quality-classification
description: Use when TARDIS-integrated targeted LC–MS peaks across biofluid matrices
  (stool, saliva, urine) have labeled quality classes and you need to reproduce or
  evaluate the RF 'Bad'-peak classifier and its variable-importance ranking.
when_to_use_negative:
- Peaks have not been integrated and quality metrics computed yet — run TARDIS peak
  integration and quality-metric extraction first
- The task is generic supervised classification on arbitrary features rather than
  chromatographic peak quality filtering in targeted LC–MS data
- Unlabeled peak data without Good/Bad/Ambiguous quality annotations is all that is
  available, so the RF model cannot be trained or its accuracy evaluated
edam_operation: http://edamontology.org/operation_2990
tools:
- name: TARDIS
  role: Source of the peak quality metrics (AUC, maximum intensity, points over the
    peak, custom SNR, bell-curve similarity) that form the classifier's feature space
  repo: https://github.com/UGent-LIMET/TARDIS
- name: R
  role: Environment for training and evaluating the single-matrix and multimatrix
    random-forest classifiers using the deposited scripts
- name: tardis_publication_code
  role: Provides the R scripts and labeled dataset used to reproduce the RF classification
    accuracy and variable-importance results
  repo: https://github.com/UGent-LIMET/tardis_publication_code
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:2eb195cbb21a68d2e72a6bf3723f0162b0beaad1012704956f4f291e5b82b478
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/UGent-LIMET/TARDIS
  repo_url: https://github.com/UGent-LIMET/TARDIS
  tools:
  - R
  - TARDIS
  - tardis_publication_code
  edam_operation: http://edamontology.org/operation_2990
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# random-forest-peak-quality-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains single-matrix and multimatrix random-forest classifiers on TARDIS peak quality metrics (AUC, maximum intensity, points over the peak, custom signal-to-noise ratio, bell-curve similarity) to identify 'Bad' chromatographic peaks in targeted LC–MS metabolomics, evaluating against the reported FGFP accuracy targets.

## When to use

Use when TARDIS-integrated targeted LC–MS peaks across biofluid matrices (stool, saliva, urine) have labeled quality classes and you need to reproduce or evaluate the RF 'Bad'-peak classifier and its variable-importance ranking.

## When NOT to use

- Peaks have not been integrated and quality metrics computed yet — run TARDIS peak integration and quality-metric extraction first
- The task is generic supervised classification on arbitrary features rather than chromatographic peak quality filtering in targeted LC–MS data
- Unlabeled peak data without Good/Bad/Ambiguous quality annotations is all that is available, so the RF model cannot be trained or its accuracy evaluated

## Inputs

- Labeled peak quality-metric table (AUC, maximum intensity, points over the peak, custom signal-to-noise ratio, bell-curve similarity) with Good/Bad/Ambiguous labels
- Data from the FGFP cohort study runs (stool, saliva, urine matrices)
- R classification scripts from the UGent-LIMET/tardis_publication_code deposit (Zenodo doi 10.5281/zenodo.14548032)

## Outputs

- peak_quality_rf_results.csv with reproduced accuracy/precision metrics
- Random-forest variable-importance ranking of the five quality metrics
- Regenerated classification figures (Bad-peak prediction quality per matrix)

## How to apply

Load the labeled peak quality-metric dataset (AUC, maximum intensity, points over the peak, custom signal-to-noise ratio, bell-curve similarity) from the UGent-LIMET/tardis_publication_code deposit on Zenodo. In R (>= 4.4.0 with BiocManager 3.20), train single-matrix random-forest classifiers for each biofluid matrix (stool, saliva, urine) using the deposited classification scripts, then train the pooled multimatrix classifier. Evaluate the multimatrix model against the reported FGFP targets: overall accuracy 59.45%, 'Bad'-prediction precision 98.57% with zero 'Good' peaks labeled 'Bad', and 70% balanced accuracy on the FGFP 'Ambiguous' class. Compute random-forest variable importance and confirm that custom SNR and peak shape metrics outrank AUC and points over the peak (POP), reflecting the paper's validation of the Kumler et al. quality metrics. Regenerate the reported classification figures and write the reproduced metrics to peak_quality_rf_results.csv.

## Related tools

- **TARDIS** (Source of the peak quality metrics (AUC, maximum intensity, points over the peak, custom SNR, bell-curve similarity) that form the classifier's feature space) — https://github.com/UGent-LIMET/TARDIS
- **R** (Environment for training and evaluating the single-matrix and multimatrix random-forest classifiers using the deposited scripts)
- **tardis_publication_code** (Provides the R scripts and labeled dataset used to reproduce the RF classification accuracy and variable-importance results) — https://github.com/UGent-LIMET/tardis_publication_code

## Examples

```
BiocManager::install("pablovgd/TARDIS",build_vignettes = FALSE, dependencies=TRUE)
```

## Evaluation signals

- Multimatrix model overall accuracy on FGFP metabolomics reproduces 59.45%
- 'Bad'-prediction precision is 98.57% and no 'Good' peaks are classified as 'Bad'
- Balanced accuracy on the FGFP 'Ambiguous' class is 70%
- Variable importance ranks custom SNR and peak shape metrics above AUC and points over the peak (POP)
- Reproduced metrics match the values reported in the deposited classification scripts and figures

## Limitations

- Overall multi-class accuracy is modest (59.45%); the classifier's strength is specifically high-precision 'Bad' detection rather than accurate classification of all quality classes
- Requires labeled peak-quality training data; performance depends on matrix-specific single-matrix models, so behavior on matrices outside stool/saliva/urine is unverified
- R version >= 4.4.0 and BiocManager 3.20 are required, and the TARDIS package's example data can trigger connection-timeout issues (increase options(timeout = '300'))

## Evidence

- [readme] This repository contains all R and python scripts to reproduce the data-analysis and figures presented in Vangeenderhuysen et al. 2025.: [evidence span withheld — anchor status: external_doc]
- [readme] Make sure `R` (**version >= 4.4.0**) is installed on your computer: [evidence span withheld — anchor status: external_doc]
