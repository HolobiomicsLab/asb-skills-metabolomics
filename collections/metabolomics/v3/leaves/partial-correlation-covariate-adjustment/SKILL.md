---
name: partial-correlation-covariate-adjustment
description: Use when testing whether a specific metabolite feature (e.g. a Level
  1-annotated carnitine peak from untargeted metabolomics) is associated with a continuous
  clinical biomarker in a human cohort subset, and demographic covariates (age, sex,
  BMI) must be held constant.
when_to_use_negative:
- When the metabolite of interest is not Level 1 or Level 2 annotated (e.g. only a
  putative MSI Level 3 assignment) — the directional validation against a named metabolite
  would be uninterpretable.
- When covariates are collinear with the biomarker itself (e.g. age is part of the
  diagnostic definition rather than a confounder), partialling it out may remove signal
  rather than confounding.
- When the sample size after subsetting is too small to support a partial correlation
  with the chosen covariate set (the method assumes sufficient residual degrees of
  freedom).
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: tissueMASST
  role: Used upstream to identify which metabolite feature (e.g. phenylacetyl-carnitine)
    to test against the clinical biomarkers by matching the MS/MS spectrum against
    public repositories of human AD cohorts.
- name: GNPS
  role: Provides the spectral reference libraries used to reach a Level 2/3 annotation
    for the metabolite feature before it is subjected to partial correlation analysis.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:7a1fa2b59a3d150e197ceb0dcfa683d415cc784ec62a9cba595c0508990dee6e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - tissueMASST
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# partial-correlation-covariate-adjustment

## Summary

Computes partial correlations between a single metabolite feature (e.g. phenylacetyl-carnitine peak areas from untargeted LC-MS/MS) and clinical/biomarker outcomes (CSF p-tau, α-synuclein, Aβ42/Aβ40) while adjusting for confounders such as age, sex, and BMI. Used to separate metabolite-biomarker associations from demographic confounding in cohort subsets.

## When to use

Use when testing whether a specific metabolite feature (e.g. a Level 1-annotated carnitine peak from untargeted metabolomics) is associated with a continuous clinical biomarker in a human cohort subset, and demographic covariates (age, sex, BMI) must be held constant.

## When NOT to use

- When the metabolite of interest is not Level 1 or Level 2 annotated (e.g. only a putative MSI Level 3 assignment) — the directional validation against a named metabolite would be uninterpretable.
- When covariates are collinear with the biomarker itself (e.g. age is part of the diagnostic definition rather than a confounder), partialling it out may remove signal rather than confounding.
- When the sample size after subsetting is too small to support a partial correlation with the chosen covariate set (the method assumes sufficient residual degrees of freedom).

## Inputs

- metabolite feature table (peak area per subject) for the target metabolite (e.g. phenylacetyl-carnitine, m/z 280.153 M+H)
- subject-level biomarker table with phosphorylated tau, α-synuclein, and Aβ42/Aβ40 ratio values
- subject metadata: age, sex, BMI

## Outputs

- partial correlation coefficient, p-value, and sign for each biomarker-metabolite pair (e.g. mars_pac_csf_correlations.csv)

## How to apply

First restrict the analysis to the subset of subjects with both the metabolite measurement and the target biomarker available (e.g. the n=77 MARS CSF subset). Log-transform the metabolite peak areas to stabilize variance, then compute the partial correlation of log-transformed peak area with each biomarker (phosphorylated tau, α-synuclein, Aβ42/Aβ40 ratio) using age, sex, and BMI as covariates in a partial correlation model. Record the sign, coefficient, and p-value for each biomarker. Validation is directional: the expected pattern is a positive partial correlation with phosphorylated tau and α-synuclein, and a negative one with the Aβ42/Aβ40 ratio, consistent with the reported tissueMASST-guided finding that phenylacetyl-carnitine is associated with aging and cognitive impairment.

## Related tools

- **tissueMASST** (Used upstream to identify which metabolite feature (e.g. phenylacetyl-carnitine) to test against the clinical biomarkers by matching the MS/MS spectrum against public repositories of human AD cohorts.)
- **GNPS** (Provides the spectral reference libraries used to reach a Level 2/3 annotation for the metabolite feature before it is subjected to partial correlation analysis.)

## Evaluation signals

- The subset used for each correlation must match the number of subjects with both metabolite and biomarker data (e.g. n=77 for the MARS CSF subset).
- The sign of each partial correlation must match the expected direction: positive with phosphorylated tau and α-synuclein, negative with the Aβ42/Aβ40 ratio.
- The metabolite peak must have been annotated at MSI Level 1 or 2 (e.g. via spectral matching to the GNPS library, as in the M+H m/z 280.153 identification of PAC) before being entered into the model.
- The covariate set (age, sex, BMI) must be identical across all biomarker correlations within a single run for comparability.
- Peak areas should be log-transformed before computing correlations, and the transformation should be reported alongside the coefficients.

## Limitations

- Annotations reached via GNPS spectral matching are only MSI Level 2 or 3, not fully validated structures, so downstream correlation results are conditional on the annotation confidence.
- Covariate adjustment assumes the chosen covariates (age, sex, BMI) fully capture confounding; unmeasured confounders (e.g. medication, diet) are not accounted for.
- Feature detection depends on study-specific extraction methods, workflows, and instrumentation, so lack of detection in a reference dataset does not imply biological absence — this can bias which metabolites are selected for correlation testing.

## Evidence

- [other] subset: [evidence span withheld — anchor status: not_found]
- [discussion] annotation: "MS/MS spectra annotation via spectral matching against the public GNPS reference libraries represent a Level 2 or 3 annotation"
- [discussion] feature_detection: "Feature detection across reference datasets present in tissueMASST can be influenced by study-specific extraction methods, workflows, and instrumentation. Thus, lack of detection does not necessarily"
