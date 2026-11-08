---
name: cohort-metadata-harmonization
description: Use when a metabolomics feature table (e.g. LC-MS/MS peak areas) must
  be joined to per-subject cohort metadata (age, sex, BMI, CSF biomarkers such as
  p-tau, α-synuclein, Aβ42/Aβ40) to run covariate-adjusted correlation analyses in
  a human cohort subset.
when_to_use_negative:
- The input is a cross-tissue mouse molecular network with no human subject metadata
  to join — mapping to human data should instead proceed via tissueMASST search first
- The goal is purely spectral annotation of MS/MS spectra against GNPS reference libraries,
  with no downstream cohort statistics
- The cohort metadata lacks a shared subject identifier or biomarker subset, so no
  per-subject merge can be validated
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: tissueMASST
  role: Maps the feature to multiple human AD cohorts to identify which cohort's metadata
    should be harmonized for association testing
- name: GNPS
  role: Spectral matching against public reference libraries to confirm the identity
    of the feature being joined to cohort metadata (e.g. PAC Level 1 annotation, m/z
    280.153)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:0ada78c190ca5dcc53097106bed3a4b7df52d53f6e0a50ff3c0c3260b6ff7e0a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - tissueMASST
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cohort-metadata-harmonization

## Summary

Harmonizes cohort-level subject metadata (age, sex, BMI, CSF biomarkers) with extracted metabolomics feature tables so that metabolites discovered in mouse models via tissueMASST can be tested for covariate-adjusted associations in human AD cohorts, e.g. phenylacetyl-carnitine vs. phosphorylated tau, α-synuclein, and the Aβ42/Aβ40 ratio in the MARS CSF subset (n=77).

## When to use

Use when a metabolomics feature table (e.g. LC-MS/MS peak areas) must be joined to per-subject cohort metadata (age, sex, BMI, CSF biomarkers such as p-tau, α-synuclein, Aβ42/Aβ40) to run covariate-adjusted correlation analyses in a human cohort subset.

## When NOT to use

- The input is a cross-tissue mouse molecular network with no human subject metadata to join — mapping to human data should instead proceed via tissueMASST search first
- The goal is purely spectral annotation of MS/MS spectra against GNPS reference libraries, with no downstream cohort statistics
- The cohort metadata lacks a shared subject identifier or biomarker subset, so no per-subject merge can be validated

## Inputs

- Untargeted LC-MS/MS metabolomics files from a human cohort (e.g. MARS) downloaded from the AD Knowledge Portal
- Per-subject metadata table: age, sex, BMI, and CSF biomarkers (phosphorylated tau, α-synuclein, Aβ42/Aβ40 ratio) for the biomarker-available subset (n=77)
- Extracted feature peak-area table for the metabolite of interest (e.g. PAC, m/z 280.153 M+H)

## Outputs

- Merged feature-by-subject table linking log-transformed peak areas to age, sex, BMI, and CSF biomarker values (e.g. mars_pac_csf_correlations.csv inputs)
- Adjusted partial correlation estimates: coefficients and p-values for each biomarker association
- Annotation provenance record (annotation level and spectral-match basis for the feature)

## How to apply

First identify the cohort and its biomarker subset (e.g. MARS individuals with available CSF measurements, n=77) and download subject metadata alongside the untargeted metabolomics files. Reprocess the LC-MS/MS data to extract peak areas for the feature of interest — for phenylacetyl-carnitine this means confirming the annotation (M+H, m/z 280.153) by spectral matching against the GNPS libraries, recognizing that GNPS matching yields a Level 2 or 3 MSI annotation. Merge the extracted peak-area table with the subject table on a shared subject identifier, log-transform the feature intensities, and compute partial correlations of log-transformed peak areas with each biomarker adjusted for age, sex, and BMI (as in Figure 6G). The rationale is that tissueMASST bridges animal-model findings with human data, and harmonizing metadata is what turns a mouse-model hit into a testable human cohort association.

## Related tools

- **tissueMASST** (Maps the feature to multiple human AD cohorts to identify which cohort's metadata should be harmonized for association testing)
- **GNPS** (Spectral matching against public reference libraries to confirm the identity of the feature being joined to cohort metadata (e.g. PAC Level 1 annotation, m/z 280.153))

## Evaluation signals

- The merged analysis subset has the expected size (n=77 individuals with CSF measurements in MARS)
- Signs of the age-, sex-, and BMI-adjusted partial correlations match the reported directions: positive with phosphorylated tau and α-synuclein, negative with the Aβ42/Aβ40 ratio (Figure 6G)
- Peak-area values were log-transformed before correlation, and age, sex, and BMI are included as covariates in every model
- The feature annotation is traceable to spectral matching against GNPS libraries, with the expected precursor mass (PAC M+H at m/z 280.153)
- Output CSV contains, for each biomarker, both a coefficient and a p-value

## Limitations

- GNPS spectral matching yields Level 2 or 3 annotations under the Metabolomics Standards Initiative, not fully validated structures
- Feature detection across reference datasets can be influenced by study-specific extraction methods, workflows, and instrumentation, so lack of detection does not necessarily mean absence — cross-cohort comparability of peak areas is imperfect
- Cohort subsets restricted to individuals with CSF measurements (n=77) may not represent the full cohort, and hundreds of differentially abundant molecules remain non-annotated

## Evidence

- [abstract] tissueMASST maps PAC to human cohorts: "tissueMASST identifies microbially-modulated phenylacetyl-carnitine as positively associated with aging and cognitive impairment across human AD studies"
- [intro] PAC mapped to multiple human AD cohorts: "we mapped phenylacetyl-carnitine to multiple human AD cohorts, where this metabolite was positively associated with aging and cognitive impairment"
- [discussion] GNPS annotation level caveat: "MS/MS spectra annotation via spectral matching against the public GNPS reference libraries represent a Level 2 or 3 annotation"
- [discussion] Cross-cohort detection confounding: "Feature detection across reference datasets present in tissueMASST can be influenced by study-specific extraction methods, workflows, and instrumentation"
