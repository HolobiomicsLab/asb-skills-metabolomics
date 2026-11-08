---
name: metabolite-candidate-filtering
description: Use when candidate metabolites have been matched against user-defined
  NMR ROIs from an HMDB-style chemical shift/SRRM database and must be filtered by
  cross-ROI match ratio, significance, and concentration before building a final categorized
  output table.
when_to_use_negative:
- Automated identification of ROIs is required — this is beyond the scope of the current
  version; ROIs must be user-defined
- Absolute quantification of metabolite concentrations is the goal — the program does
  not primarily aim at absolute quantification
- Validation of the accuracy of user-defined ROIs is needed — ROIAL-NMR was not built
  to test ROI accuracy
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: ROIAL-NMR
  role: Implements the match-ratio filter, significance/FDR categorization, concentration
    filter, and Result Show/dataResult output
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: Python
  role: Runtime (>=3.9) for executing the ROIAL-NMR filtering pipeline
- name: pandas
  role: Loads and manipulates the candidate metabolite tables, ROI tables, and HMDB
    chemical shift/SRRM database
- name: XlsxWriter
  role: Writes the final categorized tables to the dataResult Excel file
- name: openpyxl
  role: Reads/writes the Excel-format dataResult output
- name: PyQt5
  role: GUI framework for the 'All Metabolites' and 'Result Show' windows where filtered
    metabolites are reviewed
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1002/nbm.70131
    title: Automatic Identification of Potential Cellular Metabolites for Untargeted
      NMR Metabolomics
  source_content_hash: sha256:5bda48da60a8ee3a05c1ff3ff5e038cdafc251e5184ecaf89860317266f56994
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# metabolite-candidate-filtering

## Summary

A filtering skill from ROIAL-NMR that prunes candidate metabolites matched to user-defined proton NMR regions-of-interest (ROIs) using a >=50% SRRM match ratio, peak-exclusion beyond the analyzed 0.5-4.5 ppm region, and downstream significance/concentration (>=5 uM) thresholds, to avoid over-identification in crowded spectra.

## When to use

Use when candidate metabolites have been matched against user-defined NMR ROIs from an HMDB-style chemical shift/SRRM database and must be filtered by cross-ROI match ratio, significance, and concentration before building a final categorized output table.

## When NOT to use

- Automated identification of ROIs is required — this is beyond the scope of the current version; ROIs must be user-defined
- Absolute quantification of metabolite concentrations is the goal — the program does not primarily aim at absolute quantification
- Validation of the accuracy of user-defined ROIs is needed — ROIAL-NMR was not built to test ROI accuracy

## Inputs

- Per-ROI candidate metabolite table produced by ROI-to-HMDB matching (task_003 output)
- HMDB-derived database of chemical shift peaks and metabolite-specific SRRMs (891 metabolites demonstrated)
- ROI table with trend markers ('+' increase / '-' decrease) and significance markers ('*' Level 1 / '!' Level 2)
- User-defined analyzed spectral region (e.g., 0.5-4.5 ppm for biofluids)
- Optional user-supplied disease-specific metabolite list (e.g., lung cancer, Alzheimer's disease, prostate cancer lists shipped with ROIAL-NMR)

## Outputs

- Scored metabolite table with match ratios and matched regions (tabular file)
- Final categorized dataResult Excel file assigning each metabolite to one of six categories (increase/decrease x not significant/significant/FDR-significant)

## How to apply

Load the per-ROI candidate metabolite table and the HMDB-derived chemical shift/SRRM database with pandas (Python >=3.9). Pre-filter by excluding peaks/SRRMs outside the analyzed spectral region (e.g., 0.5-4.5 ppm for biofluids); if more than 50% of a metabolite's peaks lie beyond the analyzed region, exclude those peaks from the match-ratio calculation. Count a metabolite-specific SRRM as matched to a ROI when at least one of its chemical shift peaks falls within the experimentally determined ROI boundaries (e.g., lactate peaks at 1.324 and 4.111 ppm against ROIs [1.31-1.34] and [4.10-4.12] ppm), then compute the match ratio as matched SRRMs / total SRRMs and keep only metabolites with ratio >= 0.5 (>= 50%). For disease-versus-control studies, additionally compute a significance ratio (statistically significant ROIs at p<0.05 among SRRMs; >50% marks the metabolite significant), flag FDR-significance when at least one SRRM remains significant after FDR correction, and apply the HMDB concentration filter excluding metabolites below 5 uM (the lower limit of detection by most NMR instruments). Assign trends (increase '+', decrease '-') letting FDR-significant ROI trends override non-significant ones, tag disease-relevant metabolites against a user-supplied disease list, and write the six-category output (increase/decrease x not significant/significant/FDR-significant) to the dataResult Excel file via XlsxWriter/openpyxl.

## Related tools

- **ROIAL-NMR** (Implements the match-ratio filter, significance/FDR categorization, concentration filter, and Result Show/dataResult output) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **Python** (Runtime (>=3.9) for executing the ROIAL-NMR filtering pipeline)
- **pandas** (Loads and manipulates the candidate metabolite tables, ROI tables, and HMDB chemical shift/SRRM database)
- **XlsxWriter** (Writes the final categorized tables to the dataResult Excel file)
- **openpyxl** (Reads/writes the Excel-format dataResult output)
- **PyQt5** (GUI framework for the 'All Metabolites' and 'Result Show' windows where filtered metabolites are reviewed)

## Examples

```
python main.py
```

## Evaluation signals

- Reproduce the paper's lactate worked example: both HMDB SRRMs ([1.30-1.33] and [4.08-4.13] ppm) matched against ROIs [1.31-1.34] and [4.10-4.12] ppm must yield a match ratio of 2/2 = 1.0
- Metabolites with match ratio < 0.5 must be absent from the downstream categorization table
- Metabolites with HMDB concentration limits below 5 uM must not appear in the final dataResult table
- Reproduce the reported scale: 88 identified metabolites, with 66 differentiating LC from controls and 80 distinguishing LC-ADRD from LC-non-ADRD
- Verify the dataResult output exists in Excel format with metabolites assigned to exactly one of the six trend/significance categories

## Limitations

- Match ratio and filter parameter defaults are fixed but users can customize and adjust filter parameters according to study needs
- Results depend on user consistency in creating ROIs; variability between users is acknowledged
- Automated ROI identification is not implemented and is beyond the current scope of the program
- Absolute quantification of compounds is not the primary aim of the program
- ROIAL-NMR was not built to validate the accuracy of user-defined ROIs

## Evidence
