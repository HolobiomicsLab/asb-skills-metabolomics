---
name: nmr-region-of-interest-metabolite-identification
description: Use when you have 1D proton NMR spectra of biological samples (serum,
  saliva, urine, CSF, sweat, tissues) with pre-determined ROIs, trends, and significance
  levels, and need to identify which HMDB metabolites explain those ROI patterns.
when_to_use_negative:
- You need automated ROI identification from raw spectra — this is beyond the current
  scope of ROIAL-NMR
- You need absolute quantification of metabolites — this is not the primary aim of
  the program
- You need to validate the accuracy of user-defined ROIs — the program was not built
  to test ROI accuracy
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0593
tools:
- name: ROIAL-NMR
  role: Python program with PyQt5 GUI that identifies and categorizes potential metabolites
    from user-defined proton NMR ROIs using HMDB as reference
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: Python >=3.9
  role: Runtime environment for ROIAL-NMR
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: pandas 2.2.3
  role: Pinned dependency of ROIAL-NMR for data handling
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: PyQt5 5.15.11
  role: Pinned dependency providing the ROIAL-NMR graphical interface
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: XlsxWriter 3.2.2
  role: Pinned dependency for writing the 'dataResult' spreadsheet output
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: openpyxl 3.1.5
  role: Pinned dependency for reading/writing Excel inputs and outputs
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: Human Metabolome Database (HMDB)
  role: Reference platform providing chemical shift values and SRRMs for 891 metabolites
    used for identification
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/nbm.70131
    title: Automatic Identification of Potential Cellular Metabolites for Untargeted
      NMR Metabolomics
  source_content_hash: sha256:3c6a02f07010ffc8c37804bf1f7eef04a8f83db87d4dff3d3da6e16a1fc1ccd2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
  tools:
  - Human Metabolome Database (HMDB)
  - PyQt5
  - PyQt5 5.15.11
  - Python
  - Python >=3.9
  - ROIAL-NMR
  - XlsxWriter
  - XlsxWriter 3.2.2
  - openpyxl
  - openpyxl 3.1.5
  - pandas
  - pandas 2.2.3
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# nmr-region-of-interest-metabolite-identification

## Summary

Use ROIAL-NMR, a Python/PyQt5 program, to systematically identify and categorize potential metabolites from user-defined proton NMR spectral regions-of-interest (ROIs) against the HMDB reference database, supporting disease-vs-control metabolomics studies such as lung cancer serum analysis.

## When to use

Use when you have 1D proton NMR spectra of biological samples (serum, saliva, urine, CSF, sweat, tissues) with pre-determined ROIs, trends, and significance levels, and need to identify which HMDB metabolites explain those ROI patterns.

## When NOT to use

- You need automated ROI identification from raw spectra — this is beyond the current scope of ROIAL-NMR
- You need absolute quantification of metabolites — this is not the primary aim of the program
- You need to validate the accuracy of user-defined ROIs — the program was not built to test ROI accuracy

## Inputs

- User-defined table of proton NMR ROIs with trends ('+'/'-') and significance levels ('*'/'!')
- 1D proton NMR spectra of biological samples (e.g., serum measured on a Bruker Avance III 600 MHz spectrometer)
- Sample type selection (serum, saliva, sweat, urine, CSF, tissue)
- Analyzed spectral region specification (e.g., 0.5-4.5 ppm)
- HMDB-derived metabolite database (chemical shift values plus SRRMs for 891 metabolites)
- Optional disease-specific metabolite list for highlighting disease-relevant metabolites

## Outputs

- 'All Metabolites' table with abbreviations, match ratios, matched regions, and concentration ranges
- 'Result Show' tables: metabolites categorized by three significance levels and metabolites per ROI
- 'dataResult' output file with final categorized results
- Optional Combine Analysis table highlighting metabolites shared between two comparison groups (overline = same trend, underline = opposite trend)

## How to apply

Set up Python >=3.9 with pinned dependencies (pandas 2.2.3, PyQt5 5.15.11, XlsxWriter 3.2.2, openpyxl 3.1.5) and run the ROIAL-NMR GUI. In parameter setting, input the ROIs with '+'/'-' for increase/decrease trends and '*'/'!' for significance levels 1/2, select the sample type (e.g., serum), and set the analyzed region (e.g., 0.5-4.5 ppm). Run identification against the HMDB-derived database (chemical shifts and SRRMs for 891 metabolites), keeping only candidates whose match ratio is >= 50% of their SRRMs within the analyzed ROIs. Apply the built-in filters: significance ratio > 50% of significant (p<0.05) ROIs, FDR-significance tagging when at least one SRRM remains significant after FDR correction, FDR-significant trend override, the six increase/decrease significance categories, and a 5 uM concentration cutoff. For two-group comparisons (e.g., LC vs healthy control), use Combine Analysis to highlight metabolites identified in both groups, then inspect results in the 'Result Show' window and save to the 'dataResult' file.

## Related tools

- **ROIAL-NMR** (Python program with PyQt5 GUI that identifies and categorizes potential metabolites from user-defined proton NMR ROIs using HMDB as reference) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **Python >=3.9** (Runtime environment for ROIAL-NMR) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **pandas 2.2.3** (Pinned dependency of ROIAL-NMR for data handling) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **PyQt5 5.15.11** (Pinned dependency providing the ROIAL-NMR graphical interface) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **XlsxWriter 3.2.2** (Pinned dependency for writing the 'dataResult' spreadsheet output) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **openpyxl 3.1.5** (Pinned dependency for reading/writing Excel inputs and outputs) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **Human Metabolome Database (HMDB)** (Reference platform providing chemical shift values and SRRMs for 891 metabolites used for identification)

## Examples

```
python main.py
```

## Evaluation signals

- On the example lung cancer serum study (18 ROIs, 0.5-4.5 ppm, 36 serum samples), the run identifies exactly 88 potential metabolites, with 66 differentiating LC from healthy controls and 80 distinguishing LC patients with ADRD from those without
- The 'dataResult' output file exists and contains metabolites categorized by the three significance levels plus per-ROI tables
- Every retained metabolite has a match ratio >= 50% of its SRRMs within the analyzed ROIs
- Every retained metabolite passes the 5 uM concentration cutoff for HMDB biofluid values
- Single-population analysis completes in about 5 seconds and multi-group analysis in less than 10 seconds

## Limitations

- Automated identification of ROIs is beyond the scope of the current version; ROIs must be user-defined
- Absolute quantification of compounds is not the primary aim of the program
- ROIAL-NMR was not built to test the accuracy of the user-defined ROIs
- Using user-defined NMR spectral ROIs can lead to variable results obtained by different users
- Automatic synchronization of NMR chemical shift databases is planned but not yet implemented

## Evidence
