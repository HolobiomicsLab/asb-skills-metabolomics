---
name: classifier-variable-importance-analysis
description: Use when labeled chromatographic peak-quality metrics are available and
  you need to train a random-forest 'Bad'-peak classifier and rank which quality metrics
  drive the classification.
when_to_use_negative:
- Input is unlabeled peak-integration output without Good/Bad quality annotations
  — no training labels exist for the RF classifier
- The goal is raw peak integration from LC–MS files rather than post-integration quality
  classification (use TARDIS integration instead)
- Data come from a single matrix only and the question is cross-matrix generalization,
  which cannot be assessed without pooled/multimatrix training
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Language used to train the single- and multimatrix random-forest classifiers
    and compute variable importance with the deposited scripts
- name: TARDIS
  role: Source of the peak-quality metrics (AUC, maximum intensity, points over the
    peak, custom SNR, bell-curve similarity) that form classifier features
  repo: https://github.com/UGent-LIMET/TARDIS
- name: tardis_publication_code
  role: Deposit containing the R scripts and labeled datasets to reproduce the RF
    classification and figures
  repo: https://github.com/UGent-LIMET/tardis_publication_code
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:024277d03475a9550102ad8625d459f103f2057ee968fdd157d1589190cbe18a
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# classifier-variable-importance-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains random-forest classifiers on TARDIS peak-quality metrics (AUC, maximum intensity, points over the peak, custom SNR, bell-curve similarity) and computes variable importance to determine which metrics best identify 'Bad' chromatographic peaks across biofluid matrices.

## When to use

Use when labeled chromatographic peak-quality metrics are available and you need to train a random-forest 'Bad'-peak classifier and rank which quality metrics drive the classification.

## When NOT to use

- Input is unlabeled peak-integration output without Good/Bad quality annotations — no training labels exist for the RF classifier
- The goal is raw peak integration from LC–MS files rather than post-integration quality classification (use TARDIS integration instead)
- Data come from a single matrix only and the question is cross-matrix generalization, which cannot be assessed without pooled/multimatrix training

## Inputs

- Labeled peak-quality-metric dataset (AUC, maximum intensity, points over the peak, custom SNR, bell-curve similarity per peak per run) from the tardis_publication_code / Zenodo deposit
- Matrix labels (stool, saliva, urine) and peak-quality labels (Good/Bad/Ambiguous) per peak
- Deposited R classification scripts from UGent-LIMET/tardis_publication_code

## Outputs

- Random-forest classifier per matrix and a pooled multimatrix model
- Variable-importance ranking of quality metrics (SNR and bell-curve similarity vs AUC and POP)
- Classification metrics: overall accuracy, Bad-prediction precision, confusion counts (e.g., zero Good peaks labeled Bad)
- peak_quality_rf_results.csv with reproduced metrics

## How to apply

Load the labeled peak-quality dataset (AUC, maximum intensity, points over the peak, custom signal-to-noise ratio, bell-curve similarity) from the tardis_publication_code deposit, then train random-forest classifiers in R using the deposited scripts. Train single-matrix classifiers per biofluid (stool, saliva, urine) and a multimatrix model pooling all matrices, evaluating against reported targets such as overall accuracy 59.45%, Bad-prediction precision 98.57% with zero Good peaks labeled Bad, and 70% balanced Ambiguous accuracy on FGFP. Compute random-forest variable importance and compare metric rankings: the article's expectation is that SNR and peak shape (bell-curve similarity) outrank AUC and points over the peak (POP). Rationale: the key utility of the classifier is not overall accuracy but the high precision of 'Bad' predictions, so variable importance and Bad-class reliability are the primary diagnostics.

## Related tools

- **R** (Language used to train the single- and multimatrix random-forest classifiers and compute variable importance with the deposited scripts)
- **TARDIS** (Source of the peak-quality metrics (AUC, maximum intensity, points over the peak, custom SNR, bell-curve similarity) that form classifier features) — https://github.com/UGent-LIMET/TARDIS
- **tardis_publication_code** (Deposit containing the R scripts and labeled datasets to reproduce the RF classification and figures) — https://github.com/UGent-LIMET/tardis_publication_code

## Examples

```
Rscript tardis_publication_code/peak_quality_rf.R  # trains single- and multimatrix RFs, computes variable importance, writes peak_quality_rf_results.csv
```

## Evaluation signals

- Multimatrix RF overall accuracy reproduces 59.45% on FGFP metabolomics data
- Bad-prediction precision reaches 98.57% with zero Good peaks classified as Bad
- Balanced accuracy on FGFP Ambiguous peaks is 70%
- Variable-importance ranking shows SNR and peak shape metrics outrank AUC and points over the peak (POP)
- Reproduced metrics written to peak_quality_rf_results.csv match the reported figures

## Limitations

- Overall classifier accuracy is modest (59.45%), so the classifier's value lies in Bad-prediction precision rather than full three-class accuracy
- Missing-signals note: biological test data used for large-scale evaluation are not named or accessioned in the article
- Results depend on the specific FGFP/FAME/ENVIRONAGE cohort data; cross-matrix behavior may differ for other study designs or instruments

## Evidence

- [abstract] Both established (area under the curve, maximum intensity and points over the peak) and recently developed (custom signal-to-noise ratio and bell-curve similarity) quality metrics were included: "Both established (area under the curve, maximum intensity and points over the peak) and recently developed (custom signal-to-noise ratio and bell-curve similarity) quality metrics were included"
- [discussion] Machine learning classification potential: "the potential of machine learning classification to enhance peak filtering by quality"
- [readme] This repository contains all R and python scripts to reproduce the data-analysis and figures: [evidence span withheld — anchor status: external_doc]
