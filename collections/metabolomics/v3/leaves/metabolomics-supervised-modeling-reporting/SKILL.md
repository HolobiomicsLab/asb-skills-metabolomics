---
name: metabolomics-supervised-modeling-reporting
description: Use when GCxGC-TOFMS metabolite feature tables (e.g. 195 metabolites
  from plasma) carry sample-class labels such as S. Typhi carriage, S. Paratyphi A
  carriage, and non-carriage controls, and the goal is supervised discrimination with
  significance testing and ROC-validated latent biomarkers.
when_to_use_negative:
- The question is purely exploratory structure discovery with no class labels or discrimination
  hypothesis — plain PCA suffices and supervised OPLS-DA would overfit.
- The data are not GCxGC-TOFMS metabolite intensities but already-validated diagnostic
  test results, where the goal is prospective clinical screening rather than latent
  biomarker pattern discovery.
- The comparison targets acute enteric fever signatures rather than chronic carriage
  — the article notes substantial heterogeneity between these profiles, so carriage-model
  p-values should not be reused.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2269
tools:
- name: GCxGC-TOFMS
  role: Source instrument producing the 195-metabolite plasma feature matrix that
    the PCA/OPLS-DA and ROC workflow consumes
- name: OPLS-DA
  role: Supervised pattern-recognition model whose per-component separation significance
    (e.g. p = 0.0031 along the second component) is extracted and tabulated
- name: PCA
  role: Unsupervised prerequisite modeling step to visualize distribution of the three
    sample groups before supervised fitting
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pntd.0006215
    title: Diagnostic metabolite biomarkers of chronic typhoid carriage
  source_content_hash: sha256:f35403c6dc135082eab93dc0941b7d14f8e723dfa6bc2f9d3bbe369c8493f19d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCxGC-TOFMS
  - OPLS-DA
  - PCA
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# metabolomics-supervised-modeling-reporting

## Summary

Builds and reports supervised chemometric models (PCA for unsupervised overview, OPLS-DA for class discrimination) on GCxGC-TOFMS plasma metabolite profiles, extracting model significance p-values and ROC-validated biomarker panels (e.g. a five-metabolite carriage signature) into structured records.

## When to use

Use when GCxGC-TOFMS metabolite feature tables (e.g. 195 metabolites from plasma) carry sample-class labels such as S. Typhi carriage, S. Paratyphi A carriage, and non-carriage controls, and the goal is supervised discrimination with significance testing and ROC-validated latent biomarkers.

## When NOT to use

- The question is purely exploratory structure discovery with no class labels or discrimination hypothesis — plain PCA suffices and supervised OPLS-DA would overfit.
- The data are not GCxGC-TOFMS metabolite intensities but already-validated diagnostic test results, where the goal is prospective clinical screening rather than latent biomarker pattern discovery.
- The comparison targets acute enteric fever signatures rather than chronic carriage — the article notes substantial heterogeneity between these profiles, so carriage-model p-values should not be reused.

## Inputs

- GCxGC-TOFMS plasma metabolite feature table (e.g. 195 metabolites × cholecystectomy-patient samples from Nepal)
- Sample-class labels: S. Typhi carriage, S. Paratyphi A carriage, non-carriage controls
- Manuscript full text / figure captions containing model statistics (Results section, Fig 1, S1 Fig, Table 1)

## Outputs

- PCA scores plot (unsupervised distribution check of the three sample groups)
- OPLS-DA scores plots showing component-wise class separation with significance p-values per model
- Normalized p-value records written to oplsda_pvalues.csv (columns: model, comparison, p_value, source_location)
- ROC curves (false positive rate vs true positive rate) for the reduced five-metabolite carriage panel

## How to apply

First run unsupervised PCA scores to check the distribution of the sample groups (S. Typhi carriage, S. Paratyphi A carriage, non-carriage controls) before any supervised fitting. Then fit a multi-class OPLS-DA on the GCxGC-TOFMS metabolite matrix: interpret separation of non-carriage controls from Salmonella carriage samples along the first component and S. Typhi vs S. Paratyphi A carriage along the second component (reported p = 0.0031). Complement this with reduced models — a two-class combined-carriage model (p = 2.8*10−6), pairwise carrier-vs-control models (p = 3.1*10−5 and p = 3.7*10−4), and a direct S. Typhi vs S. Paratyphi A model (p = 0.07, non-significant) — normalizing asterisk notation (2.8*10−6 → 2.8e-06) and recording model, comparison, p_value, and source_location for each. Validate any reduced biomarker panel (e.g. the five-metabolite carriage signature) with ROC curves plotting sensitivity vs 1-specificity, and cross-check every reported statistic against its verbatim text span in the manuscript, Table 1, and Fig 1/S1 Fig captions.

## Related tools

- **GCxGC-TOFMS** (Source instrument producing the 195-metabolite plasma feature matrix that the PCA/OPLS-DA and ROC workflow consumes)
- **OPLS-DA** (Supervised pattern-recognition model whose per-component separation significance (e.g. p = 0.0031 along the second component) is extracted and tabulated)
- **PCA** (Unsupervised prerequisite modeling step to visualize distribution of the three sample groups before supervised fitting)

## Evaluation signals

- Every extracted p-value matches its verbatim text span in the article (e.g. '(p = 2.8*10−6)', '(p = 0.07)') as reported in Table 1 and the Fig 1 / S1 Fig descriptions.
- All five expected model records are present: three-class second-component separation (0.0031), two-class combined carriage (2.8e-06), S. Typhi vs controls (3.1e-05), S. Paratyphi A vs controls (3.7e-04), and direct S. Typhi vs S. Paratyphi A (0.07).
- Asterisk scientific notation is normalized to standard floats (2.8*10−6 → 2.8e-06) with no loss of magnitude.
- Each record in oplsda_pvalues.csv has non-empty model, comparison, p_value, and source_location fields.
- Resulting biomarker panel (five metabolites) achieves significant carrier-vs-control discrimination with an accompanying ROC curve plotting sensitivity against 1-specificity.

## Limitations

- Small sample size, difficult to overcome given the ethical, surgical, and diagnostic issues of identifying and confirming invasive Salmonella carriage.
- No independent validation cohort was available; profiles should be validated in an additional independent cohort before diagnostic use.
- All patients presented with gallbladder conditions, so the primary cause for cholecystectomy may confound the metabolite signatures, and controls with other bacterial gallbladder carriage were not included to confirm Salmonella specificity.

## Evidence

- [abstract] GCxGC-TOFMS input and supervised modeling: "using two-dimensional gas chromatography coupled with time-of-flight mass spectrometry (GCxGC-TOFMS) and supervised pattern recognition modeling"
- [supplementary] OPLS-DA first-component separation: "(B) OPLS-DA scores showing the separation of non-carriage controls from the Salmonella carriage samples along the first component"
- [supplementary] Second-component significance: "the separation of S. Typhi carriage samples from the S. Paratyphi A carriage samples along the second component (p = 0.0031)"
- [supplementary] Feature table scale: "models based on 195 metabolites generated from GCxGC-TOFMS analysis of plasma samples from patients in Nepal undergoing cholecystectomy"
- [supplementary] ROC validation format: "Panels A-E are showing ROC curves with false positive rates (i.e. 1-specificity) and true positive rates (i.e. sensitivity) on the x- and y-axes respectively"
