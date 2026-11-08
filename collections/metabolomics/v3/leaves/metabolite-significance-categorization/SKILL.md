---
name: metabolite-significance-categorization
description: Use when proton NMR spectra of biological samples (serum, saliva, urine,
  CSF, tissues) have user-defined ROIs with trend and significance annotations and
  you need metabolites classified by trend and statistical significance for disease-vs-control
  comparison.
when_to_use_negative:
- You need automated ROI identification from raw spectra — automated identification
  of ROIs is beyond the current scope of ROIAL-NMR and ROIs must be user-defined
- You need absolute quantification of compounds — this is not the primary aim of the
  program
- You need to validate the accuracy of user-defined ROIs — the program was not built
  to test ROI accuracy, and user-defined ROIs can yield variable results between users
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0593
tools:
- name: ROIAL-NMR
  role: Python/PyQt5 program that performs the ROI-based metabolite identification,
    significance categorization, and Combine Analysis between groups
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: PyQt5 5.15.11
  role: GUI framework driving the parameter-setting, All Metabolites, Result Show,
    and Combine Analysis windows
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: pandas 2.2.3
  role: Pinned dependency for tabular data handling of ROI tables and metabolite results
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: XlsxWriter 3.2.2
  role: Pinned dependency for exporting the dataResult file and Combine Analysis tables
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: openpyxl 3.1.5
  role: Pinned dependency for reading/writing spreadsheet inputs and outputs
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: Human Metabolome Database (HMDB)
  role: Reference platform supplying chemical shift values and SRRMs (891 serum metabolites)
    and biofluid concentration values used for the 5 uM cutoff
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/nbm.70131
    title: Automatic Identification of Potential Cellular Metabolites for Untargeted
      NMR Metabolomics
  source_content_hash: sha256:a159dcd431caae94c55b1a5e87dc11a24a953733c5fa36493fc106b9b8d3ebc9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
  tools:
  - Human Metabolome Database (HMDB)
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# metabolite-significance-categorization

## Summary

Categorize candidate metabolites identified from user-defined proton NMR ROIs into six trend/significance categories (increase/decrease crossed with not significant, significant, FDR-significant) using ROIAL-NMR, and optionally tag disease relevance. This turns ROI-level statistical signals into a per-metabolite categorized output table.

## When to use

Use when proton NMR spectra of biological samples (serum, saliva, urine, CSF, tissues) have user-defined ROIs with trend and significance annotations and you need metabolites classified by trend and statistical significance for disease-vs-control comparison.

## When NOT to use

- You need automated ROI identification from raw spectra — automated identification of ROIs is beyond the current scope of ROIAL-NMR and ROIs must be user-defined
- You need absolute quantification of compounds — this is not the primary aim of the program
- You need to validate the accuracy of user-defined ROIs — the program was not built to test ROI accuracy, and user-defined ROIs can yield variable results between users

## Inputs

- User-defined ROI table for proton NMR spectra with trend ('+'/'-') and significance level ('*'/'!') annotations
- Sample type selection (e.g., serum) and analyzed spectral region (e.g., 0.5-4.5 ppm)
- HMDB-derived metabolite database with chemical shift values and SRRMs (891 metabolites for serum)
- Statistical ROI-level results (p<0.05 and FDR-corrected significance) from the study comparisons
- Optional user-prepared disease-specific metabolite list (e.g., ADRD-relevant metabolites from HMDB)

## Outputs

- 'Result Show' first table: metabolites categorized by three significance levels
- 'Result Show' second table: metabolites within each ROI
- 'dataResult' output file with final categorized metabolites and per-ROI tables
- 'All Metabolites' window listing metabolites with abbreviations, match ratios, matched regions, and concentration ranges
- Optional Combine Analysis table highlighting metabolites identified in both comparison groups (overline = same trend, underline = opposite trend)

## How to apply

In the ROIAL-NMR PyQt5 interface, input the ROI table encoding trends with '+' (increase) and '-' (decrease) and significance levels with '*' (level 1) and '!' (level 2), select the sample type, and set the analyzed region (e.g., 0.5-4.5 ppm for serum). After candidate identification against the HMDB-derived database (only metabolites with match ratio >= 50% of their SRRMs within the analyzed ROIs are kept), apply the built-in filters: significance ratio > 50% (proportion of p<0.05 ROIs among all SRRMs), FDR-significance tagging when at least one SRRM remains significant after FDR correction (p<0.05), trend override by FDR-significant ROIs, and a 5 uM concentration cutoff (lower NMR detection limit) on HMDB biofluid values. Metabolites fall into six categories: 1) increase, not significant; 2) increase, significant; 3) increase, FDR-significant; 4) decrease, not significant; 5) decrease, significant; 6) decrease, FDR-significant. Optionally run Combine Analysis on two comparison groups (e.g., LC vs healthy control and LC-ADRD vs LC-non-ADRD) where overline marks shared trends and underline marks opposite trends, then save results to the 'dataResult' file.

## Related tools

- **ROIAL-NMR** (Python/PyQt5 program that performs the ROI-based metabolite identification, significance categorization, and Combine Analysis between groups) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **PyQt5 5.15.11** (GUI framework driving the parameter-setting, All Metabolites, Result Show, and Combine Analysis windows) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **pandas 2.2.3** (Pinned dependency for tabular data handling of ROI tables and metabolite results) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **XlsxWriter 3.2.2** (Pinned dependency for exporting the dataResult file and Combine Analysis tables) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **openpyxl 3.1.5** (Pinned dependency for reading/writing spreadsheet inputs and outputs) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **Human Metabolome Database (HMDB)** (Reference platform supplying chemical shift values and SRRMs (891 serum metabolites) and biofluid concentration values used for the 5 uM cutoff)

## Examples

```
python main.py
```

## Evaluation signals

- dataResult file exists and the first table groups metabolites by the three significance levels while the second table lists metabolites within each ROI
- Every retained metabolite has a match ratio >= 50% of its SRRMs within the analyzed ROIs
- Every metabolite falls into exactly one of the six defined categories (increase/decrease x not significant / significant / FDR-significant)
- No metabolite with HMDB biofluid concentration below 5 uM remains in the final list
- Reproducing the example serum run yields 88 potential metabolites, with 66 differentiating LC from healthy controls and 80 distinguishing LC-ADRD from LC-non-ADRD, completing in ~5 s (single population) or <10 s (multi-group)

## Limitations

- Automated identification of ROIs is beyond the scope of the current version of the program
- Absolute quantification of the compounds is not the primary aim of the program
- ROIAL-NMR was not built to test the accuracy of the user-defined ROIs; results depend on user consistency, and different users can obtain variable results
- Users can customize and adjust filter parameters, so defaults (e.g., 50% match ratio, 50% significance ratio, 5 uM cutoff) may differ across studies
- Future automatic synchronization of NMR chemical shift databases (HMDB, BMRB) is planned but not yet implemented

## Evidence

- [readme] Use “+” for increase and “-” for decrease. Use “*” for Significance Level 1 and “!” for Significance Level 2.: [evidence span withheld — anchor status: not_found]
