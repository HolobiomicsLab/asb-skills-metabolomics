---
name: serum-metabolomics-group-comparison-analysis
description: Use when a serum (or other biofluid) 1H NMR study has user-defined spectral
  ROIs with trend ('+'/'-') and significance ('*', '!') annotations, and the goal
  is to identify which HMDB metabolites fall within those ROIs and which differentiate
  disease from control groups.
when_to_use_negative:
- When the goal is automated ROI identification from spectra — automated ROI identification
  is beyond the current scope of ROIAL-NMR and ROIs must be user-defined
- When absolute quantification of metabolites is required — absolute quantification
  is not the primary aim of the program
- When the objective is to validate the accuracy of user-defined ROIs — ROIAL-NMR
  was not built to test ROI accuracy
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: ROIAL-NMR
  role: Core program that identifies and categorizes potential metabolites from user-defined
    NMR ROIs against the HMDB and performs multi-group comparisons via its PyQt5 GUI
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: PyQt5 5.15.11
  role: GUI framework driving parameter setting, 'All Metabolites', 'Result Show',
    and Combine Analysis windows
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: pandas 2.2.3
  role: Pinned dependency for data handling in the ROIAL-NMR pipeline
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: XlsxWriter 3.2.2
  role: Pinned dependency for writing result files such as 'dataResult'
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: openpyxl 3.1.5
  role: Pinned dependency for spreadsheet file handling in ROIAL-NMR
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/nbm.70131
    title: Automatic Identification of Potential Cellular Metabolites for Untargeted
      NMR Metabolomics
  source_content_hash: sha256:291f815b5b3a481d48cf809f6a3f48f2e80bf7d00afd2d2e90cbe22358b2d014
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
  tools:
  - PyQt5
  - PyQt5 5.15.11
  - Python
  - ROIAL-NMR
  - XlsxWriter
  - XlsxWriter 3.2.2
  - openpyxl
  - openpyxl 3.1.5
  - pandas
  - pandas 2.2.3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# serum-metabolomics-group-comparison-analysis

## Summary

Uses the ROIAL-NMR Python/PyQt5 program to identify potential metabolites from user-defined proton NMR ROIs in human serum and categorize them across disease-vs-control comparisons, using the HMDB (891 metabolites' chemical shifts plus SRRMs) as the reference platform.

## When to use

Use when a serum (or other biofluid) 1H NMR study has user-defined spectral ROIs with trend ('+'/'-') and significance ('*', '!') annotations, and the goal is to identify which HMDB metabolites fall within those ROIs and which differentiate disease from control groups.

## When NOT to use

- When the goal is automated ROI identification from spectra — automated ROI identification is beyond the current scope of ROIAL-NMR and ROIs must be user-defined
- When absolute quantification of metabolites is required — absolute quantification is not the primary aim of the program
- When the objective is to validate the accuracy of user-defined ROIs — ROIAL-NMR was not built to test ROI accuracy

## Inputs

- User-defined serum proton NMR ROI table (18 curve-fitted ROIs over 0.5-4.5 ppm) with '+'/'-' trend and '*'/'!' significance-level annotations
- HMDB-derived serum metabolite database (chemical shifts plus SRRMs for 891 metabolites)
- Sample-type selection (serum) and analyzed spectral region (0.5-4.5 ppm)
- Group comparison definitions (e.g., LC vs healthy control; LC-ADRD vs LC-non-ADRD)
- Optional user-prepared disease-specific metabolite list for disease-relevance tagging

## Outputs

- 'All Metabolites' table with metabolite abbreviations, match ratios, matched regions, and concentration ranges
- 'dataResult' file with metabolites categorized by three significance levels plus per-ROI tables
- Identified metabolite counts per comparison (e.g., 88 total, 66 LC-vs-control, 80 LC-ADRD-relevant)
- Optional Combine Analysis table (overline = same trends in both groups; underline = opposite trends)

## How to apply

Install Python >=3.9 with pinned packages (pandas 2.2.3, PyQt5 5.15.11, XlsxWriter 3.2.2, openpyxl 3.1.5) and clone github.com/Leo-Cheng-Lab/ROIAL-NMR. In the PyQt5 parameter-setting step, input the 18 curve-fitted ROIs (0.5-4.5 ppm) with '+'/'-' trends, '*' (significance level 1) and '!' (significance level 2) annotations, select serum as the sample type, and set the analyzed region to 0.5-4.5 ppm. The program intersects ROI specifications with HMDB chemical shift values and SRRMs, keeping only candidates whose match ratio is >= 50% of their SRRMs within the analyzed ROIs, then applies a significance ratio filter (> 50% of SRRMs at p<0.05), FDR-significance tagging (at least one SRRM significant after FDR correction), trend override by FDR-significant ROIs, six increase/decrease significance categories, and a 5 μM concentration cutoff on HMDB biofluid values. Run the single-population and two multi-group comparisons (LC vs healthy control; LC-ADRD vs LC-non-ADRD), optionally using Combine Analysis to highlight metabolites shared by both comparisons, and save results to the 'dataResult' file. Rationale: metabolite identification requires the intersection of ROI specifications, metabolic NMR information, and sample-specific information; both chemical shift values and SRRMs are required for accurate identification.

## Related tools

- **ROIAL-NMR** (Core program that identifies and categorizes potential metabolites from user-defined NMR ROIs against the HMDB and performs multi-group comparisons via its PyQt5 GUI) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **PyQt5 5.15.11** (GUI framework driving parameter setting, 'All Metabolites', 'Result Show', and Combine Analysis windows) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **pandas 2.2.3** (Pinned dependency for data handling in the ROIAL-NMR pipeline) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **XlsxWriter 3.2.2** (Pinned dependency for writing result files such as 'dataResult') — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **openpyxl 3.1.5** (Pinned dependency for spreadsheet file handling in ROIAL-NMR) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git

## Examples

```
git clone https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git; cd ROIAL-NMR; python main.py
```

## Evaluation signals

- Run completes in the reported time range (~5 s single-population, <10 s multi-group) and the 'dataResult' file exists
- 'dataResult' contains metabolites categorized by three significance levels plus per-ROI tables
- Metabolite counts match the reported values: 88 potential metabolites total, 66 differentiating LC from healthy controls, 80 distinguishing LC-ADRD from LC-non-ADRD
- Every retained metabolite has a match ratio >= 50% of its SRRMs within the analyzed ROIs and an HMDB biofluid concentration above 5 μM
- Combine Analysis output marks shared metabolites with overline (same trends) / underline (opposite trends) across the two group comparisons

## Limitations

- Automated identification of ROIs is beyond the scope of the current version of the program; users must define ROIs consistently, and variability between users can lead to variable results
- Absolute quantification of compounds is not the primary aim of the program
- Program results depend on user consistency in creating ROIs; ROIAL-NMR was not built to test the accuracy of user-defined ROIs
- Automatic synchronization of NMR chemical shift databases is planned but not yet implemented

## Evidence
