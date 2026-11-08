---
name: confidence-level-annotation-interpretation
description: Use when interpreting LC-HRMS/MS annotation outputs for PFAS (e.g., AFFF
  samples) where suspect-level (B/B−) calls must be benchmarked against level-A assignments
  or against vendor formula confirmation (Agilent MassHunter Find-by-Formula).
when_to_use_negative:
- Input is a targeted quantification dataset for well-known PFAS only (e.g., PFOA/PFOS
  panels) — this skill addresses non-targeted annotation, not targeted measurement
- The original FluoroMatch suspect-screening release was used with curated MS/MS spectra
  rather than the truly non-targeted FluoroMatch 2.0 workflow; results are not comparable
  to the confidence-level framework described here
- Features have already been filtered and assigned confidence levels upstream; applying
  this skill again risks double-counting
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FluoroMatch 2.0
  role: Produces the non-targeted annotations and the confidence-level scoring framework
    (level-A vs. B/B−) that this skill interprets
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Modular
  role: Modular R implementation used to run annotation workflows and validate suspect-level
    calls against ESM comparison tables
  repo: github.com/InnovativeOmics/Core-Match
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:425e44a1344dfe4cb4fdd78f7efa13c30a0dcef1d488644d0eedd1c34ce82878
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FluoroMatch 2.0
  - FluoroMatch Modular
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# confidence-level-annotation-interpretation

## Summary

Interprets FluoroMatch 2.0 non-targeted PFAS annotations through its comprehensive confidence-level scoring framework (e.g., level-A structural assignments vs. in-silico fragment-based B and B− levels), integrating layers of evidence to reduce overreporting.

## When to use

Use when interpreting LC-HRMS/MS annotation outputs for PFAS (e.g., AFFF samples) where suspect-level (B/B−) calls must be benchmarked against level-A assignments or against vendor formula confirmation (Agilent MassHunter Find-by-Formula).

## When NOT to use

- Input is a targeted quantification dataset for well-known PFAS only (e.g., PFOA/PFOS panels) — this skill addresses non-targeted annotation, not targeted measurement
- The original FluoroMatch suspect-screening release was used with curated MS/MS spectra rather than the truly non-targeted FluoroMatch 2.0 workflow; results are not comparable to the confidence-level framework described here
- Features have already been filtered and assigned confidence levels upstream; applying this skill again risks double-counting

## Inputs

- FluoroMatch 2.0 annotation output table (LC-HRMS/MS features with confidence levels A/B/B−)
- ESM Excel comparison file of previously published manual AFFF annotation
- Agilent MassHunter Find-by-Formula confirmation results (ESM validation tables)
- EPA CompTox Chemicals Dashboard PFAS structure list (8000+ structures) for exact mass matching

## Outputs

- afff_annotation_comparison_table.xlsx (comparison of automated vs. manual annotation: 16/17 classes, 13 identical level-A classes, 87 vs. 50 species, 22/50 confident series)
- validation_statistics.csv (3/58 B/B− vs. level-A disagreements; 78/11/11 B and 33/26/40 B− Find-by-Formula confirmation fractions; Br false-positive flag)
- Per-annotation confidence level with integrated evidence layers for expert review

## How to apply

Load the FluoroMatch 2.0 annotation output (pandas/openpyxl for the ESM Excel comparison files) and classify each annotation by its evidence layer: level-A confident structures, in-silico fragment-based B/B− annotations, or MS/MS/predicted-fragmentation matches. Benchmark confidence levels against ground truth by comparing the 58 in-silico B/B− annotations to level-A assignments (expected: 3 of 58 disagreements, ~5%) and by tallying Agilent MassHunter Find-by-Formula confirmation categories (B suspects: 78% confident, 11% tentative, 11% not found; B− suspects: 33% confident, 26% tentative, 40% not found). Flag suspects whose class-diagnostics fail — e.g., a bromine-containing B-series proven false by the absence of the characteristic Br isotope pattern — and reserve such calls for expert review rather than automated acceptance. When reproducing the AFFF benchmark, exclude branched isomers when tallying species per matched homologous series (87 automated vs. 50 manual), count PFAS classes covered (16 of 17) and classes with identical level-A structures (13), and tabulate series with confident structures (22 of 50). Write results to structured tables (afff_annotation_comparison_table.xlsx, validation_statistics.csv) so integer fields and row counts can be verified against reference values.

## Related tools

- **FluoroMatch 2.0** (Produces the non-targeted annotations and the confidence-level scoring framework (level-A vs. B/B−) that this skill interprets) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch Modular** (Modular R implementation used to run annotation workflows and validate suspect-level calls against ESM comparison tables) — github.com/InnovativeOmics/Core-Match

## Examples

```
FLOW <- FALSE; Lipid <- FALSE; csvInput <- FALSE; ManuallyInputVariables <- FALSE; source('LipidMatch-4.2/FluoroMatch_Modular/Modular.r')
```

## Evaluation signals

- Comparison table integer fields exactly equal reference values: 16/17 PFAS classes, 13 identical level-A classes, 87 vs. 50 species (excluding branched isomers), 22/50 confident series
- Confident-series table has row_count = 22
- Disagreement table matches 3/58 between in-silico B/B− annotations and level-A assignments
- Find-by-Formula confirmation fractions match 78/11/11 (B) and 33/26/40 (B−) for confident/tentative/not found
- The bromine-containing false-positive B-series is flagged with absent Br isotope pattern and marked for expert review

## Limitations

- B− (negative-mode in-silico fragment-based) annotations are much less reliable than B annotations: only 33% confirmed confidently vs. 78%, with 40% not found
- One B-series of bromine-containing PFAS was proven false by the absence of the characteristic Br isotope pattern — class-diagnostic isotope checks remain necessary
- Confident structures were assigned to only 22 of 50 homologous series, so many series retain uncertain structural level

## Evidence
