---
name: fragment-intensity-threshold-binning
description: Use when you have a feature-lipid candidate table from exact-mass MS1
  matching and need to decide which experimental fragments in ddMS2-topN .ms2 data
  are trustworthy evidence for confirming each lipid.
when_to_use_negative:
- The input is raw vendor files without prior peak picking and .ms2 conversion, since
  the skill operates on pre-processed .ms2 scan data.
- You only need precursor m/z-level identifications and have no MS/MS data, since
  the binarization depends on fragment-level evidence.
- The instrument produces Waters files, which LipidMatch does not currently support.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: LipidMatch
  role: R software implementing the MS/MS scan selection, fragment matching, statistics
    computation, and binary threshold assignment for each feature-lipid pair.
  repo: https://github.com/GarrettLab-UF/LipidMatch
- name: R
  role: Language used to implement the binarization logic, including scan selection
    and intensity/scan-count thresholding.
- name: MSConvert
  role: Used upstream to convert vendor MS/MS data into the .ms2 format that LipidMatch
    scans during binarization.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-017-1744-3
    title: 'LipidMatch: an automated workflow for rule-based lipid identification
      using untargeted high-resolution tandem mass spectrometry data'
  source_content_hash: sha256:dcdee9e72d46b33079901b5107b272989d4f4744f76b99eb5397a2b65cb12dd0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GarrettLab-UF/LipidMatch
  tools:
  - LipidMatch
  - MSConvert
  - R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# fragment-intensity-threshold-binning

## Summary

Binarize candidate lipid fragments by tagging each matched in silico fragment with a 1 or 0 depending on whether it exceeds user-defined minimum intensity and minimum scan-count thresholds across MS/MS scans, then rank identifications by summed fragment intensity.

## When to use

Use when you have a feature-lipid candidate table from exact-mass MS1 matching and need to decide which experimental fragments in ddMS2-topN .ms2 data are trustworthy evidence for confirming each lipid.

## When NOT to use

- The input is raw vendor files without prior peak picking and .ms2 conversion, since the skill operates on pre-processed .ms2 scan data.
- You only need precursor m/z-level identifications and have no MS/MS data, since the binarization depends on fragment-level evidence.
- The instrument produces Waters files, which LipidMatch does not currently support.

## Inputs

- Feature-lipid candidate pairs from the MS1 exact-mass matching stage
- .ms2 MS/MS files (generated via MSConvert) for pooled or representative samples
- In silico fragmentation library for the lipid class (e.g., LipidBlast-derived .csv)

## Outputs

- Per-lipid-class .csv table with fragment scan count, average m/z, maximum intensity, RT at max intensity, and a 0/1 binary flag
- Feature table appended with ranked lipid identifications based on summed fragment intensity

## How to apply

For each feature-lipid pair, select MS/MS scans from the .ms2 files within a user-defined retention-time and m/z tolerance where the m/z tolerance equals the isolation window used for ion selection. Match experimental fragment m/z values against the candidate's in silico fragment m/z within a ppm tolerance window and, for each matched fragment, compute the total number of scans containing it, its average m/z, its maximum intensity, and the retention time at maximum intensity across all scans. Assign each fragment a 1 if it exceeds the user-defined minimum intensity and minimum scan-count threshold (default 1 scan for orbitrap data) and 0 otherwise, producing a binary fragment-presence flag for every feature-lipid pair. Write the resulting statistics (scan count, average m/z, maximum intensity, RT at max intensity, binary flag) as a .csv table per lipid class. Rank candidate lipids by the summed intensity of all their fragments with in silico exact mass matches, including those not used for confirmation.

## Related tools

- **LipidMatch** (R software implementing the MS/MS scan selection, fragment matching, statistics computation, and binary threshold assignment for each feature-lipid pair.) — https://github.com/GarrettLab-UF/LipidMatch
- **R** (Language used to implement the binarization logic, including scan selection and intensity/scan-count thresholding.)
- **MSConvert** (Used upstream to convert vendor MS/MS data into the .ms2 format that LipidMatch scans during binarization.)

## Examples

```
Rscript LipidMatch.R --input feature_table.csv --ms2 pooled.ms2 --rt-tol 0.3 --mz-tol-isolation 1.0 --ppm-tol 5 --min-intensity 1000 --min-scans 1
```

## Evaluation signals

- Each fragment's binary flag is 1 only when the fragment was found within the ppm tolerance and met both the intensity and scan-count thresholds, and 0 when it did not or was absent.
- A .csv table is saved for each lipid class and contains the columns for scan count, average m/z, maximum intensity, RT at max intensity, and binary flag.
- Ranking of candidate lipids for a feature is based on the summed intensity of all fragments with in silico exact mass matches, including those not used for confirmation.
- Default minimum scan-count threshold is 1 scan for orbitrap data; lower counts than this should result in a binary flag of 0.

## Limitations

- Software only tested on Windows 7 through 10; no cross-platform support stated.
- Pinned to R version 3.3.3 (older R version requirement).
- Depends on external MSConvert or equivalent for generating .ms2 files.
- Users must develop their own fragmentation libraries in Excel for full customization.

## Evidence

- [methods] Assign fragments 0/1 based on intensity and scan thresholds: "Each fragment is assigned 1 if it is above the user defined minimum intensity and scans threshold and 0 if the fragment does not meet these criteria or was not found within the m/z tolerance (Step 4;"
- [methods] Match fragments and calculate statistics: "experimental fragments are matched against in silico lipid fragments m/z using a tolerance window (ppm). The total number of scans across a feature containing that fragment is calculated."
- [methods] MS/MS scan selection: "MS/MS scans from .ms2 files within a user defined retention time and m/z tolerance of each feature is determined (Step 2; Fig. 3)."
- [methods] Rank lipids by summed fragment intensity: "the lipids are ranked by the summed intensity of all their fragments with in silico fragment exact mass matches, including those not used for confirmation (Step 7, Fig. 3)."
- [readme] Supported input format and scope: "LipidMatch has been tested and validated using Q-Exactive orbitrap UHPLC-HRMS/MS data obtained from multiple sample types using targeted, data-dependent top-N (ddMS2-topN), and all ion fragmentation"
- [readme] Library scale: "in-silico fragmentation libraries of over 500,000 lipid species across over 60 lipid types making it one of the most comprehensive open-source software"
- [methods] Upstream processing requirement: "Peak picking of MS/MS data and conversion to .ms2 file format should be done using MSConvert [16]."
