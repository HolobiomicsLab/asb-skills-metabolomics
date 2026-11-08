---
name: hmdb-spectral-database-matching
description: Use when untargeted 1H NMR spectra have user-defined ROIs (e.g. 0.5-4.5
  ppm curve-fitted peaks) and metabolites must be identified by matching HMDB chemical
  shifts and SRRMs within those ROIs rather than by whole-spectrum database search.
when_to_use_negative:
- You need automated ROI detection — automated identification of ROIs is beyond the
  scope of the current version; ROIs must be supplied by the user.
- You need absolute quantification — absolute quantification of compounds is not the
  primary aim of the program.
- You want to validate or test the accuracy of user-defined ROIs — ROIAL-NMR was not
  built for that purpose, and user variability in ROI creation can yield variable
  results.
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ROIAL-NMR
  role: 'Performs the matching: intersects user-defined ROIs with HMDB chemical shifts
    and SRRMs, computes match ratios, applies significance and 5 uM concentration
    filters, and writes the ''dataResult'' file'
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: PyQt5
  role: Provides the GUI (parameter setting, 'All Metabolites', and 'Result Show'
    windows) used to drive the matching workflow (5.15.11)
- name: pandas
  role: Pinned dependency (2.2.3) handling ROI and metabolite tables during candidate
    matching and categorization
- name: XlsxWriter
  role: Writes the categorized 'dataResult' results workbook (3.2.2)
- name: openpyxl
  role: Excel input/output for ROI templates and result files (3.1.5)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/nbm.70131
    title: Automatic Identification of Potential Cellular Metabolites for Untargeted
      NMR Metabolomics
  source_content_hash: sha256:a10d9702197ee9c965612f55cae427966ea1d76a6c05b01e93e6c7789fd2d18e
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
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# hmdb-spectral-database-matching

## Summary

ROI-based metabolite identification for untargeted 1H NMR (ROIAL-NMR): candidates are proposed by matching user-defined spectral regions-of-interest against HMDB chemical shift values and metabolite-specific SRRMs for 891 metabolites, then filtered by match ratio, significance, and 5 uM concentration cutoffs.

## When to use

Use when untargeted 1H NMR spectra have user-defined ROIs (e.g. 0.5-4.5 ppm curve-fitted peaks) and metabolites must be identified by matching HMDB chemical shifts and SRRMs within those ROIs rather than by whole-spectrum database search.

## When NOT to use

- You need automated ROI detection — automated identification of ROIs is beyond the scope of the current version; ROIs must be supplied by the user.
- You need absolute quantification — absolute quantification of compounds is not the primary aim of the program.
- You want to validate or test the accuracy of user-defined ROIs — ROIAL-NMR was not built for that purpose, and user variability in ROI creation can yield variable results.

## Inputs

- User-defined ROI table of curve-fitted proton NMR ROIs (e.g. 18 ROIs spanning 0.5-4.5 ppm) with trends encoded '+' (increase) / '-' (decrease) and significance levels '*' (level 1) / '!' (level 2)
- HMDB-derived reference database of chemical shift values plus metabolite-specific SRRMs for 891 metabolites
- Sample type selection (human serum, saliva, sweat, urine, CSF, or tissue) and analyzed spectral region
- Optional user-prepared disease-specific metabolite list for tagging disease-relevant metabolites
- Grouped proton NMR spectral data for comparison populations (e.g. LC vs healthy controls; LC-ADRD vs LC-non-ADRD)

## Outputs

- 'All Metabolites' table listing identified metabolites with abbreviations, match ratios, matched regions, and concentration ranges
- 'dataResult' file containing metabolites categorized by three significance levels plus a per-ROI metabolite table
- Optional Combine Analysis table flagging metabolites identified in both groups (overline = same trends, underline = opposite trends)

## How to apply

In ROIAL-NMR (Python >=3.9 with pandas 2.2.3, PyQt5 5.15.11, XlsxWriter 3.2.2, openpyxl 3.1.5), select the sample type, input the analyzed region (e.g. 0.5-4.5 ppm for serum), and encode each ROI's trend with '+'/'-' and significance level with '*' (level 1) or '!' (level 2). Run matching against the HMDB-derived database of chemical shift values plus SRRMs for 891 metabolites by taking the intersection of ROI specifications, metabolic NMR information, and sample-specific information, so any metabolite with chemical shifts lying within the determined ROIs becomes a candidate. Keep only candidates whose match ratio is >= 50% of their SRRMs within the analyzed ROIs; when more than 50% of a potential metabolite's peaks lie beyond the analyzed region, those peaks are excluded from the match-ratio calculation. Categorize survivors into the six increase/decrease classes using the significance ratio (>50% of SRRMs at p<0.05), FDR significance (at least one SRRM significant after FDR correction, p<0.05) with trend override in FDR-significant ROIs, and exclude metabolites with HMDB biofluid concentration limits below 5 uM. Review the 'All Metabolites' window (abbreviations, match ratios, matched regions, concentration ranges), save categorized results to the 'dataResult' file, and optionally run Combine Analysis to flag metabolites shared by two group comparisons; ROI-level matching with match-ratio scoring avoids the over-identification that single-peak database searches cause.

## Related tools

- **ROIAL-NMR** (Performs the matching: intersects user-defined ROIs with HMDB chemical shifts and SRRMs, computes match ratios, applies significance and 5 uM concentration filters, and writes the 'dataResult' file) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **PyQt5** (Provides the GUI (parameter setting, 'All Metabolites', and 'Result Show' windows) used to drive the matching workflow (5.15.11))
- **pandas** (Pinned dependency (2.2.3) handling ROI and metabolite tables during candidate matching and categorization)
- **XlsxWriter** (Writes the categorized 'dataResult' results workbook (3.2.2))
- **openpyxl** (Excel input/output for ROI templates and result files (3.1.5))

## Examples

```
python main.py
```

## Evaluation signals

- The 'dataResult' file exists and contains a first table of metabolites categorized by three significance levels plus a second table of metabolites within each ROI.
- Every retained metabolite has a match ratio >= 50% of its SRRMs within the analyzed ROIs, and peaks beyond the analyzed region (>50% of a metabolite's peaks) were excluded from the match-ratio calculation.
- No reported metabolite has an HMDB biofluid concentration limit below 5 uM.
- On the example serum study (18 ROIs, 0.5-4.5 ppm), the run identifies 88 potential metabolites — 66 differentiating LC from healthy controls and 80 distinguishing LC-ADRD from LC-non-ADRD.
- Single-population analysis completes in about 5 seconds and multi-group analysis in less than 10 seconds.

## Limitations

- Automated identification of ROIs is beyond the scope of the current version, and results depend on user consistency in creating ROIs — different users can obtain variable results.
- Absolute quantification of compounds is not the primary aim of the program.
- The program was not built to test the accuracy of user-defined ROIs; automatic synchronization of NMR chemical shift databases (HMDB, BMRB) is planned but not yet implemented, so database updates are manual.

## Evidence
