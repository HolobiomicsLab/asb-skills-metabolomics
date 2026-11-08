---
name: afff-nontargeted-data-processing
description: Use when processing LC-HRMS/MS acquisitions of AFFF or environmental/biological
  samples for non-targeted PFAS annotation, where raw MS/MS data must be filtered
  and scored rather than curated manually.
when_to_use_negative:
- Input is already a filtered, annotated PFAS list — the skill operates on raw LC-HRMS/MS
  data through feature detection onward.
- Targeted analysis of only a few well-known PFAS (e.g., PFOA and PFOS) is sufficient;
  manual or targeted workflows are simpler in that scope.
- The sample type or acquisition mode makes MS/MS evidence unavailable (MS1-only data),
  since class-based MS/MS screening and fragment screening depend on tandem spectra.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: FluoroMatch Flow 2.0
  role: Covers all steps of data processing for PFAS discovery in LC-HRMS/MS samples,
    from feature detection to scoring
- name: FluoroMatch Modular
  role: Modular pipeline in which the AFFF acquisitions are loaded and feature detection,
    blank filtering, MS/MS screening, and scoring are run per injection round
  repo: github.com/InnovativeOmics/Core-Match
- name: LipidMatch 4.2
  role: Distribution package containing the FluoroMatch Modular framework and LipidMatch_Libraries/Scripts
    into which edited scripts are integrated
  repo: github.com/InnovativeOmics/Core-Match
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:1ec0ab0d6832b33c0db7e3edd087f3d9ebb85f5820c0d562cc00956c7db53e47
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FluoroMatch Flow 2.0
  - FluoroMatch Modular
  - LipidMatch
  - LipidMatch 4.2
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# afff-nontargeted-data-processing

## Summary

Automates the full non-targeted LC-HRMS/MS data processing pipeline for PFAS discovery in aqueous film forming foam (AFFF) using FluoroMatch 2.0/Flow 2.0, from raw feature detection through class-based and fragment MS/MS screening to a scored confidence-level annotation output.

## When to use

Use when processing LC-HRMS/MS acquisitions of AFFF or environmental/biological samples for non-targeted PFAS annotation, where raw MS/MS data must be filtered and scored rather than curated manually.

## When NOT to use

- Input is already a filtered, annotated PFAS list — the skill operates on raw LC-HRMS/MS data through feature detection onward.
- Targeted analysis of only a few well-known PFAS (e.g., PFOA and PFOS) is sufficient; manual or targeted workflows are simpler in that scope.
- The sample type or acquisition mode makes MS/MS evidence unavailable (MS1-only data), since class-based MS/MS screening and fragment screening depend on tandem spectra.

## Inputs

- Raw LC-HRMS/MS acquisitions of AFFF samples (initial DDA injection plus iterative-exclusion rounds)
- Catalogued PFAS structure list from the EPA CompTox Chemicals Dashboard (over 8000 structures)
- LipidMatch-4.2 distribution (FluoroMatch_Modular with FluoroMatch_Manual.docx and Trouble_Shooting_Common_Issues.docx)

## Outputs

- Scored PFAS annotations with confidence levels (score A = confident) per injection round
- Per-round counts of features with PFAS MS/MS evidence and confident annotations (e.g., iex_rounds_table.csv, one row per injection round)
- Filtered feature set with likely-non-PFAS features removed

## How to apply

Load the LC-HRMS/MS acquisitions (e.g., four AFFF injections: an initial DDA injection plus iterative-exclusion rounds) into FluoroMatch Modular, copying root Scripts into RapidTest/LipidMatch_Libraries/Scripts/ per the Core-Match README. Run the Flow 2.0 pipeline steps in order: feature detection, feature blank filtering, exact mass matching to catalogued PFAS (e.g., the EPA CompTox list of over 8000 structures), mass defect filtering, homologous series detection, retention time pattern analysis, class-based MS/MS screening, fragment screening, and predicted MS/MS from SMILES structures. Apply the comprehensive confidence level criterion to collapse all evidence layers into scored annotations (score A = confident), which reduces overreporting. Count per-injection features with PFAS MS/MS evidence and confident annotations to quantify coverage across rounds. Validation centers on reproducing the reported coverage trend, e.g., 105 features with PFAS MS/MS evidence in the initial AFFF injection rising to 399 after three iterative-exclusion rounds (+280%), with score A annotations up 139%.

## Related tools

- **FluoroMatch Flow 2.0** (Covers all steps of data processing for PFAS discovery in LC-HRMS/MS samples, from feature detection to scoring)
- **FluoroMatch Modular** (Modular pipeline in which the AFFF acquisitions are loaded and feature detection, blank filtering, MS/MS screening, and scoring are run per injection round) — github.com/InnovativeOmics/Core-Match
- **LipidMatch 4.2** (Distribution package containing the FluoroMatch Modular framework and LipidMatch_Libraries/Scripts into which edited scripts are integrated) — github.com/InnovativeOmics/Core-Match

## Examples

```
Rscript Modular.r  # with FLOW <- FALSE, csvInput <- FALSE, ManuallyInputVariables <- FALSE, Lipid <- TRUE set in the script
```

## Evaluation signals

- Round-wise counts table contains four round rows (one per injection) with exact integer counts (row_count_equals).
- Counts reproduce the reported trend: 105 features with PFAS MS/MS evidence in the initial injection rising to 399 after three iterative-exclusion rounds (output_matches_reference).
- Confident (score A) annotations improve by approximately 139% across the iterative-exclusion rounds.
- The additional confident annotations decrease across subsequent iterative injections, indicating high-abundance compounds were fragmented by the fourth injection.
- A large fraction of non-PFAS features is removed — reference behavior filtered out 96% of features likely not PFAS.

## Limitations

- Additional confident annotations per iterative-exclusion round diminish, so gains from repeated injections saturate once high-abundance compounds have been fragmented.
- Data processing and interpretation remain the major bottleneck in non-targeted PFAS analysis; manual comprehensive survey is tedious and prone to human error.
- Commonly screened PFAS may not represent the most abundant exposures (30–70% of total carbon-fluorine content in serum was not attributable to commonly measured PFAS), so coverage gains should be interpreted against unmeasured species.

## Evidence

- [abstract] pipeline steps: "These steps include feature detection, feature blank filtering, exact mass matching to catalogued PFAS, mass defect filtering, homologous series detection"
- [abstract] MS/MS screening steps: "retention time pattern analysis, class-based MS/MS screening, fragment screening, and predicted MS/MS from SMILES structures"
- [abstract] filtering performance: "Furthermore, we were able to filter out 96% of features which were likely not PFAS."
- [readme] integration instructions: "Copy the contents of the root Scripts folder into:
   * RapidTest/LipidMatch_Libraries/Scripts/"
- [intro] bottleneck rationale: "The current major bottleneck in non-targeted mass spectrometric analysis is in data processing and interpretation"
