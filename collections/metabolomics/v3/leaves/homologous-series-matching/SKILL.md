---
name: homologous-series-matching
description: Use when processing untargeted LC-HRMS/MS data (e.g. aqueous film forming
  foam or environmental samples) to discover PFAS homologous series and assign per-compound
  confidence levels beyond a small targeted PFOA/PFOS panel.
when_to_use_negative:
- The original FluoroMatch release's suspect-screening approach is the right choice
  when you only want to target compounds with available MS/MS spectra rather than
  discover unknown/uncommon PFAS.
- Analysis is general small-molecule or lipid non-targeted screening; here the pipeline
  is PFAS-specific unless Lipid <- TRUE is set for lipids.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0593
tools:
- name: FluoroMatch 2.0
  role: Performs the automated non-targeted PFAS annotation pipeline including homologous
    series detection and confidence scoring
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Flow 2.0
  role: Covers all steps of data processing for PFAS discovery in LC-HRMS/MS samples
    end to end
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Modular
  role: Module-by-module execution of the pipeline (FLOW <- FALSE) for PFAS analysis
    and troubleshooting per the FluoroMatch_Manual.docx
  repo: github.com/InnovativeOmics/Core-Match
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:46bf7479e38c2ef6fcc90e20404e7b1abd9e579c25627c77a1121fff980feda5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FluoroMatch 2.0
  - FluoroMatch Flow 2.0
  - FluoroMatch Modular
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# homologous-series-matching

## Summary

Automated detection and structural annotation of PFAS homologous series in LC-HRMS/MS data using FluoroMatch 2.0, which chains feature detection, blank/mass-defect filtering, exact mass matching, retention-time pattern analysis, and class-based MS/MS screening to replace tedious manual series surveying.

## When to use

Use when processing untargeted LC-HRMS/MS data (e.g. aqueous film forming foam or environmental samples) to discover PFAS homologous series and assign per-compound confidence levels beyond a small targeted PFOA/PFOS panel.

## When NOT to use

- The original FluoroMatch release's suspect-screening approach is the right choice when you only want to target compounds with available MS/MS spectra rather than discover unknown/uncommon PFAS.
- Analysis is general small-molecule or lipid non-targeted screening; here the pipeline is PFAS-specific unless Lipid <- TRUE is set for lipids.

## Inputs

- Raw LC-HRMS/MS data files (e.g. mzML) for sample and blanks, such as aqueous film forming foam runs
- EPA CompTox Chemicals Dashboard PFAS structure list (over 8000 structures, with SMILES)
- FluoroMatch Modular Excel/CSV input configuration (csvInput option)
- Prior manual annotation tables (e.g. ESM Excel comparison file) for benchmarking

## Outputs

- Annotation table of features grouped into homologous series with class assignments
- Per-compound confidence scores/levels (including level-A structural assignments)
- Filter report of non-PFAS features removed (96% filtered in the AFFF application)
- Comparison table (e.g. afff_annotation_comparison_table.xlsx) of automated vs manual annotation metrics

## How to apply

Run the FluoroMatch 2.0 pipeline (via FluoroMatch Flow 2.0 or FluoroMatch Modular with FLOW <- FALSE, which defaults to PFAS analysis) on raw LC-HRMS/MS files: feature detection, feature blank filtering, exact mass matching against the EPA CompTox Chemicals Dashboard PFAS list of over 8000 structures, mass defect filtering, and homologous series detection. Follow with retention time pattern analysis, class-based MS/MS screening, fragment screening, and predicted MS/MS from SMILES structures to raise series members to higher annotation confidence (e.g. level-A structural assignments). Interpret results through the built-in comprehensive confidence level criterion, which integrates evidence layers to reduce overreporting. For validation, benchmark against a published manual annotation by matching series on class and retention-time/mass-defect ordering, excluding branched isomers when tallying species, and writing a comparison table (e.g. afff_annotation_comparison_table.xlsx) whose integer fields reproduce the reported values (16/17 classes, 13 identical level-A classes, 87 vs 50 species, 22/50 confident series).

## Related tools

- **FluoroMatch 2.0** (Performs the automated non-targeted PFAS annotation pipeline including homologous series detection and confidence scoring) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch Flow 2.0** (Covers all steps of data processing for PFAS discovery in LC-HRMS/MS samples end to end) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch Modular** (Module-by-module execution of the pipeline (FLOW <- FALSE) for PFAS analysis and troubleshooting per the FluoroMatch_Manual.docx) — github.com/InnovativeOmics/Core-Match

## Examples

```
FLOW <- FALSE; Lipid <- FALSE; TWeen_pos <- FALSE; source("LipidMatch-4.2/FluoroMatch_Modular/Modular.r")  # defaults to PFAS analysis per README
```

## Evaluation signals

- Comparison table integer fields equal benchmark values: 16/17 PFAS classes covered, 13 identical level-A classes, 87 vs 50 species (excluding branched isomers), 22/50 confident series.
- Confident-series table contains row_count = 22 when benchmarked against the manual AFFF workflow.
- Approximately 96% of features likely not PFAS are removed by the filtering steps.
- Coverage of likely PFAS is roughly tenfold greater than the previous FluoroMatch release on the same sample.
- Every annotation carries a confidence level from the comprehensive criterion rather than an unscored identification.

## Limitations

- Only 22 of 50 homologous series received confident structural assignments even with automation; many series remain unannotated at high confidence.
- Branched isomers are excluded from species tallies during benchmark comparisons, which affects counts.
- Only 13 of the 16 covered classes had level-A structures identical to the manual workflow, so some class-level disagreements persist.
- Developers must integrate edited code manually into the InnovativeOmics distribution directory structure (e.g. LipidMatch-4.2\FluoroMatch_Modular); the GitHub repo is not the stable user release.

## Evidence
