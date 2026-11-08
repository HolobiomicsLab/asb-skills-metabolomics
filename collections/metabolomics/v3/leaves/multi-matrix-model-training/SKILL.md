---
name: multi-matrix-model-training
description: Use when labeled peak-quality data from multiple biofluid matrices (stool,
  saliva, urine) is available and a single classifier is needed that reliably flags
  'Bad' targeted LC–MS peaks across matrices.
when_to_use_negative:
- Data are unlabeled — the classifier is supervised and requires peaks already annotated
  as 'Good'/'Bad' quality
- Only a single biofluid matrix is in scope — the article also trains single-matrix
  RF classifiers per matrix, which may be the better fit
- The task is untargeted LC–MS preprocessing (e.g. xcms, MZmine, OpenMS) rather than
  targeted peak integration and quality classification
edam_operation: http://edamontology.org/operation_0389
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: TARDIS
  role: Open-source R package that integrates targeted LC–MS chromatographic peaks
    and computes the quality metrics used as classifier features
  repo: https://github.com/UGent-LIMET/TARDIS
- name: R
  role: Language used to train and evaluate the random-forest classifiers and compute
    variable importance
  repo: https://github.com/UGent-LIMET/tardis_publication_code
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:580fc10c4b2af10345d72ac42c5b7c4d12471764fe1e6270fa194cc1df6cad58
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
  edam_operation: http://edamontology.org/operation_0389
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# multi-matrix-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains a pooled (multimatrix) random-forest classifier on TARDIS peak-quality metrics (AUC, maximum intensity, points over the peak, custom SNR, bell-curve similarity) to identify 'Bad' chromatographic peaks across stool, saliva and urine matrices in FGFP LC–MS metabolomics data.

## When to use

Use when labeled peak-quality data from multiple biofluid matrices (stool, saliva, urine) is available and a single classifier is needed that reliably flags 'Bad' targeted LC–MS peaks across matrices.

## When NOT to use

- Data are unlabeled — the classifier is supervised and requires peaks already annotated as 'Good'/'Bad' quality
- Only a single biofluid matrix is in scope — the article also trains single-matrix RF classifiers per matrix, which may be the better fit
- The task is untargeted LC–MS preprocessing (e.g. xcms, MZmine, OpenMS) rather than targeted peak integration and quality classification

## Inputs

- Labeled peak-quality-metric dataset (AUC, maximum intensity, points over the peak, custom signal-to-noise ratio, bell-curve similarity) from tardis_publication_code / Zenodo 10.5281/zenodo.14548032
- Peak integration output from TARDIS runs on stool, saliva and urine FGFP metabolomics data

## Outputs

- Trained multimatrix random-forest classifier for peak quality ('Good'/'Bad') prediction
- Random-forest variable-importance ranking of the five quality metrics
- peak_quality_rf_results.csv with reproduced accuracy and precision metrics
- Regenerated classification figures

## How to apply

Load the labeled peak-quality-metric dataset (AUC, maximum intensity, points over the peak, custom signal-to-noise ratio, bell-curve similarity) from the UGent-LIMET/tardis_publication_code deposit (Zenodo 10.5281/zenodo.14548032). In R, first train single-matrix random-forest classifiers per biofluid (stool, saliva, urine), then pool all matrices to train the multimatrix RF model. Evaluate against the reported targets: overall accuracy 59.45% on FGFP metabolomics, 'Bad'-prediction precision of 98.57% with zero 'Good' peaks labeled 'Bad', and 70% balanced accuracy on FGFP 'Ambiguous' peaks. Compute random-forest variable importance and confirm that SNR and peak-shape metrics outrank AUC and points over the peak (POP). Regenerate the reported classification figures and write the reproduced metrics to peak_quality_rf_results.csv. The rationale is that machine-learning classification can enhance peak filtering by quality, with the multimatrix model prized for cross-matrix robustness rather than raw overall accuracy.

## Related tools

- **TARDIS** (Open-source R package that integrates targeted LC–MS chromatographic peaks and computes the quality metrics used as classifier features) — https://github.com/UGent-LIMET/TARDIS
- **R** (Language used to train and evaluate the random-forest classifiers and compute variable importance) — https://github.com/UGent-LIMET/tardis_publication_code

## Evaluation signals

- Multimatrix RF overall accuracy on FGFP metabolomics reproduces ~59.45%
- 'Bad'-prediction precision is 98.57% with zero 'Good' peaks classified as 'Bad'
- Balanced accuracy on FGFP 'Ambiguous' peaks reaches ~70%
- Variable-importance ranking places SNR and peak shape (bell-curve similarity) above AUC and points over the peak (POP)
- Reproduced metrics and figures are written to peak_quality_rf_results.csv without errors

## Limitations

- Overall multimatrix accuracy is modest (~59.45%); the model's value lies in high-precision 'Bad' detection, not overall accuracy
- The single-matrix vs multimatrix distinction must be respected when comparing to the paper's reported per-matrix results
- The article does not report explicit limitations of TARDIS in its discussion section, and vendor comparison software is only described generically as 'state-of-the-art vendor software'

## Evidence

- [readme] reproducibility scripts: [evidence span withheld — anchor status: external_doc]
