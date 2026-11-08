---
name: pesticide-metabolite-detection-screening
description: Use when a curated xenobiotic-metabolite database (accurate m/z + RT)
  must be matched against LC-HRMS features from a urine cohort to determine which
  metabolites are detected, their detection-frequency bins, and mean identifications
  per subject.
when_to_use_negative:
- The PMDB or other annotation database has not yet been curated (i.e., entries lack
  accurate m/z and retention time) — run the CMDN annotation/curation workflow first
- The LC-HRMS data have not been converted to a processed feature table (raw profiles
  still need MS-Dial/peak-picking preprocessing)
- The goal is de novo discovery of unknown biotransformation products rather than
  screening a fixed database of known entries
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CMDN
  role: produced the CMDN-annotated PMDB (N = 492) that serves as the screening database
  repo: https://github.com/LeaveMeNotTonight/CMDN
- name: LC-HRMS
  role: acquisition platform whose feature table from the 438 urine samples is screened
    against the PMDB
- name: TraceFinder
  role: supplies the detection criteria used to mark each PMDB metabolite detected
    or not detected per sample
- name: MS-Dial
  role: compatible software used upstream to generate the processed LC-HRMS feature
    table
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:9666b046b41a8391e183bde8fbab987bd248d8e8de03a70f03b81160c09c0d84
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/LeaveMeNotTonight/CMDN
  tools:
  - CMDN
  - LC-HRMS
  - MS-Dial
  - TraceFinder
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# pesticide-metabolite-detection-screening

## Summary

Screen a manually curated pesticide metabolite database (CMDN-annotated PMDB, N = 492 entries with accurate m/z and retention time) against a processed LC-HRMS feature table from a human urine cohort using nDATA/TraceFinder detection criteria, yielding detection-frequency stratification and per-subject exposure summaries.

## When to use

Use when a curated xenobiotic-metabolite database (accurate m/z + RT) must be matched against LC-HRMS features from a urine cohort to determine which metabolites are detected, their detection-frequency bins, and mean identifications per subject.

## When NOT to use

- The PMDB or other annotation database has not yet been curated (i.e., entries lack accurate m/z and retention time) — run the CMDN annotation/curation workflow first
- The LC-HRMS data have not been converted to a processed feature table (raw profiles still need MS-Dial/peak-picking preprocessing)
- The goal is de novo discovery of unknown biotransformation products rather than screening a fixed database of known entries

## Inputs

- CMDN-annotated PMDB, manually curated, N = 492 entries with accurate m/z and retention time (Table S2, sheet 1)
- Processed LC-HRMS feature table from 438 BCP (human urine cohort) samples
- nDATA/TraceFinder detection criteria definition

## Outputs

- pmdb_bcp_screening_results.csv listing detected metabolites with frequency-group labels (high >50%, moderate 10-50%, low <10%)
- Per-subject summary of distinct metabolite identifications and cohort mean (~11 per subject)

## How to apply

Load the curated CMDN-annotated PMDB (Table S2, sheet 1; N = 492 manually curated pesticide metabolites with accurate m/z and retention time) via readxl in R, alongside the processed LC-HRMS feature table from the 438 BCP urine samples. Apply nDATA/TraceFinder detection criteria to match PMDB entries to sample features, marking each metabolite detected or not detected in each of the 438 samples. Classify detected metabolites into frequency groups: high (>50% of samples), moderate (10-50%), and low (<10%). Count distinct metabolite identifications per subject and compute the cohort average (expected ~11 per subject). Write the detected metabolites, frequency-group labels, and per-subject summary to pmdb_bcp_screening_results.csv.

## Related tools

- **CMDN** (produced the CMDN-annotated PMDB (N = 492) that serves as the screening database) — https://github.com/LeaveMeNotTonight/CMDN
- **LC-HRMS** (acquisition platform whose feature table from the 438 urine samples is screened against the PMDB)
- **TraceFinder** (supplies the detection criteria used to mark each PMDB metabolite detected or not detected per sample)
- **MS-Dial** (compatible software used upstream to generate the processed LC-HRMS feature table)

## Examples

```
readxl::read_excel("TableS2_sheet1.xlsx", sheet = 1)  # load curated PMDB for screening against BCP urine feature table
```

## Evaluation signals

- The screen recovers exactly 92 detected pesticide metabolites from the 492-entry PMDB
- Frequency-group assignments partition cleanly into >50%, 10-50%, and <10% bins over the 438 samples
- Mean distinct identifications per subject equals the reported ~11
- Every PMDB entry carries a binary detected/not-detected call in each of the 438 samples (no missing calls)
- Output CSV schema contains metabolite identity, frequency-group label, and per-subject counts

## Limitations

- Detection depends on the established nDATA/TraceFinder matching criteria; metabolites present but below detection thresholds will be missed
- The PMDB is limited to manually curated entries (N = 492) from the 2,886 biotransformed derivatives CMDN annotated, so uncurated metabolites are not screened
- Frequency binning thresholds (>50%, 10-50%, <10%) are fixed and may not suit much smaller or larger cohorts

## Evidence
