---
name: pfas-class-coverage-analysis
description: Use when an automated PFAS annotation table (e.g. from FluoroMatch 2.0
  on AFFF LC-HRMS/MS data) must be compared feature-by-feature against a published
  manual non-targeted annotation to quantify agreement and coverage.
when_to_use_negative:
- Input is raw LC-HRMS/MS data requiring feature detection or blank filtering — this
  skill compares finished annotation tables, not spectra
- The goal is de novo PFAS discovery or suspect screening; use FluoroMatch 2.0's pipeline
  steps (mass defect filtering, homologous series detection) instead
- Annotations come from targeted/suspect-screening workflows without level-A confidence
  scoring, making the level-A agreement metric meaningless
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3391
tools:
- name: FluoroMatch 2.0
  role: Source of the automated non-targeted annotation table being benchmarked against
    the manual workflow
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Modular
  role: Distributed FluoroMatch framework whose Modular component (LipidMatch-4.2\FluoroMatch_Modular)
    is configured to run PFAS analysis and produce the annotation output
  repo: github.com/InnovativeOmics/Core-Match
- name: pandas/openpyxl
  role: Python libraries used to load the ESM Excel comparison file and write the
    comparison workbook
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:4c8662d0856833b66076bd6b1429e80a812efb83560cfaa51a897ddca8e93cd4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FluoroMatch 2.0
  - FluoroMatch Modular
  - pandas/openpyxl
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# pfas-class-coverage-analysis

## Summary

Benchmarks FluoroMatch 2.0's automated non-targeted PFAS annotation of an AFFF sample against a previously published manual workflow, quantifying class coverage, level-A structural agreement, species counts, and confidently assigned homologous series.

## When to use

Use when an automated PFAS annotation table (e.g. from FluoroMatch 2.0 on AFFF LC-HRMS/MS data) must be compared feature-by-feature against a published manual non-targeted annotation to quantify agreement and coverage.

## When NOT to use

- Input is raw LC-HRMS/MS data requiring feature detection or blank filtering — this skill compares finished annotation tables, not spectra
- The goal is de novo PFAS discovery or suspect screening; use FluoroMatch 2.0's pipeline steps (mass defect filtering, homologous series detection) instead
- Annotations come from targeted/suspect-screening workflows without level-A confidence scoring, making the level-A agreement metric meaningless

## Inputs

- ESM Excel comparison file with previously published manual non-targeted AFFF annotation (per-class, per-series, per-species assignments)
- FluoroMatch 2.0 annotation table for the AFFF sample (species, classes, homologous series, level-A structural assignments)
- Homologous series metadata: PFAS class labels plus retention-time and mass-defect ordering information

## Outputs

- afff_annotation_comparison_table.xlsx containing the class-coverage and species-count comparison (16/17 classes, 13 identical level-A classes, 87 vs 50 species)
- Confident-series comparison table with exactly 22 rows, tabulating homologous series with confident structures out of 50 manually reported series

## How to apply

Load the ESM Excel comparison file containing the previously published manual AFFF annotation and the FluoroMatch 2.0 annotation table using pandas/openpyxl. Align homologous series between the automated and manual annotations by matching on PFAS class and retention-time/mass-defect ordering. Exclude branched isomers when tallying species assigned per matched series, then count classes covered, classes with identical level-A structures, and series with confident structures. Write the comparison to afff_annotation_comparison_table.xlsx, keeping the confident-series table at exactly 22 rows. Validate that all integer fields reproduce the reported benchmark (16/17 classes, 13 identical level-A classes, 87 vs 50 species excluding branched isomers, 22/50 confident series). The comparison is meaningful because FluoroMatch 2.0 applies a scoring framework integrating mass defect filtering, homologous series detection, and class-based MS/MS screening into level-A confidence assignments.

## Related tools

- **FluoroMatch 2.0** (Source of the automated non-targeted annotation table being benchmarked against the manual workflow) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch Modular** (Distributed FluoroMatch framework whose Modular component (LipidMatch-4.2\FluoroMatch_Modular) is configured to run PFAS analysis and produce the annotation output) — github.com/InnovativeOmics/Core-Match
- **pandas/openpyxl** (Python libraries used to load the ESM Excel comparison file and write the comparison workbook)

## Examples

```
import pandas as pd; manual = pd.read_excel('ESM_manual_AFFF_annotation.xlsx', engine='openpyxl'); fm2 = pd.read_excel('FluoroMatch2_AFFF_annotation.xlsx', engine='openpyxl'); comparison.to_excel('afff_annotation_comparison_table.xlsx')
```

## Evaluation signals

- Comparison table's integer fields exactly equal the reported benchmark: 16/17 PFAS classes, 13 classes with identical level-A structures, 87 vs 50 species, 22/50 confident series
- Confident-series table has row_count = 22
- Species tallies exclude branched isomers consistently on both the automated and manual sides before the 87 vs 50 comparison
- Every matched homologous series is assigned to a named PFAS class and consistent with retention-time/mass-defect ordering in both annotation tables

## Limitations

- Branched isomers are excluded from species counts, so the 87 vs 50 comparison understates total annotated features
- The benchmark is specific to a single AFFF sample and one published manual workflow; agreement metrics may not transfer to other matrices (environmental, biological)
- Only 22 of 50 homologous series received confident structures, reflecting that many series lack level-A evidence even after automated annotation

## Evidence
