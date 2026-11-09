---
name: fdr-multiple-comparison-correction
description: Use when assigning significance categories to metabolites whose many
  curve-fitted ROI peaks (SRRMs) each get a p-value, preventing uncorrected p<0.05
  calls from inflating identifications in NMR metabolomics.
when_to_use_negative:
- The analysis is a single pre-registered comparison with one ROI per metabolite,
  so no multiplicity across SRRMs/ROIs exists to correct
- The goal is validating the accuracy of user-defined ROIs or absolute quantification
  — the article states ROIAL-NMR was not built to test ROI accuracy and absolute quantification
  is not its aim
- Input is raw FID/spectral data without curve-fitted ROI p-values; FDR correction
  operates on ROI-level significance calls, not on the spectra directly
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3173
tools:
- name: ROIAL-NMR
  role: Applies the FDR-significance determination and trend-override logic within
    its candidate filtering and six-category metabolite categorization
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: pandas 2.2.3
  role: Pinned dependency used by the ROIAL-NMR pipeline that performs the FDR-corrected
    significance calculations on ROI/SRRM tables
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/nbm.70131
    title: Automatic Identification of Potential Cellular Metabolites for Untargeted
      NMR Metabolomics
  source_content_hash: sha256:314266615d3a24170c23181e638e8d18550f80361b40b2958ca802e378f99027
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
  tools:
  - PyQt5
  - Python
  - ROIAL-NMR
  - XlsxWriter
  - openpyxl
  - pandas
  - pandas 2.2.3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# fdr-multiple-comparison-correction

## Summary

False-discovery-rate (FDR) correction applied to repeated significance testing across user-defined NMR spectral ROIs and their SRRMs in ROIAL-NMR, so that metabolites remain flagged as FDR-significant when at least one of their SRRMs passes p<0.05 after correction.

## When to use

Use when assigning significance categories to metabolites whose many curve-fitted ROI peaks (SRRMs) each get a p-value, preventing uncorrected p<0.05 calls from inflating identifications in NMR metabolomics.

## When NOT to use

- The analysis is a single pre-registered comparison with one ROI per metabolite, so no multiplicity across SRRMs/ROIs exists to correct
- The goal is validating the accuracy of user-defined ROIs or absolute quantification — the article states ROIAL-NMR was not built to test ROI accuracy and absolute quantification is not its aim
- Input is raw FID/spectral data without curve-fitted ROI p-values; FDR correction operates on ROI-level significance calls, not on the spectra directly

## Inputs

- Per-ROI p-values for each candidate metabolite's SRRMs (curve-fitted ROI peaks from 0.5-4.5 ppm proton NMR spectra)
- User-defined ROI table with trends ('+'/'-') and significance levels ('*' level 1, '!' level 2)
- Matched metabolite list from the HMDB-derived serum database (891 metabolites) with match ratio >= 50%

## Outputs

- Per-metabolite FDR-significance tags (significant if >=1 SRRM remains p<0.05 after FDR correction)
- Metabolite category assignments among the six increase/decrease significance categories in the 'Result Show' table
- Entries persisted to the 'dataResult' file, organized by three significance levels and per-ROI tables

## How to apply

After the user-defined ROI table (trends marked '+'/'-', significance levels marked '*'/'!') is matched against the HMDB serum database of 891 metabolites, each metabolite's SRRMs are tested per ROI with p<0.05 as the raw threshold. Apply FDR correction across these repeated ROI-level tests; a metabolite is considered FDR-significant if at least one of its SRRMs remains significant after correction (p<0.05). FDR-significant ROIs then override signal-intensity trends in non-significant ROIs, and the metabolite is placed in one of six categories (e.g., 3) increase, FDR-significant; 6) decrease, FDR-significant). The significance ratio filter (>50% of SRRMs significant at p<0.05) runs alongside, and the corrected calls feed the final 'Result Show' table and 'dataResult' file used in the LC vs healthy control and LC-ADRD vs LC-non-ADRD comparisons.

## Related tools

- **ROIAL-NMR** (Applies the FDR-significance determination and trend-override logic within its candidate filtering and six-category metabolite categorization) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **pandas 2.2.3** (Pinned dependency used by the ROIAL-NMR pipeline that performs the FDR-corrected significance calculations on ROI/SRRM tables) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git

## Examples

```
python main.py  # then in the PyQt5 GUI: set ROIs/trends/significance levels ('*','!'), run matching vs the HMDB serum database, and FDR-corrected significance appears in the 'Result Show' / 'dataResult' output
```

## Evaluation signals

- Every FDR-significant metabolite has at least one SRRM with corrected p<0.05; metabolites with no surviving SRRM are not tagged FDR-significant
- In the reproduced serum study run, FDR-corrected categorization feeds counts that total 88 potential metabolites, with 66 relevant to LC and 80 relevant to ADRD
- The 'dataResult' file exists and contains metabolites categorized by the three significance levels (including the FDR-significant level) plus per-ROI tables
- FDR-significant ROI trends override any conflicting trends from non-significant ROIs in the assigned six-category label
- Run completes in the expected time window (~5 s single-population, <10 s multi-group), indicating the correction is embedded in the fast filtering path

## Limitations

- Results depend on user consistency in creating ROIs; variability between users is acknowledged, and ROI trends with '*'/'!' levels drive which tests exist before FDR correction
- ROIAL-NMR was not built to test the accuracy of the user-defined ROIs, so FDR-corrected significance inherits any ROI specification errors
- Match ratio calculation, scoring scheme, and filter parameter defaults are not fully described in all sections; FDR correction settings must be inspected in the code
- Exceeding the analyzed region: if more than 50% of a metabolite's peaks lie beyond the analyzed region, those peaks are excluded from the match ratio, which can shrink the SRRM set available for FDR testing

## Evidence
