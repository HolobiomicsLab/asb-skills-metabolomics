---
name: feature-blank-filtering
description: Use when raw vendor feature tables (e.g. a sample plus its solvent blanks)
  contain background features that must be stripped before PFAS annotation, such as
  an AFFF sample table with tens of thousands of detected features dominated by non-analyte
  background.
when_to_use_negative:
- No matched solvent-blank feature tables exist for the sample — the blank average
  + 3× SD criterion cannot be computed
- The feature table has already been blank-filtered and the goal is downstream annotation
  (exact mass matching, mass defect filtering, homologous series detection)
- The analysis target is MS/MS-based PFAS annotation rather than background removal
  from MS1 feature lists
edam_operation: http://edamontology.org/operation_3695
tools:
- name: FluoroMatch 2.0
  role: Provides the automated non-targeted PFAS workflow whose feature blank filtering
    step this skill implements
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Modular
  role: Modular (FLOW <- FALSE) execution mode in which the blank-filtering workflow
    is run on sample and blank vendor feature tables
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch
  role: Original suspect-screening release whose FluoroMatch_Manual.docx documents
    setup and dependencies for the Modular code
  repo: github.com/InnovativeOmics/Core-Match
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:b1587d02e4bbb5adce48febd4c871115af4a5158129cf1550a30bae540985f80
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FluoroMatch
  - FluoroMatch 2.0
  - FluoroMatch Modular
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# feature-blank-filtering

## Summary

Removes background/background-contaminant features from an LC-HRMS feature table by comparing each feature's sample abundance against matched solvent-blank abundances, as implemented in the FluoroMatch 2.0 feature blank filtering workflow. It is the second step of the FluoroMatch pipeline, immediately after feature detection and before exact mass matching.

## When to use

Use when raw vendor feature tables (e.g. a sample plus its solvent blanks) contain background features that must be stripped before PFAS annotation, such as an AFFF sample table with tens of thousands of detected features dominated by non-analyte background.

## When NOT to use

- No matched solvent-blank feature tables exist for the sample — the blank average + 3× SD criterion cannot be computed
- The feature table has already been blank-filtered and the goal is downstream annotation (exact mass matching, mass defect filtering, homologous series detection)
- The analysis target is MS/MS-based PFAS annotation rather than background removal from MS1 feature lists

## Inputs

- Sample vendor feature table (e.g. AFFF LC-HRMS/MS vendor feature list, e.g. 31,132 features)
- Three solvent-blank vendor feature tables matched to the sample run

## Outputs

- blank_filtered_features.csv containing the surviving features (reference: 3740 rows for the AFFF dataset)
- Removed-feature set representing background/background-contaminant features (~88% of detected features on the reference AFFF data)

## How to apply

Load the sample vendor feature table (e.g. AFFF) and the three solvent-blank vendor feature tables into FluoroMatch Modular's blank-filtering workflow. Align features between sample and blanks with a precursor mass tolerance of ±0.005 Da. For each feature, compute the average and standard deviation of abundance across the three solvent blanks and flag any feature whose sample maximum is not greater than blank average + 3× blank SD; remove all flagged features. Export the surviving features (e.g. blank_filtered_features.csv) and check the removed fraction against the reference outcome — on the AFFF data, filtering reduced 31,132 detected features to 3740, an ≈88% removal, within FluoroMatch 2.0's broader finding that 96% of features likely not PFAS were filtered out.

## Related tools

- **FluoroMatch 2.0** (Provides the automated non-targeted PFAS workflow whose feature blank filtering step this skill implements) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch Modular** (Modular (FLOW <- FALSE) execution mode in which the blank-filtering workflow is run on sample and blank vendor feature tables) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch** (Original suspect-screening release whose FluoroMatch_Manual.docx documents setup and dependencies for the Modular code) — github.com/InnovativeOmics/Core-Match

## Examples

```
Rscript Modular.r  # FLOW <- FALSE; csvInput <- FALSE; ManuallyInputVariables <- FALSE
```

## Evaluation signals

- Output table contains exactly 3740 rows on the reference AFFF dataset (31,132 features in → 3740 out)
- Removed fraction matches the reported blank-filtering outcome (≈88% of detected features removed)
- Every retained feature's sample maximum exceeds its three-blank average + 3× blank SD under the ±0.005 Da precursor mass tolerance alignment
- No feature absent from all solvent blanks is removed
- Exported file is a valid feature table (blank_filtered_features.csv) suitable for the next FluoroMatch steps (exact mass matching, mass defect filtering)

## Limitations

- Features present in the sample but at abundances near or below blank average + 3× SD will be discarded, so the filter trades sensitivity for background removal
- FluoroMatch Modular must be manually configured (FLOW <- FALSE, csvInput/ManuallyInputVariables toggles, Lipid/TWeen_pos parameters) and dependencies installed per FluoroMatch_Manual.docx; misconfiguration defaults the tool to PFAS analysis mode only if Lipid and TWeen_pos are both FALSE
- Blank-filtering alone does not confirm PFAS identity — it only removes likely non-PFAS background; subsequent exact mass matching and MS/MS screening steps are required for annotation

## Evidence
