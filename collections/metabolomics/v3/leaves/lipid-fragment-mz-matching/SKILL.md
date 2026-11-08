---
name: lipid-fragment-mz-matching
description: Use when feature tables paired with .ms2 tandem MS files are available
  and lipid identifications must be assigned by matching observed fragment m/z values
  to in silico library fragments of candidate lipids from MS1 exact-mass matching.
when_to_use_negative:
- Input MS/MS files are not convertible to .ms2 format (e.g. Waters vendor files,
  which LipidMatch does not currently support)
- No MS/MS spectra exist and identifications must rely solely on precursor m/z matching
  without fragment confirmation
- The candidate lipid's characteristic fragments fall outside the in silico fragmentation
  libraries (e.g. lipid types not covered by the 56 lipid types)
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: LipidMatch
  role: R-based tool that performs fragment m/z matching against in silico fragmentation
    libraries and assigns binary fragment presence flags
  repo: http://secim.ufl.edu/secim-tools/
- name: R
  role: Language in which LipidMatch is written and executed for the fragment-matching
    workflow
- name: MSConvert
  role: Converts vendor raw MS/MS data to .ms2 file format required as input for fragment
    matching
- name: LipidBlast
  role: In silico fragmentation database whose structurally indicative fragments were
    compiled into LipidMatch's libraries
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-017-1744-3
    title: 'LipidMatch: an automated workflow for rule-based lipid identification
      using untargeted high-resolution tandem mass spectrometry data'
  source_content_hash: sha256:d1929e0a909042e25e134ab0751184a86327e7e9fc3f6d9a0fcb0620c4cc4a35
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidBlast
  - LipidMatch
  - MSConvert
  - R
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lipid-fragment-mz-matching

## Summary

Matches experimental MS/MS fragment m/z values against LipidMatch's in silico fragmentation library values (over 250,000 lipid species across 56 lipid types) within a ppm tolerance, to obtain rule-based lipid identifications for each LC-MS/MS feature.

## When to use

Use when feature tables paired with .ms2 tandem MS files are available and lipid identifications must be assigned by matching observed fragment m/z values to in silico library fragments of candidate lipids from MS1 exact-mass matching.

## When NOT to use

- Input MS/MS files are not convertible to .ms2 format (e.g. Waters vendor files, which LipidMatch does not currently support)
- No MS/MS spectra exist and identifications must rely solely on precursor m/z matching without fragment confirmation
- The candidate lipid's characteristic fragments fall outside the in silico fragmentation libraries (e.g. lipid types not covered by the 56 lipid types)

## Inputs

- Feature-lipid candidate pairs from MS1 exact-mass matching between in silico precursor ions and feature m/z
- MS/MS .ms2 files (converted from vendor raw data with MSConvert)
- User-defined parameters: RT tolerance, m/z tolerance (isolation window), ppm fragment tolerance, minimum intensity, minimum scan-count threshold (default 1 scan for orbitrap data)
- In silico fragmentation libraries in .csv format (e.g. LipidMatch lipid libraries covering over 250,000 lipid species across 56 lipid types)

## Outputs

- Per-lipid-class .csv fragment-statistics tables (scan count, average m/z, maximum intensity, RT at max intensity, binary presence flag per fragment)
- Feature tables with appended lipid identifications, ranked by summed fragment intensity
- Identification-level assignments (1: class and fatty acyl constituents; 2: data-independent analysis; 3: class only; 4: precursor m/z only)

## How to apply

In R/LipidMatch, load the feature-lipid candidate pairs from the MS1 exact-mass matching stage together with the .ms2 files (produced via MSConvert). For each feature-lipid pair, select MS/MS scans within a user-defined RT and m/z tolerance, where the m/z tolerance equals the isolation window used for ion selection. Match experimental fragment m/z values against the candidate's in silico fragment m/z values within a ppm tolerance window, computing for each matched fragment the number of scans containing it, its average m/z, its maximum intensity, and the retention time at maximum intensity across all scans. Assign each fragment a 1 if it exceeds the user-defined minimum intensity and minimum scan-count threshold (default 1 scan for orbitrap data) and 0 otherwise, then write per-lipid-class fragment-statistics tables (.csv) with scan count, average m/z, maximum intensity, RT at max intensity, and the binary flag. Lipids are identified if they contain the user-selected necessary fragments, after which identifications are ranked by the summed intensity of all exact-mass-matched fragments.

## Related tools

- **LipidMatch** (R-based tool that performs fragment m/z matching against in silico fragmentation libraries and assigns binary fragment presence flags) — http://secim.ufl.edu/secim-tools/
- **R** (Language in which LipidMatch is written and executed for the fragment-matching workflow)
- **MSConvert** (Converts vendor raw MS/MS data to .ms2 file format required as input for fragment matching)
- **LipidBlast** (In silico fragmentation database whose structurally indicative fragments were compiled into LipidMatch's libraries)

## Examples

```
Run LipidMatch via its gWidgets/tcltk dialogue boxes in R (Rscript LipidMatch.R), supplying the feature table, .ms2 files, and in silico lipid libraries in .csv format
```

## Evaluation signals

- Each fragment's binary flag equals 1 only when the fragment was found within the ppm tolerance and met both the minimum intensity and minimum scan-count thresholds; it is 0 when either criterion failed or the fragment was absent
- A .csv fragment-statistics table is saved for each corresponding lipid class, containing scan count, average m/z, maximum intensity, and RT at maximum intensity
- Lipid identifications are only appended when all user-selected necessary fragments for the lipid type are observed
- For features with identifications from multiple tools, cross-check that LipidMatch identifications by fatty acyl constituents agree with other software (92% positive mode, 98% negative mode in the Red Cross plasma benchmark)
- Identification levels (1-4) are correctly assigned based on whether fragments support class and fatty acyl constituents, data-independent analysis, class only, or precursor m/z only

## Limitations

- Default minimum scan-count threshold is 1 scan, tuned for orbitrap data; may need adjustment for other instruments
- The m/z tolerance for scan selection equals the isolation window used for ion selection, so it must be set consistently with acquisition parameters
- Software was only tested on Windows 7 through 10 and pinned to R version 3.3.3; no cross-platform support stated
- Users must develop their own fragmentation libraries in Excel for full customization of lipid types or adducts not covered
- Depends on external peak-picking software (MZmine, XCMS, MS-DIAL) and MSConvert to generate .ms2 files

## Evidence

- [methods] fragment matching with ppm tolerance and scan statistics: "experimental fragments are matched against in silico lipid fragments m/z using a tolerance window (ppm). The total number of scans across a feature containing that fragment is calculated."
- [methods] binary assignment thresholds: "Each fragment is assigned 1 if it is above the user defined minimum intensity and scans threshold and 0 if the fragment does not meet these criteria or was not found within the m/z tolerance"
- [abstract] ranking by summed intensity: "Another unique feature of LipidMatch is ranking lipid identifications for a given feature by the sum of fragment intensities."
- [abstract] library scale: "LipidMatch currently has over 250,000 lipid species spanning 56 lipid types contained in in silico fragmentation libraries."
- [readme] instrument and acquisition scope: "LipidMatch has been tested and validated using Q-Exactive orbitrap UHPLC-HRMS/MS data obtained from multiple sample types using targeted, data-dependent top-N (ddMS2-topN), and all ion fragmentation"
