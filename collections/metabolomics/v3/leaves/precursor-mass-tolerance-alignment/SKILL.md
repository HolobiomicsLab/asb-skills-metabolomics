---
name: precursor-mass-tolerance-alignment
description: Use when untargeted LC-HRMS feature tables from a sample (e.g., AFFF)
  must be aligned to solvent-blank feature tables by precursor m/z to distinguish
  real sample features from background contamination.
when_to_use_negative:
- Input is already a blank-filtered feature list or the downstream annotation does
  not require background subtraction.
- The blank files are not from the same LC-HRMS acquisition batch, since the ±0.005
  Da precursor alignment assumes identical ion detection across sample and blank runs.
- 'You are processing lipids with LipidMatch rather than PFAS: the README notes the
  Lipid/TWeen flags must be toggled, otherwise Modular ''defaults to PFAS analysis''.'
edam_operation: http://edamontology.org/operation_3695
tools:
- name: FluoroMatch Modular
  role: Executes the blank-filtering workflow in which the ±0.005 Da precursor-mass
    alignment of sample vs. blank feature tables is performed; run with FLOW <- FALSE.
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch 2.0
  role: The non-targeted PFAS annotation pipeline whose feature blank filtering step
    this alignment supports.
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch
  role: Original suspect-screening release; latest stable version for users is downloaded
    from innovativeomics.com, with core algorithms edited via the Core-Match repository.
  repo: github.com/InnovativeOmics/Core-Match
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:21b43362289ff6d9cb3544491731de6c1ad323603decd8fb2861dbecd385fee2
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
yes_rate: 0.8181818181818182
n_claims: 11
---

# precursor-mass-tolerance-alignment

## Summary

Aligns sample and blank vendor feature tables by precursor mass (±0.005 Da) in FluoroMatch Modular so blank-derived features can be matched and removed as background before downstream PFAS annotation.

## When to use

Use when untargeted LC-HRMS feature tables from a sample (e.g., AFFF) must be aligned to solvent-blank feature tables by precursor m/z to distinguish real sample features from background contamination.

## When NOT to use

- Input is already a blank-filtered feature list or the downstream annotation does not require background subtraction.
- The blank files are not from the same LC-HRMS acquisition batch, since the ±0.005 Da precursor alignment assumes identical ion detection across sample and blank runs.
- You are processing lipids with LipidMatch rather than PFAS: the README notes the Lipid/TWeen flags must be toggled, otherwise Modular 'defaults to PFAS analysis'.

## Inputs

- Sample vendor feature table (e.g., AFFF sample, 31,132 detected features)
- Solvent-blank vendor feature tables (e.g., three solvent blanks)

## Outputs

- blank_filtered_features.csv (features not from background; e.g., 3740 surviving features from 31,132, ≈88% removed)

## How to apply

Load the sample vendor feature table and the solvent-blank vendor feature tables into FluoroMatch Modular's blank-filtering workflow, keeping Modular mode (FLOW <- FALSE) rather than Flow mode. Align features between the sample and blank files using a precursor mass tolerance of ±0.005 Da. For each aligned feature, compute the average and standard deviation of abundance across the blanks (e.g., three solvent blanks) and flag any feature whose sample maximum is not greater than blank average + 3× blank SD. Remove flagged features and export the surviving table (e.g., blank_filtered_features.csv). The strict ±0.005 Da tolerance ensures only the same ion observed in sample and blank is treated as background; the 3× SD threshold avoids removing real sample features that merely sit near blank noise.

## Related tools

- **FluoroMatch Modular** (Executes the blank-filtering workflow in which the ±0.005 Da precursor-mass alignment of sample vs. blank feature tables is performed; run with FLOW <- FALSE.) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch 2.0** (The non-targeted PFAS annotation pipeline whose feature blank filtering step this alignment supports.) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch** (Original suspect-screening release; latest stable version for users is downloaded from innovativeomics.com, with core algorithms edited via the Core-Match repository.) — github.com/InnovativeOmics/Core-Match

## Examples

```
Rscript Modular.r # with FLOW <- FALSE; csvInput <- TRUE; Lipid <- FALSE; TWeen_pos <- FALSE (README-specified Modular settings for PFAS analysis)
```

## Evaluation signals

- Exported blank_filtered_features.csv contains exactly 3740 rows for the AFFF test data (from 31,132 input features).
- Removed fraction matches the reported blank-filtering outcome (≈88% of detected features removed).
- Every removed feature has a blank counterpart within ±0.005 Da precursor mass in at least one solvent-blank table.
- No retained feature has sample maximum ≤ blank average + 3× blank SD.
- For a broader sanity check, the paper reports 96% of features likely not PFAS were filtered out in the full Flow 2.0 pipeline.

## Limitations

- The ±0.005 Da tolerance and 3× blank SD rule are specific to this vendor-table-based blank-filtering implementation; other data may need recalibrated thresholds.
- Users should get the latest stable FluoroMatch from innovativeomics.com; only developers should edit Core-Match code and integrate edited Modular.r into LipidMatch-4.2\FluoroMatch_Modular.
- Feature blank filtering is only the first of many steps (exact mass matching, mass defect filtering, homologous series detection, RT pattern analysis, MS/MS screening); alignment quality alone does not guarantee PFAS annotation.

## Evidence
