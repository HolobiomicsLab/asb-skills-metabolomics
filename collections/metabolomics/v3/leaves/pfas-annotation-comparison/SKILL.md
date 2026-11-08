---
name: pfas-annotation-comparison
description: Use when benchmarking an automated non-targeted PFAS annotation run (e.g.,
  FluoroMatch 2.0 output on an AFFF sample) against a previously published manual
  annotation, to compare classes, level-A structures, species counts, and homologous
  series coverage.
when_to_use_negative:
- Input is raw LC-HRMS/MS data (mzML) that has not yet been annotated — run feature
  detection, blank/mass-defect filtering, and homologous series detection in FluoroMatch
  Flow 2.0 first
- The goal is de novo discovery of new PFAS rather than quantifying agreement with
  an existing published manual annotation
- Branch/isomer-resolved species accounting is required, since branched isomers are
  explicitly excluded from the species tallies here
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: FluoroMatch 2.0
  role: Produces the automated non-targeted annotation table for the AFFF sample that
    is benchmarked against the manual workflow
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Modular
  role: Modular pipeline used to run/adjust the annotation steps whose output is compared;
    code edits are integrated by replacing Modular.r in the FluoroMatch_Modular directory
  repo: github.com/InnovativeOmics/Core-Match
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:a3db3cac9f9396a2d94eb7e3f8a8ade5a81c1b65ecb94c16eabb01069fca2328
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FluoroMatch 2.0
  - FluoroMatch Modular
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# pfas-annotation-comparison

## Summary

Benchmarks FluoroMatch 2.0's automated non-targeted PFAS annotation of an AFFF sample against a previously published manual workflow by aligning homologous series and tabulating PFAS classes covered, identical level-A structures, species assigned (excluding branched isomers), and confidently annotated series.

## When to use

Use when benchmarking an automated non-targeted PFAS annotation run (e.g., FluoroMatch 2.0 output on an AFFF sample) against a previously published manual annotation, to compare classes, level-A structures, species counts, and homologous series coverage.

## When NOT to use

- Input is raw LC-HRMS/MS data (mzML) that has not yet been annotated — run feature detection, blank/mass-defect filtering, and homologous series detection in FluoroMatch Flow 2.0 first
- The goal is de novo discovery of new PFAS rather than quantifying agreement with an existing published manual annotation
- Branch/isomer-resolved species accounting is required, since branched isomers are explicitly excluded from the species tallies here

## Inputs

- ESM Excel comparison file containing the previously published manual non-targeted AFFF annotation
- FluoroMatch 2.0 annotation table for the AFFF sample (species, classes, homologous series, level-A structural assignments, retention times, mass defects)

## Outputs

- afff_annotation_comparison_table.xlsx with integer fields for classes covered (16/17), identical level-A classes (13), species counts (87 vs 50)
- Confident-series table with exactly 22 rows (series with confidently assigned structures out of 50)

## How to apply

Load the previously published manual annotation (supplied as an ESM Excel comparison file) and the FluoroMatch 2.0 annotation table for the AFFF sample using pandas/openpyxl. Align homologous series between the automated and manual annotations, matching series by PFAS class and by retention-time/mass-defect ordering. When tallying species assigned per matched series, exclude branched isomers so counts are directly comparable (87 species for FluoroMatch 2.0 vs 50 for the manual workflow). Count PFAS classes covered (16 of 17) and classes whose level-A structural assignments are identical between approaches (13), then tabulate series with confident structures (22 of 50). Write the comparison to afff_annotation_comparison_table.xlsx; the confident-series table must contain exactly 22 rows. Validate that all integer fields equal the reported values before considering the benchmark complete.

## Related tools

- **FluoroMatch 2.0** (Produces the automated non-targeted annotation table for the AFFF sample that is benchmarked against the manual workflow) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch Modular** (Modular pipeline used to run/adjust the annotation steps whose output is compared; code edits are integrated by replacing Modular.r in the FluoroMatch_Modular directory) — github.com/InnovativeOmics/Core-Match

## Examples

```
import pandas as pd; auto = pd.read_excel('FluoroMatch2.0_AFFF_annotation.xlsx'); manual = pd.read_excel('ESM_AFFF_comparison.xlsx'); auto.to_excel('afff_annotation_comparison_table.xlsx')
```

## Evaluation signals

- Comparison table's integer fields equal the reported values: 16/17 PFAS classes, 13 identical level-A classes, 87 vs 50 species (excluding branched isomers), 22/50 confident series
- Confident-series table has row_count = 22
- Each matched homologous series is reconciled by both PFAS class and retention-time/mass-defect ordering before counting
- Branched isomers are excluded from every species tally (87, not a branched-inflated count)
- Output file afff_annotation_comparison_table.xlsx is written and loadable via pandas/openpyxl with the expected fields

## Limitations

- Comparison fidelity depends on correctly matching series by class and retention-time/mass-defect ordering; mismatched ordering can distort the 87 vs 50 species comparison
- Branched isomers are excluded from species counts, so isomer-level coverage is not benchmarked
- Level-A structural assignments are identical for only 13 of the 16 matched classes, so structural-level agreement is partial
- The benchmark reflects the EPA CompTox Chemicals Dashboard PFAS structure list (over 8000 structures); coverage is bounded by that catalogue

## Evidence
