---
name: spectral-region-boundary-filtering
description: Use when you have user-defined proton NMR ROIs (e.g., 18 ROIs spanning
  0.5-4.5 ppm of serum spectra) and need to systematically find which database metabolites
  (HMDB, 891 entries) possess resonances within those ROI boundaries.
when_to_use_negative:
- Absolute quantification of compounds is the goal — the program's authors state 'absolute
  quantification of the compounds is not the primary aim of the program'
- You need automated identification of ROIs from spectra — 'Automated identification
  of ROIs is beyond the scope of the current version of the program'
- You want to validate the accuracy or placement of user-defined ROIs — 'ROIAL-NMR
  was not built to test the accuracy of the user-defined ROIs'
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: ROIAL-NMR
  role: Implements the per-ROI boundary matching, exclusion of out-of-region peaks,
    match-ratio (>=50%) and concentration (>=5 uM) filters, and writes all_metabolites_matches.csv
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: pandas
  role: DataFrames used to load the HMDB metabolite library and ROI table and perform
    per-ROI matching and filtering (v2.2.3)
- name: Python >=3.9
  role: Runtime environment for ROIAL-NMR (with XlsxWriter 3.2.2, PyQt5 5.15.11, openpyxl
    3.1.5)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/nbm.70131
    title: Automatic Identification of Potential Cellular Metabolites for Untargeted
      NMR Metabolomics
  source_content_hash: sha256:ecd91bfb8c5c1fea5ebe8470094160e2c4cba70be0d413fcce8aa45c0633de0a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
  tools:
  - Python
  - Python >=3.9
  - ROIAL-NMR
  - pandas
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# spectral-region-boundary-filtering

## Summary

Boundary-matching filter from ROIAL-NMR that retrieves metabolites whose NMR chemical shift peaks fall inside user-defined spectral ROIs, then discards peaks outside the analyzed segment (0.5-4.5 ppm) and metabolites with >50% of peaks beyond the region before computing per-metabolite match ratios (matched SRRMs / total SRRMs, kept at >=50%).

## When to use

Use when you have user-defined proton NMR ROIs (e.g., 18 ROIs spanning 0.5-4.5 ppm of serum spectra) and need to systematically find which database metabolites (HMDB, 891 entries) possess resonances within those ROI boundaries.

## When NOT to use

- Absolute quantification of compounds is the goal — the program's authors state 'absolute quantification of the compounds is not the primary aim of the program'
- You need automated identification of ROIs from spectra — 'Automated identification of ROIs is beyond the scope of the current version of the program'
- You want to validate the accuracy or placement of user-defined ROIs — 'ROIAL-NMR was not built to test the accuracy of the user-defined ROIs'

## Inputs

- HMDB-derived metabolite library with chemical shift values and SRRMs for 891 metabolites (produced by the library-preparation step, task_001)
- User-defined ROI table (e.g., 18 ROIs spanning the 0.5-4.5 ppm up-field segment) with trends and significance levels
- Optional user-prepared disease-specific metabolite list for tagging
- Sample type selection and analyzed region boundaries

## Outputs

- all_metabolites_matches.csv — per-ROI matched metabolite table (abbreviation, match ratio, matched regions, HMDB concentration range)
- Match-ratio-filtered metabolite list (match ratio >= 50%) carried into significance categorization
- dataResult file with metabolites categorized by three significance levels and per-ROI groupings

## How to apply

Load the HMDB-derived library (chemical shift values and SRRMs for 891 metabolites) and the ROI table into pandas DataFrames, then for each ROI mark every metabolite with at least one peak inside the ROI boundaries as matched. Exclude peaks lying outside the analyzed 0.5-4.5 ppm segment and drop any metabolite with more than 50% of its peaks beyond the analyzed region so questionable metabolites cannot be overrepresented in ratio calculation. Compute each metabolite's match ratio as matched SRRMs divided by total SRRMs, keep only metabolites with a match ratio >= 50%, and exclude metabolites with concentration limits below 5 uM (the lower limit of detection by most NMR instruments). Attach matched region boundaries and HMDB concentration ranges, then write the per-ROI matched metabolite table to all_metabolites_matches.csv. Validate with the lactate control case: peaks at 1.324 ppm (ROI 1.31-1.34) and 4.111 ppm (ROI 4.10-4.12) must both match, yielding a match ratio of 2/2.

## Related tools

- **ROIAL-NMR** (Implements the per-ROI boundary matching, exclusion of out-of-region peaks, match-ratio (>=50%) and concentration (>=5 uM) filters, and writes all_metabolites_matches.csv) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **pandas** (DataFrames used to load the HMDB metabolite library and ROI table and perform per-ROI matching and filtering (v2.2.3))
- **Python >=3.9** (Runtime environment for ROIAL-NMR (with XlsxWriter 3.2.2, PyQt5 5.15.11, openpyxl 3.1.5))

## Examples

```
python main.py
```

## Evaluation signals

- Lactate control case reproduces the paper's result: peaks at 1.324 ppm (ROI 1.31-1.34) and 4.111 ppm (ROI 4.10-4.12) both match, giving a match ratio of 2/2
- Match ratio invariant: every retained metabolite has matched SRRMs / total SRRMs >= 50%
- Boundary invariant: no peak outside the analyzed 0.5-4.5 ppm segment contributes to any match ratio; metabolites with >50% of peaks beyond the region are dropped before ratio calculation
- Concentration invariant: no retained metabolite has concentration limits below 5 uM
- Runtime check: single-population analysis completes in about 5 seconds (multi-group in less than 10 seconds); replicate example finds 88 metabolites from 18 ROIs

## Limitations

- Results depend on user consistency in creating ROIs; 'using user-defined NMR spectral ROIs can lead to variable results obtained by different users'
- Automated identification of ROIs is not implemented; ROIs must be user-defined
- Absolute quantification of compounds is not supported
- Both chemical shift values and metabolite-specific SRRMs are required for accurate metabolite identification; chemical shifts alone are insufficient
- Automatic synchronization of NMR chemical shift databases (HMDB, BMRB) is planned but not yet implemented

## Evidence
