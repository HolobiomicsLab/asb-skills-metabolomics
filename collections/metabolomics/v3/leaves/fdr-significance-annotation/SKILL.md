---
name: fdr-significance-annotation
description: Use when ROIs from disease-vs-control NMR spectra (e.g. serum from lung
  cancer vs controls) have been scored for trend ('+'/'-') and significance ('*'/'!')
  and metabolites must be classified into the six ROIAL-NMR categories in the final
  dataResult table.
when_to_use_negative:
- When the goal is absolute quantification of metabolite concentrations — ROIAL-NMR
  explicitly does not target absolute quantification
- When ROIs have not been user-defined or validated — the program was not built to
  test the accuracy of user-defined ROIs, so categorization of garbage ROIs is meaningless
- When peaks outside the analyzed ROI dominate the spectrum — more than 50% of a metabolite's
  peaks beyond the analyzed region are excluded from the match-ratio calculation,
  making categorization unreliable
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: ROIAL-NMR
  role: GUI program that performs the six-category trend/significance categorization,
    Result Show rendering, and dataResult export
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR
- name: pandas
  role: Loads and manipulates the match-ratio and ROI tables during categorization
- name: XlsxWriter
  role: Writes the final categorized tables to the dataResult Excel file
- name: openpyxl
  role: Excel I/O support for saving categorized results
- name: PyQt5
  role: GUI framework hosting the Result Show output window
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/nbm.70131
    title: Automatic Identification of Potential Cellular Metabolites for Untargeted
      NMR Metabolomics
  source_content_hash: sha256:c4d3771099698f75fe4b7d2c7c7b628b5e4fbf3877f9ce57bf4ece3470f45dd8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Leo-Cheng-Lab/ROIAL-NMR
  tools:
  - PyQt5
  - Python
  - ROIAL-NMR
  - XlsxWriter
  - openpyxl
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# fdr-significance-annotation

## Summary

Categorizes candidate metabolites identified by ROIAL-NMR into trend x significance classes (increase/decrease x not significant, significant, FDR-significant), applying match-ratio, significance-ratio, and HMDB concentration filters before rendering the final output table. This turns raw ROI-level intensity statistics into an interpretable disease-relevance table.

## When to use

Use when ROIs from disease-vs-control NMR spectra (e.g. serum from lung cancer vs controls) have been scored for trend ('+'/'-') and significance ('*'/'!') and metabolites must be classified into the six ROIAL-NMR categories in the final dataResult table.

## When NOT to use

- When the goal is absolute quantification of metabolite concentrations — ROIAL-NMR explicitly does not target absolute quantification
- When ROIs have not been user-defined or validated — the program was not built to test the accuracy of user-defined ROIs, so categorization of garbage ROIs is meaningless
- When peaks outside the analyzed ROI dominate the spectrum — more than 50% of a metabolite's peaks beyond the analyzed region are excluded from the match-ratio calculation, making categorization unreliable

## Inputs

- Match-ratio table of candidate metabolites vs SRRMs within the analyzed ROI region (e.g. 0.5-4.5 ppm), from the ROIAL-NMR 'All Metabolites' step
- ROI table with trend markers ('+' increase / '-' decrease) and significance markers ('*' Level 1 / '!' Level 2) plus sample type
- HMDB-derived concentration limits for each candidate metabolite
- Optional user-supplied disease-specific metabolite list (e.g. lung cancer, Alzheimer's disease, or prostate cancer lists shipped with ROIAL-NMR)

## Outputs

- Categorized tables displayed in the ROIAL-NMR 'Result Show' window (metabolites grouped by three significance levels, plus metabolites within each ROI)
- Final categorized metabolite tables written to the 'dataResult' Excel file (via XlsxWriter/openpyxl)

## How to apply

Load the match-ratio table plus the ROI table carrying trend markers ('+' increase / '-' decrease) and significance markers ('*' Level 1 / '!' Level 2) with pandas. First retain only metabolites whose match ratio is >= 50% of their SRRMs within the analyzed 0.5-4.5 ppm region, excluding peaks beyond the analyzed segment from the calculation. Compute each metabolite's significance ratio (fraction of SRRMs with p<0.05); a ratio > 50% marks the metabolite significant, and it becomes FDR-significant if at least one SRRM remains significant after FDR correction (p<0.05). Assign an overall trend by comparing integrated peak intensities between tested and control groups, letting FDR-significant ROI trends override non-significant ones and marking conflicting FDR-significant increase vs. decrease as no trend. Apply the HMDB concentration filter (exclude metabolites with concentration limits below 5 uM; classify the rest as 5-500 uM, >500 uM, or 'not quantified'), optionally tag disease-relevant metabolites against a user-supplied disease list (lung cancer, Alzheimer's disease, or prostate cancer), assign each metabolite to one of the six categories, render them in the Result Show window, and write the final tables to the dataResult Excel file with XlsxWriter/openpyxl.

## Related tools

- **ROIAL-NMR** (GUI program that performs the six-category trend/significance categorization, Result Show rendering, and dataResult export) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR
- **pandas** (Loads and manipulates the match-ratio and ROI tables during categorization)
- **XlsxWriter** (Writes the final categorized tables to the dataResult Excel file)
- **openpyxl** (Excel I/O support for saving categorized results)
- **PyQt5** (GUI framework hosting the Result Show output window)

## Examples

```
python main.py
```

## Evaluation signals

- dataResult file exists and is a valid Excel workbook readable by openpyxl
- Reproduced scale matches the reported results: 88 identified metabolites, 66 differentiating LC from controls, 80 distinguishing LC-ADRD from LC-non-ADRD
- No metabolite with match ratio < 50% of its SRRMs within the analyzed region appears in the final table
- No metabolite with HMDB concentration limits below 5 uM appears in the final table
- Every categorized metabolite falls into exactly one of the six categories (increase/decrease x not significant/significant/FDR-significant), and metabolites with conflicting FDR-significant increase vs. decrease trends are marked as no trend

## Limitations

- Automated identification of ROIs is beyond the scope of the current version of the program
- User-defined ROIs can lead to variable results between users; ROIAL-NMR was not built to test the accuracy of user-defined ROIs
- Absolute quantification of compounds is not the primary aim of the program
- Metabolites failing the 50% match-ratio or sub-5-uM concentration thresholds are silently dropped from the final table

## Evidence
