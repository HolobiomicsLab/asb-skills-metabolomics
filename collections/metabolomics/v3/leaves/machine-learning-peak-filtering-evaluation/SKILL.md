---
name: machine-learning-peak-filtering-evaluation
description: Use when you have TARDIS-extracted peak quality metrics for targeted
  LC–MS peaks (with Good/Bad/Ambiguous labels) across one or more biofluid matrices
  and want to train or reproduce a random-forest peak-quality classifier or assess
  its filtering reliability.
when_to_use_negative:
- Input is untargeted peak-picking output (e.g., from xcms, MZmine or OpenMS) rather
  than TARDIS targeted peak integration results with per-peak quality metrics
- Peaks lack quality labels (Good/Bad/Ambiguous), so supervised random-forest training
  cannot be performed
- The goal is vendor-style automatic integration accuracy benchmarking against TraceFinder/Xcalibur/ChromScope/SCIEX
  OS-MQ rather than ML-based quality filtering
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3174
tools:
- name: R
  role: Language in which the random-forest classifiers on peak quality metrics are
    trained and evaluated
  repo: https://github.com/UGent-LIMET/TARDIS
- name: TARDIS
  role: Open-source R package whose peak quality metrics (AUC, maximum intensity,
    POP, custom SNR, bell-curve similarity) form the classifier feature set
  repo: https://github.com/UGent-LIMET/TARDIS
- name: tardis_publication_code
  role: Deposited R and python scripts reproducing the data analysis and figures of
    Vangeenderhuysen et al. 2025, including classification scripts
  repo: UGent-LIMET/tardis_publication_code
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:22b4e6d3b36c35c81a527afbabebaffeb6ec15ad836a6be22c26d57d88151bbd
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# machine-learning-peak-filtering-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train and evaluate random-forest classifiers on TARDIS peak quality metrics (AUC, maximum intensity, points over the peak, custom signal-to-noise ratio, bell-curve similarity) to identify 'Bad' chromatographic peaks in targeted LC–MS metabolomics/lipidomics, using deposited classification scripts.

## When to use

Use when you have TARDIS-extracted peak quality metrics for targeted LC–MS peaks (with Good/Bad/Ambiguous labels) across one or more biofluid matrices and want to train or reproduce a random-forest peak-quality classifier or assess its filtering reliability.

## When NOT to use

- Input is untargeted peak-picking output (e.g., from xcms, MZmine or OpenMS) rather than TARDIS targeted peak integration results with per-peak quality metrics
- Peaks lack quality labels (Good/Bad/Ambiguous), so supervised random-forest training cannot be performed
- The goal is vendor-style automatic integration accuracy benchmarking against TraceFinder/Xcalibur/ChromScope/SCIEX OS-MQ rather than ML-based quality filtering

## Inputs

- Labeled peak quality-metric dataset (AUC, maximum intensity, points over the peak, custom signal-to-noise ratio, bell-curve similarity) with Good/Bad/Ambiguous peak labels, from Zenodo (doi=10.5281/zenodo.14548032)
- R classification scripts from the tardis_publication_code repository
- Peak metrics derived from TARDIS integration of targeted LC–MS data across stool, saliva and urine matrices

## Outputs

- Trained single-matrix and multimatrix random-forest classifiers
- Overall accuracy, per-class precision and balanced accuracy metrics (targets: 59.45% overall, 98.57% Bad precision, 70% balanced-FGFP Ambiguous accuracy)
- Random-forest variable importance ranking
- peak_quality_rf_results.csv with reproduced classification metrics
- Regenerated classification figures

## How to apply

Load the labeled peak quality-metric dataset (AUC, maximum intensity, points over the peak, custom signal-to-noise ratio, bell-curve similarity) from the UGent-LIMET/tardis_publication_code deposit. Train single-matrix random-forest classifiers per biofluid matrix (stool, saliva, urine) in R using the deposited classification scripts, then train the multimatrix model pooling all matrices. Benchmark the multimatrix model against the reported targets: 59.45% overall accuracy on FGFP metabolomics, 98.57% 'Bad'-prediction precision with zero 'Good' peaks classified as 'Bad', and 70% balanced-FGFP 'Ambiguous' accuracy. Compute random-forest variable importance and confirm that SNR and peak shape outrank AUC and points over the peak (POP), then regenerate the reported figures and write the reproduced metrics to peak_quality_rf_results.csv.

## Related tools

- **R** (Language in which the random-forest classifiers on peak quality metrics are trained and evaluated) — https://github.com/UGent-LIMET/TARDIS
- **TARDIS** (Open-source R package whose peak quality metrics (AUC, maximum intensity, POP, custom SNR, bell-curve similarity) form the classifier feature set) — https://github.com/UGent-LIMET/TARDIS
- **tardis_publication_code** (Deposited R and python scripts reproducing the data analysis and figures of Vangeenderhuysen et al. 2025, including classification scripts) — UGent-LIMET/tardis_publication_code

## Examples

```
R -e 'library(randomForest); rf <- randomForest(quality ~ AUC + maximum_intensity + points_over_peak + custom_SNR + bell_curve_similarity, data = peak_metrics)'
```

## Evaluation signals

- Multimatrix RF classifier overall accuracy on FGFP metabolomics matches the reported 59.45%
- No 'Good' peaks are classified as 'Bad' (Bad-prediction precision = 98.57%, zero false-Bad peaks)
- Balanced-FGFP 'Ambiguous' accuracy reaches 70%
- Variable importance ranks SNR and peak shape above AUC and points over the peak (POP)
- Reproduced metrics are written to peak_quality_rf_results.csv and match the reported figures

## Limitations

- Overall classifier accuracy is only 59.45%; the reported strength is precision of 'Bad' calls, not overall discrimination across all classes
- Comparison vendor software is only described generically and never named or versioned; no version number for the TARDIS R package is stated
- Biological test data used for large-scale LC-MS evaluation are not named or accessioned

## Evidence

- [readme] Reproduction scripts available: [evidence span withheld — anchor status: external_doc]
