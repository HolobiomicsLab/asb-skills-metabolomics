---
name: lcms-quality-metric-interpretation
description: 'Use when targeted LC–MS peak integration output must be quality-filtered:
  peaks carry computed metrics (AUC, max intensity, POP, SNR, bell-curve similarity)
  and ''Good''/''Bad''/''Ambiguous'' calls must be evaluated or reproduced.'
when_to_use_negative:
- The goal is untargeted peak detection/preprocessing with tools like xcms, MZmine
  or OpenMS — this skill only covers quality assessment of already-integrated targeted
  peaks
- No labeled ('Good'/'Bad'/'Ambiguous') peak-quality annotations exist to train or
  evaluate the classifier
- Only raw vendor LC–MS files are available without TARDIS-integrated peaks and computed
  quality metrics
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3474
tools:
- name: TARDIS
  role: Generates the peak-quality metrics (AUC, maximum intensity, POP, custom SNR,
    bell-curve similarity) that feed the random-forest classification
  repo: https://github.com/UGent-LIMET/TARDIS
- name: R
  role: Environment for training the random-forest quality classifiers and computing
    variable importance via the deposited scripts
  repo: https://github.com/UGent-LIMET/TARDIS
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:77b6a861cc76dc630c40f125c1929fc1c027388249ad51c26a3894afc059e159
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lcms-quality-metric-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interpret and rank TARDIS chromatographic peak-quality metrics (AUC, maximum intensity, points over the peak, custom signal-to-noise ratio, bell-curve similarity) and combine them into a random-forest peak-quality classification for LC–MS metabolomics data.

## When to use

Use when targeted LC–MS peak integration output must be quality-filtered: peaks carry computed metrics (AUC, max intensity, POP, SNR, bell-curve similarity) and 'Good'/'Bad'/'Ambiguous' calls must be evaluated or reproduced.

## When NOT to use

- The goal is untargeted peak detection/preprocessing with tools like xcms, MZmine or OpenMS — this skill only covers quality assessment of already-integrated targeted peaks
- No labeled ('Good'/'Bad'/'Ambiguous') peak-quality annotations exist to train or evaluate the classifier
- Only raw vendor LC–MS files are available without TARDIS-integrated peaks and computed quality metrics

## Inputs

- Labeled peak-quality-metric table from the tardis_publication_code deposit (AUC, maximum intensity, points over the peak, custom signal-to-noise ratio, bell-curve similarity per target per run)
- Biofluid matrix labels (stool, saliva, urine) for single-matrix vs multimatrix model training
- R scripts from the tardis_publication_code repository

## Outputs

- Random-forest classifier objects for each matrix and the pooled multimatrix model
- Accuracy, precision and balanced-accuracy metrics (targets: 59.45% overall, 98.57% 'Bad' precision, 70% 'Ambiguous' balanced accuracy)
- Random-forest variable-importance ranking of the five quality metrics
- peak_quality_rf_results.csv with reproduced metrics
- Regenerated classification figures

## How to apply

Load the labeled peak-quality-metric dataset from the UGent-LIMET/tardis_publication_code deposit (data downloadable from Zenodo) containing the five TARDIS metrics per target per run across stool, saliva and urine matrices. Train single-matrix random-forest classifiers per biofluid and a multimatrix pooled classifier in R using the deposited scripts, then evaluate against the reported targets: 59.45% overall accuracy on FGFP metabolomics, 98.57% precision on 'Bad' predictions with zero 'Good' peaks labeled 'Bad', and 70% balanced accuracy for FGFP 'Ambiguous'. Compute random-forest variable importance and confirm the expected ranking: custom SNR and peak shape (bell-curve similarity) outrank AUC and points over the peak. The key interpretation rule from the paper is that the multimatrix classifier's 'Bad' calls are highly trustworthy even when overall accuracy is modest, so use 'Bad' predictions for downstream peak filtering rather than overall accuracy as the quality gate.

## Related tools

- **TARDIS** (Generates the peak-quality metrics (AUC, maximum intensity, POP, custom SNR, bell-curve similarity) that feed the random-forest classification) — https://github.com/UGent-LIMET/TARDIS
- **R** (Environment for training the random-forest quality classifiers and computing variable importance via the deposited scripts) — https://github.com/UGent-LIMET/TARDIS

## Examples

```
R: BiocManager::install("pablovgd/TARDIS", build_vignettes = FALSE, dependencies = TRUE); library(TARDIS)
```

## Evaluation signals

- Multimatrix RF overall accuracy on FGFP metabolomics reproduces ~59.45%
- 'Bad'-prediction precision is ~98.57% with zero 'Good' peaks classified as 'Bad'
- Balanced accuracy on FGFP 'Ambiguous' peaks is ~70%
- Variable-importance plot shows SNR and peak shape (bell-curve similarity) above AUC and points over the peak
- Reproduced metrics written to peak_quality_rf_results.csv match the deposited script output

## Limitations

- Overall multimatrix accuracy (~59.45%) is modest; only the 'Bad' class is considered highly reliable for filtering decisions
- Metrics were validated on three biofluids (stool, saliva, urine) across two LC–MS instruments; other matrices/instruments are untested in this context
- No TARDIS package version number is stated in the discussion section, so exact reproducibility may depend on the repository state

## Evidence
