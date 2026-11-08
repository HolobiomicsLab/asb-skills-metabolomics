---
name: nmr-metabolite-trend-categorization
description: Use when candidate metabolites from ROIs in 1H NMR spectra (e.g. a 0.5-4.5
  ppm serum region) must be filtered by match ratio, significance, HMDB concentration,
  and disease relevance, then sorted into an output table for a disease-vs-control
  comparison.
when_to_use_negative:
- The user needs automated ROI identification — this is beyond the current scope of
  ROIAL-NMR, which requires user-defined ROIs
- Absolute quantification of metabolites is the goal — the program explicitly does
  not target absolute quantification
- Validating the accuracy of user-defined ROIs is the objective — ROIAL-NMR was not
  built to test ROI accuracy
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0593
tools:
- name: ROIAL-NMR
  role: Program whose filtering and six-category categorization logic this skill implements;
    performs the match-ratio, significance-ratio, FDR, trend, and concentration filters
    and renders the final tables
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: pandas
  role: Load and manipulate the match-ratio and ROI tables during filtering and categorization
- name: XlsxWriter
  role: Write the final categorized tables to the dataResult Excel file
- name: openpyxl
  role: Excel output support for the dataResult file
- name: PyQt5
  role: GUI framework hosting the 'Result Show' window where the categorized tables
    are displayed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/nbm.70131
    title: Automatic Identification of Potential Cellular Metabolites for Untargeted
      NMR Metabolomics
  source_content_hash: sha256:3713514e0dcd3f3cd9405a3d835ced3e1a6113256dc9802464ac8c9ef9201ffb
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# nmr-metabolite-trend-categorization

## Summary

A filtering and categorization step of the ROIAL-NMR workflow that retains candidate metabolites passing a >=50% match ratio and a >=5 uM HMDB concentration cutoff, then assigns each metabolite to one of six trend-by-significance categories (increase/decrease x not significant/significant/FDR-significant) and tags disease-relevant metabolites before writing the final tables.

## When to use

Use when candidate metabolites from ROIs in 1H NMR spectra (e.g. a 0.5-4.5 ppm serum region) must be filtered by match ratio, significance, HMDB concentration, and disease relevance, then sorted into an output table for a disease-vs-control comparison.

## When NOT to use

- The user needs automated ROI identification — this is beyond the current scope of ROIAL-NMR, which requires user-defined ROIs
- Absolute quantification of metabolites is the goal — the program explicitly does not target absolute quantification
- Validating the accuracy of user-defined ROIs is the objective — ROIAL-NMR was not built to test ROI accuracy

## Inputs

- Match-ratio table from the prior ROI-matching step (metabolite vs. SRRM match ratios within the analyzed 0.5-4.5 ppm region)
- ROI table with trend markers ('+' increase / '-' decrease) and significance markers ('*' Level 1 / '!' Level 2)
- Statistical significance per ROI (p<0.05 and FDR-corrected p values)
- HMDB concentration range information for each candidate metabolite
- User-supplied disease-specific metabolite list (e.g. lung cancer, Alzheimer's disease, or prostate cancer lists shipped with ROIAL-NMR)

## Outputs

- Categorized metabolite tables with six categories (increase/decrease x not significant/significant/FDR-significant) rendered in the 'Result Show' GUI output
- dataResult Excel file (written with XlsxWriter/openpyxl) containing the final categorized tables
- Disease-relevant metabolite tags applied to qualifying entries

## How to apply

Load the match-ratio table and the ROI table carrying trend markers ('+' increase / '-' decrease) and significance markers ('*' Level 1 / '!' Level 2) with pandas. Keep only metabolites whose match ratio is >= 50% of their SRRMs within the analyzed region, excluding peaks beyond the analyzed segment from the calculation; compute the significance ratio (statistically significant ROIs at p<0.05 among SRRMs, >50% marks the metabolite significant) and flag FDR-significance when at least one SRRM remains significant after FDR correction. Assign each metabolite an overall trend by comparing integrated peak intensities between tested and control groups, letting FDR-significant ROI trends override non-significant ones and marking metabolites with conflicting FDR-significant increase vs. decrease as no trend. Apply the HMDB concentration filter excluding metabolites with concentration limits below 5 uM, classify the rest as 5-500 uM, >500 uM, or 'not quantified', tag disease-relevant metabolites against a user-supplied disease list (lung cancer, Alzheimer's disease, or prostate cancer lists shipped with ROIAL-NMR), and write the six-category tables via XlsxWriter/openpyxl to the dataResult Excel file.

## Related tools

- **ROIAL-NMR** (Program whose filtering and six-category categorization logic this skill implements; performs the match-ratio, significance-ratio, FDR, trend, and concentration filters and renders the final tables) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **pandas** (Load and manipulate the match-ratio and ROI tables during filtering and categorization)
- **XlsxWriter** (Write the final categorized tables to the dataResult Excel file)
- **openpyxl** (Excel output support for the dataResult file)
- **PyQt5** (GUI framework hosting the 'Result Show' window where the categorized tables are displayed)

## Examples

```
python main.py  # then: Parameters setting -> input ROIs/trends ('+','-'; '*','!'), sample type, analyzed region -> All Metabolites -> Result Show -> dataResult file
```

## Evaluation signals

- dataResult file exists in Excel (.xlsx) format after the step completes
- Categorized output reproduces the reported scale: 88 identified metabolites, with 66 differentiating LC from controls and 80 distinguishing LC-ADRD from LC-non-ADRD
- No metabolite failing the >=50% match-ratio threshold appears in the final table
- No metabolite with HMDB concentration limits below 5 uM appears in the final table
- Every retained metabolite is assigned to exactly one of the six categories (increase/decrease x not significant/significant/FDR-significant), with conflicting FDR-significant increase vs. decrease marked as no trend

## Limitations

- Results depend on user consistency in creating ROIs; variability between users is acknowledged
- Automated identification of ROIs is beyond the scope of the current version of the program
- Absolute quantification of compounds is not the primary aim of the program
- Automatic synchronization of NMR chemical shift databases is planned but not yet implemented
- Filter parameters (match ratio, significance ratio) are customizable, so default thresholds may differ between studies

## Evidence
