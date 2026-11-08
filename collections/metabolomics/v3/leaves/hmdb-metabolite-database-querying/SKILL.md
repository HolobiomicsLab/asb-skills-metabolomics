---
name: hmdb-metabolite-database-querying
description: Use when you have user-defined proton NMR spectral ROIs (e.g., 0.5–4.5
  ppm biofluid regions) and need to identify candidate metabolites by matching HMDB
  chemical shift peaks and SRRMs, avoiding over-identification in crowded spectra.
when_to_use_negative:
- When the goal is absolute quantification of compounds — the article states 'absolute
  quantification of the compounds is not the primary aim of the program'.
- When ROIs need to be identified automatically from spectra — 'Automated identification
  of ROIs is beyond the scope of the current version of the program'.
- When validating the accuracy of user-defined ROIs — 'ROIAL-NMR was not built to
  test the accuracy of the user-defined ROIs'.
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3174
tools:
- name: ROIAL-NMR
  role: Implements the SRRM-to-ROI matching and the >=50% match-ratio filter that
    suppresses metabolite over-identification across multiple ROIs
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: Python
  role: Runtime for ROIAL-NMR (Python >=3.9) and pandas-based loading/processing of
    the HMDB peak/SRRM database
- name: pandas
  role: Loads and manipulates the HMDB-derived chemical shift/SRRM database and the
    per-ROI candidate metabolite tables
- name: Human Metabolome Database (HMDB)
  role: Reference platform supplying chemical shift values and metabolite-specific
    SRRMs (891 metabolites in the published demonstration)
- name: XlsxWriter
  role: Writes the scored metabolite/result tables to Excel files
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/nbm.70131
    title: Automatic Identification of Potential Cellular Metabolites for Untargeted
      NMR Metabolomics
  source_content_hash: sha256:40f3a11ecd3f919e1979e7d0d6ea7450b4d96071bc14ccb15fde567cec2acd81
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
  tools:
  - Human Metabolome Database (HMDB)
  - Python
  - ROIAL-NMR
  - XlsxWriter
  - pandas
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# hmdb-metabolite-database-querying

## Summary

Query the Human Metabolome Database (HMDB) for metabolite-specific chemical shift peaks and SRRMs (metabolite-specific resonance ranges), then score candidates against user-defined NMR ROIs using a match-ratio rule (retain metabolites with ≥50% of SRRMs matched to analyzed ROIs) so metabolites present in multiple spectral regions are not over-identified.

## When to use

Use when you have user-defined proton NMR spectral ROIs (e.g., 0.5–4.5 ppm biofluid regions) and need to identify candidate metabolites by matching HMDB chemical shift peaks and SRRMs, avoiding over-identification in crowded spectra.

## When NOT to use

- When the goal is absolute quantification of compounds — the article states 'absolute quantification of the compounds is not the primary aim of the program'.
- When ROIs need to be identified automatically from spectra — 'Automated identification of ROIs is beyond the scope of the current version of the program'.
- When validating the accuracy of user-defined ROIs — 'ROIAL-NMR was not built to test the accuracy of the user-defined ROIs'.

## Inputs

- Per-ROI candidate metabolite table (output of upstream ROI peak-matching step, task_003)
- HMDB-derived reference database of chemical shift peaks and metabolite-specific SRRMs (891 HMDB metabolites in the published demonstration)
- User-defined ROI boundaries (ppm ranges), e.g., lactate ROIs [1.31-1.34] and [4.10-4.12] ppm
- Analyzed spectral region definition (e.g., 0.5-4.5 ppm for biofluids)

## Outputs

- Scored metabolite table with match ratios and matched regions (tabular file, e.g., Excel via XlsxWriter/openpyxl) for the categorization step
- Metabolites retained at match ratio >= 0.5, viewable in the 'All Metabolites' window with abbreviations, match ratios, matched regions, and concentration ranges

## How to apply

Load the per-ROI candidate metabolite table and the HMDB-derived database of chemical shift peaks and SRRMs with pandas (Python >=3.9, pandas 2.2.3). Pre-filter each metabolite: exclude peaks/SRRMs outside the analyzed spectral region (e.g., 0.5–4.5 ppm for biofluids), and if more than 50% of a metabolite's peaks lie beyond the analyzed region, exclude those peaks from the match-ratio calculation. For each remaining metabolite, apply the peak-inclusion rule: a SRRM counts as matched to a ROI if at least one of its chemical shift peaks falls within the experimentally determined ROI boundaries. Compute the match ratio as (number of SRRMs matched to ROIs) / (total SRRMs for that metabolite) and keep only metabolites with a match ratio >= 0.5 for downstream categorization. Note both chemical shift values and metabolite-specific SRRMs are required for accurate identification, and the same mechanism works for any metabolite database containing NMR assignments, not just HMDB.

## Related tools

- **ROIAL-NMR** (Implements the SRRM-to-ROI matching and the >=50% match-ratio filter that suppresses metabolite over-identification across multiple ROIs) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **Python** (Runtime for ROIAL-NMR (Python >=3.9) and pandas-based loading/processing of the HMDB peak/SRRM database)
- **pandas** (Loads and manipulates the HMDB-derived chemical shift/SRRM database and the per-ROI candidate metabolite tables)
- **Human Metabolome Database (HMDB)** (Reference platform supplying chemical shift values and metabolite-specific SRRMs (891 metabolites in the published demonstration))
- **XlsxWriter** (Writes the scored metabolite/result tables to Excel files)

## Examples

```
git clone https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git && cd ROIAL-NMR && python main.py
```

## Evaluation signals

- Reproduce the paper's lactate worked example: both HMDB SRRMs ([1.30-1.33] and [4.08-4.13] ppm) match against ROIs [1.31-1.34] and [4.10-4.12] ppm, giving a match ratio of 2/2 = 1.0.
- Every metabolite retained in the output table has match ratio >= 0.5; any metabolite with fewer than half of its SRRMs inside the analyzed ROIs is dropped.
- Peaks/SRRMs outside the analyzed region (e.g., beyond 0.5-4.5 ppm) never contribute to the match ratio, and metabolites with >50% of peaks beyond the region have those peaks excluded from the calculation.
- Output table schema includes per-metabolite match ratios, matched regions, abbreviations, and concentration ranges, and is saved as a tabular file for the categorization step.
- Chemical shift coverage check: the database used spans 891 HMDB metabolites for the published 5-second ROI identification run.

## Limitations

- The match-ratio filtering depends on user-defined ROIs; results can vary between users due to inconsistent ROI creation ('using user-defined NMR spectral ROIs can lead to variable results obtained by different users').
- Automatic synchronization of NMR chemical shift databases (e.g., HMDB updates) is planned but not yet implemented.
- The mechanism requires databases that contain both chemical shift values and metabolite-specific SRRMs; both are required for accurate metabolite identification.
- Results depend on concentration limits — metabolites with concentration limits below 5 μM (the lower limit of detection by most NMR instruments) are excluded in downstream filtering.

## Evidence
