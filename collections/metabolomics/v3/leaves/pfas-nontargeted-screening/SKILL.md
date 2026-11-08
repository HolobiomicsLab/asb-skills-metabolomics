---
name: pfas-nontargeted-screening
description: Use when processing LC-HRMS/MS feature tables from PFAS discovery samples
  (e.g., AFFF or environmental extracts) and blanks, needing to separate true PFAS
  features from background and annotate known, uncommon, and unknown PFAS non-targeted.
when_to_use_negative:
- You already have a curated target list and MS/MS spectra — the original FluoroMatch
  release's suspect screening approach, not the 2.0 non-targeted workflow, fits that
  case
- Your analysis is lipidomics or Tween-related rather than PFAS — toggle Lipid <-
  TRUE is required for those applications and defaults otherwise to PFAS analysis
- You need quantification rather than annotation/discovery — the skill covers annotation
  and filtering, not quantitative calibration
edam_operation: http://edamontology.org/operation_3695
tools:
- name: FluoroMatch 2.0
  role: Primary software automating the non-targeted PFAS identification workflow,
    including blank filtering and confidence-level scoring
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Modular
  role: Modular execution mode (FLOW <- FALSE) for running individual workflow steps
    such as blank filtering on vendor feature tables
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Flow 2.0
  role: End-to-end pipeline covering all data-processing steps for PFAS discovery
    in LC-HRMS/MS samples
  repo: github.com/InnovativeOmics/Core-Match
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:a9fde20edfb7b9d25f1ea42e156a285a0dd15bf51ead72e3c4f9d15dbcbcfb29
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FluoroMatch 2.0
  - FluoroMatch Flow 2.0
  - FluoroMatch Modular
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# pfas-nontargeted-screening

## Summary

Automates the non-targeted PFAS annotation pipeline in FluoroMatch 2.0, from vendor feature-table detection through blank filtering, exact mass matching, mass defect and homologous series filtering, to MS/MS-based annotation with a confidence-level scoring framework.

## When to use

Use when processing LC-HRMS/MS feature tables from PFAS discovery samples (e.g., AFFF or environmental extracts) and blanks, needing to separate true PFAS features from background and annotate known, uncommon, and unknown PFAS non-targeted.

## When NOT to use

- You already have a curated target list and MS/MS spectra — the original FluoroMatch release's suspect screening approach, not the 2.0 non-targeted workflow, fits that case
- Your analysis is lipidomics or Tween-related rather than PFAS — toggle Lipid <- TRUE is required for those applications and defaults otherwise to PFAS analysis
- You need quantification rather than annotation/discovery — the skill covers annotation and filtering, not quantitative calibration

## Inputs

- Vendor LC-HRMS/MS sample feature table (e.g., AFFF sample)
- Vendor solvent-blank feature tables (e.g., three solvent blanks)
- Catalogued PFAS structure list (EPA CompTox Chemicals Dashboard, >8000 structures)
- SMILES structures for predicted MS/MS

## Outputs

- Blank-filtered sample feature table (e.g., blank_filtered_features.csv)
- Annotated list of likely PFAS with confidence-level scores
- Homologous series assignments and class-based MS/MS annotation results

## How to apply

Load vendor feature tables for samples (e.g., aqueous film forming foam) and solvent blanks into FluoroMatch Modular or FluoroMatch Flow 2.0, then run the workflow in sequence: feature detection, feature blank filtering, exact mass matching to catalogued PFAS (e.g., the EPA CompTox Chemicals Dashboard list of over 8000 PFAS structures), mass defect filtering, homologous series detection, retention time pattern analysis, class-based MS/MS screening, fragment screening, and predicted MS/MS from SMILES structures. For blank filtering specifically, align sample and blank features at ±0.005 Da precursor mass tolerance, compute blank average and SD across replicates, and remove features whose sample maximum does not exceed blank average + 3× blank SD (this reduced 31,132 AFFF features to 3740, ≈88% removed). Integrate all evidence layers into the paper's comprehensive confidence-level criterion to limit overreporting. Run Modular by setting FLOW <- FALSE (with FLOW <- TRUE for the Flow pipeline), following FluoroMatch_Flow_Manual.docx or FluoroMatch_Modular/FluoroMatch_Manual.docx for dependency installation.

## Related tools

- **FluoroMatch 2.0** (Primary software automating the non-targeted PFAS identification workflow, including blank filtering and confidence-level scoring) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch Modular** (Modular execution mode (FLOW <- FALSE) for running individual workflow steps such as blank filtering on vendor feature tables) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch Flow 2.0** (End-to-end pipeline covering all data-processing steps for PFAS discovery in LC-HRMS/MS samples) — github.com/InnovativeOmics/Core-Match

## Examples

```
Rscript Modular.r  # with FLOW <- FALSE; csvInput <- FALSE; ManuallyInputVariables <- FALSE; Lipid <- FALSE; TWeen_pos <- FALSE
```

## Evaluation signals

- Blank-filtered table contains exactly 3740 rows from 31,132 detected features on the AFFF reference data, matching the reported ≈88% removal fraction
- Removed features satisfy the blank criterion: sample maximum not greater than blank average + 3× blank SD across the three solvent blanks at ±0.005 Da alignment
- Overall pipeline filters out ~96% of features that are likely not PFAS on AFFF data
- Coverage of likely PFAS in Flow 2 should exceed the previous FluoroMatch release by over tenfold (~1000+ likely PFAS discovered in AFFF)
- Every reported annotation carries a confidence-level score consistent with the implemented scoring framework

## Limitations

- Blank filtering removes ~88% of detected features, so genuine low-abundance features near blank levels may be lost
- Comprehensive detection of PFAS remains difficult because of their vast structural diversity as trace components in environmental, industrial, and biological matrices
- Stable releases should be downloaded from innovativeomics.com/software; the GitHub repo is developer-edited code that must be integrated into the distribution directory structure manually
- Users must correctly toggle Lipid/TWeen flags; leaving both FALSE defaults to PFAS analysis, and misconfiguration affects downstream behavior

## Evidence
