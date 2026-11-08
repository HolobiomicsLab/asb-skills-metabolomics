---
name: metabolite-database-interrogation
description: Use when the user supplies one or more proton NMR spectral ROIs (e.g.,
  peaks at 1.31–1.34 and 4.10–4.12 ppm) and needs to enumerate candidate metabolites
  whose reference chemical shifts match those regions.
when_to_use_negative:
- The goal is automated identification of ROIs from spectra — ROIAL-NMR relies on
  user-defined ROIs and does not detect them itself
- Absolute quantification of metabolites is required — the program is not designed
  for absolute quantification
- The metabolite library is not NMR-assignment based (the program was not built to
  validate the accuracy of user-defined ROIs or non-NMR reference data)
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: ROIAL-NMR
  role: Performs per-ROI screening of chemical shift data for 891 HMDB metabolites
    against user-defined ROIs, computing match ratios and matched regions within ~5
    seconds
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: Python >=3.9
  role: Runtime environment for ROIAL-NMR's library interrogation and ROI matching
    logic
- name: pandas 2.2.3
  role: Holds the metabolite library and ROI table as DataFrames for per-ROI chemical
    shift querying
- name: PyQt5 5.15.11
  role: GUI framework exposing the 'All Metabolites' window where abbreviations, match
    ratios, matched regions, and concentration ranges are viewed
- name: Human Metabolome Database (HMDB)
  role: Reference platform supplying chemical shift assignments and concentration
    ranges for the 891-metabolite library
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/nbm.70131
    title: Automatic Identification of Potential Cellular Metabolites for Untargeted
      NMR Metabolomics
  source_content_hash: sha256:558851e85a48a2e2f0bdfa87b7dc9495e6533434fd8ea49be8841ce292201425
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
  tools:
  - Human Metabolome Database (HMDB)
  - PyQt5 5.15.11
  - Python
  - Python >=3.9
  - ROIAL-NMR
  - pandas
  - pandas 2.2.3
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# metabolite-database-interrogation

## Summary

Systematically retrieves metabolites whose NMR chemical shifts fall within user-defined spectral regions-of-interest (ROIs) by screening a reference metabolite library (e.g., 891 HMDB metabolites) rather than searching databases one metabolite at a time.

## When to use

Use when the user supplies one or more proton NMR spectral ROIs (e.g., peaks at 1.31–1.34 and 4.10–4.12 ppm) and needs to enumerate candidate metabolites whose reference chemical shifts match those regions.

## When NOT to use

- The goal is automated identification of ROIs from spectra — ROIAL-NMR relies on user-defined ROIs and does not detect them itself
- Absolute quantification of metabolites is required — the program is not designed for absolute quantification
- The metabolite library is not NMR-assignment based (the program was not built to validate the accuracy of user-defined ROIs or non-NMR reference data)

## Inputs

- HMDB-derived metabolite library (chemical shift values and SRRMs for 891 HMDB metabolites)
- User-specified ROI table (e.g., 18 ROIs spanning the 0.5–4.5 ppm up-field segment) with ppm boundaries
- Optional sample type, trend (+/-), and significance level markers (*, !) per ROI

## Outputs

- all_metabolites_matches.csv — per-ROI matched metabolite table with abbreviation, match ratio, matched regions, and HMDB concentration range
- Matched SRRM sets per metabolite (number of matched SRRMs / total SRRMs) feeding downstream categorization

## How to apply

Load the HMDB-derived metabolite library (chemical shift values and SRRMs for 891 metabolites, produced upstream from HMDB) and the user-specified ROI table (e.g., 18 ROIs spanning the 0.5–4.5 ppm up-field segment) into pandas DataFrames. For each ROI, query the library for every metabolite possessing at least one chemical shift peak within the ROI boundaries, marking the encompassing SRRM as matched whenever a peak falls inside a ROI. Exclude peaks outside the analyzed 0.5–4.5 ppm segment, and drop any metabolite with more than 50% of its peaks beyond the analyzed region before ratio calculation to avoid overrepresenting questionable metabolites. Compute each metabolite's match ratio as matched SRRMs divided by total SRRMs, and attach the matched region boundaries plus HMDB concentration ranges (concentration limits below 5 μM, roughly the NMR detection limit, are excluded downstream). Write the per-ROI matched metabolite table (abbreviation, match ratio, matched regions, concentration range) to all_metabolites_matches.csv; ROIAL-NMR performs this screening in about 5 seconds.

## Related tools

- **ROIAL-NMR** (Performs per-ROI screening of chemical shift data for 891 HMDB metabolites against user-defined ROIs, computing match ratios and matched regions within ~5 seconds) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **Python >=3.9** (Runtime environment for ROIAL-NMR's library interrogation and ROI matching logic)
- **pandas 2.2.3** (Holds the metabolite library and ROI table as DataFrames for per-ROI chemical shift querying)
- **PyQt5 5.15.11** (GUI framework exposing the 'All Metabolites' window where abbreviations, match ratios, matched regions, and concentration ranges are viewed)
- **Human Metabolome Database (HMDB)** (Reference platform supplying chemical shift assignments and concentration ranges for the 891-metabolite library)

## Examples

```
python main.py  # then set ROIs, trends, and significance levels in the GUI; matched metabolites appear in the 'All Metabolites' window and all_metabolites_matches.csv
```

## Evaluation signals

- Lactate control case: peaks at 1.324 ppm (ROI 1.31–1.34) and 4.111 ppm (ROI 4.10–4.12) both match, yielding a match ratio of 2/2
- Each output metabolite row contains non-empty abbreviation, match ratio, matched region boundaries, and HMDB concentration range columns
- Match ratio is computed only after excluding metabolites with more than 50% of peaks beyond the 0.5–4.5 ppm analyzed region
- Only metabolites with match ratio ≥ 50% of SRRMs within the analyzed ROIs are retained for further categorization
- Screening completes within ~5 seconds for the 891-metabolite library (single-population analysis ~5 s, multi-group <10 s)

## Limitations

- Automated identification of ROIs is beyond the scope of the current version of the program — ROIs must be user-defined
- ROIAL-NMR was not built to test the accuracy of the user-defined ROIs; results depend on user consistency, and different users can obtain variable results
- Absolute quantification of compounds is not the primary aim of the program
- Automatic synchronization of NMR chemical shift databases is planned but not yet implemented, requiring manual library updates

## Evidence
