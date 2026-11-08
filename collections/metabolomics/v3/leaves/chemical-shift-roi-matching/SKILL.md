---
name: chemical-shift-roi-matching
description: Use when user-defined 1H NMR ROIs (e.g., 18 ROIs over 0.5-4.5 ppm) must
  be screened against a reference metabolite library to list every metabolite with
  at least one chemical shift peak inside each ROI.
when_to_use_negative:
- Automated ROI identification or ROI accuracy checking is needed — it is beyond the
  scope of the current ROIAL-NMR version; ROIs must be user-defined and the program
  was not built to test their accuracy.
- Absolute quantification of compounds is the goal — the program's primary aim is
  identification and categorization of potential metabolites, not quantification.
- Only chemical shift values are available without metabolite-specific SRRMs — the
  paper found both are required for accurate metabolite identification.
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: ROIAL-NMR
  role: Python program that performs the per-ROI screening of the HMDB chemical-shift/SRRM
    library and computes the cross-ROI match ratios
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: Python
  role: '>=3.9 runtime in which the peak-in-ROI queries and match-ratio computation
    are implemented'
- name: pandas
  role: 2.2.3 DataFrames used to load the HMDB-derived metabolite library and the
    user ROI table for the matching queries
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1002/nbm.70131
    title: Automatic Identification of Potential Cellular Metabolites for Untargeted
      NMR Metabolomics
  source_content_hash: sha256:6c493902d5ff299e824d4d5c95b0e122ba6210d9d4b2eb6c1aa694e27b9988ac
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
  tools:
  - Python
  - ROIAL-NMR
  - pandas
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# chemical-shift-roi-matching

## Summary

Screens a reference metabolite NMR library (chemical shift values and SRRMs for 891 HMDB metabolites) against user-defined proton NMR spectral regions-of-interest, marking each SRRM with a peak inside a ROI and scoring a match ratio (matched/total SRRMs, kept if >=0.5) to shortlist candidate metabolites without over-identification.

## When to use

Use when user-defined 1H NMR ROIs (e.g., 18 ROIs over 0.5-4.5 ppm) must be screened against a reference metabolite library to list every metabolite with at least one chemical shift peak inside each ROI.

## When NOT to use

- Automated ROI identification or ROI accuracy checking is needed — it is beyond the scope of the current ROIAL-NMR version; ROIs must be user-defined and the program was not built to test their accuracy.
- Absolute quantification of compounds is the goal — the program's primary aim is identification and categorization of potential metabolites, not quantification.
- Only chemical shift values are available without metabolite-specific SRRMs — the paper found both are required for accurate metabolite identification.

## Inputs

- HMDB-derived metabolite library of chemical shift values and SRRMs for 891 metabolites (pandas DataFrame)
- User-specified ROI table (e.g., 18 ROIs spanning the 0.5-4.5 ppm up-field segment)
- Analyzed spectral region definition (0.5-4.5 ppm for biofluids)

## Outputs

- all_metabolites_matches.csv — per-ROI matched metabolite table (abbreviation, match ratio, matched regions, HMDB concentration range)
- Per-metabolite SRRM match annotations (which SRRMs matched which ROIs) feeding the >=0.5 match-ratio filter

## How to apply

Load the HMDB-derived library of chemical shift values and metabolite-specific SRRMs for 891 metabolites and the user-specified ROI table (e.g., 18 ROIs spanning the 0.5-4.5 ppm up-field segment) into pandas 2.2.3 DataFrames. For each ROI, query the library for all metabolites possessing at least one chemical shift peak within the ROI boundaries and mark the encompassing SRRM as matched — a SRRM counts as matched if any of its peaks (e.g., lactate peaks at 1.324 and 4.111 ppm) falls inside the ROI. Exclude peaks outside the analyzed 0.5-4.5 ppm segment, and if more than 50% of a metabolite's peaks lie beyond the analyzed region, drop those peaks from the match-ratio calculation to avoid overrepresentation of questionable metabolites. Compute the match ratio as (number of matched SRRMs)/(total SRRMs), keep only metabolites with ratio >= 0.5 (>=50%) for downstream categorization, and attach matched region boundaries plus HMDB concentration ranges. Write the per-ROI matched metabolite table (abbreviation, match ratio, matched regions, concentration range) to all_metabolites_matches.csv; the pass over 891 metabolites completes within ~5 seconds.

## Related tools

- **ROIAL-NMR** (Python program that performs the per-ROI screening of the HMDB chemical-shift/SRRM library and computes the cross-ROI match ratios) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **Python** (>=3.9 runtime in which the peak-in-ROI queries and match-ratio computation are implemented)
- **pandas** (2.2.3 DataFrames used to load the HMDB-derived metabolite library and the user ROI table for the matching queries)

## Examples

```
python main.py
```

## Evaluation signals

- Lactate control case reproduces the paper result: peaks at 1.324 ppm (ROI 1.31-1.34) and 4.111 ppm (ROI 4.10-4.12) both match, yielding a match ratio of 2/2 = 1.0.
- Every metabolite carried forward has match ratio >= 0.5; metabolites with >50% of peaks beyond the 0.5-4.5 ppm analyzed region have those peaks excluded before ratio calculation.
- all_metabolites_matches.csv contains the expected columns: abbreviation, match ratio, matched regions, and concentration range.
- All matched regions in the output fall entirely within the analyzed 0.5-4.5 ppm segment.
- A full screening pass over the 891-metabolite HMDB library completes within ~5 seconds.

## Limitations

- Results depend on user consistency in creating ROIs; user-defined ROIs can lead to variable results obtained by different users.
- Automated identification of ROIs and absolute quantification of compounds are outside the program's current scope.
- Automatic synchronization of NMR chemical shift databases (e.g., periodic HMDB updates) is planned but not yet implemented.
- Downstream categorization excludes metabolites with concentration limits below 5 uM (the lower limit of detection by most NMR instruments).

## Evidence
