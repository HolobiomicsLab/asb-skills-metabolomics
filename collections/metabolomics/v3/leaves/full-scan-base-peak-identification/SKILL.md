---
name: full-scan-base-peak-identification
description: Use when quantifying how many RegFilter-scheduled precursor-list ions
  that DDA failed to fragment were nevertheless the highest-intensity (base peak)
  ions in full-scan MS1, from MZmine feature tables.
when_to_use_negative:
- Input is only DDA MS/MS data with no full-scan MS1 MZmine feature table to query
  peak areas from
- The ion set has already been matched to DDA-fragmented precursors — base-peak analysis
  here applies only to list-only ions lacking DDA MS2
- Features were not preprocessed with MZmine using matching Join aligner tolerances
  (m/z 0.001 Da or 15 ppm, RT 0.2 min), so cross-mode alignment is not valid
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine
  role: Preprocesses full-scan and MS/MS data separately and exports the .csv feature
    tables (row ID, row m/z, row retention time, peak area) used to identify base
    peaks and align SPL vs DDA features
- name: RegFilter
  role: Produces the filtered scheduled precursor lists whose list-only, unfragmented
    ions are tested for base-peak status
  repo: https://github.com/computational-chemical-biology/regression_filter
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:37a0d0d8bcbcd1805b4d4bceac16ec4af731dc5173012486ca514cac842f510e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/computational-chemical-biology/regression_filter
  repo_url: https://github.com/computational-chemical-biology/regression_filter
  tools:
  - MZmine
  - RegFilter
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# full-scan-base-peak-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Determines whether RegFilter list-only ions (present in full-scan MS1 of a DDA run but lacking DDA MS2 fragmentation) were full-scan base peaks, quantifying the intensity of signals missed by DDA per strain (BRA006, BRA010, BRA177).

## When to use

Use when quantifying how many RegFilter-scheduled precursor-list ions that DDA failed to fragment were nevertheless the highest-intensity (base peak) ions in full-scan MS1, from MZmine feature tables.

## When NOT to use

- Input is only DDA MS/MS data with no full-scan MS1 MZmine feature table to query peak areas from
- The ion set has already been matched to DDA-fragmented precursors — base-peak analysis here applies only to list-only ions lacking DDA MS2
- Features were not preprocessed with MZmine using matching Join aligner tolerances (m/z 0.001 Da or 15 ppm, RT 0.2 min), so cross-mode alignment is not valid

## Inputs

- MZmine-preprocessed full-scan (MS1) feature table in .csv format with columns row ID, row m/z, row retention time, peak area
- MZmine DDA MS/MS feature table per strain (BRA006, BRA010, BRA177) listing fragmented precursor ions
- RegFilter-filtered scheduled precursor list (SPL) per strain (135 features BRA006, 219 BRA010, 150 BRA177)

## Outputs

- Per-strain list of list-only ions flagged as full-scan base peaks (59 BRA006, 84 BRA010, 73 BRA177 candidates)
- list_only_ion_fractions.csv with per-strain base-peak fractions among list-only ions

## How to apply

Load the MZmine-preprocessed full-scan feature tables (.csv with row ID, row m/z, row retention time, peak area) and the DDA MS/MS feature tables for strains BRA006, BRA010, and BRA177, plus the RegFilter-filtered features (135, 219, and 150 features respectively) defining the scheduled precursor lists (SPL). Match SPL precursors against the DDA fragmented precursor set using MZmine Join aligner tolerances (m/z tolerance 0.001 Da or 15 ppm, retention time tolerance 0.2 min) to recover common features (76 BRA006, 135 BRA010, 77 BRA177); the remainder are list-only ions (59, 84, 73). For each list-only ion, query its full-scan peak area in the MZmine table and flag it if it was the base peak (highest-intensity ion) at its retention point. Compute the per-strain fraction of list-only ions that were base peaks but not fragmented by DDA, writing results to list_only_ion_fractions.csv. The rationale is that DDA's stochastic nature can skip intense ions, so base-peak status measures how biologically prominent the missed signals were.

## Related tools

- **MZmine** (Preprocesses full-scan and MS/MS data separately and exports the .csv feature tables (row ID, row m/z, row retention time, peak area) used to identify base peaks and align SPL vs DDA features)
- **RegFilter** (Produces the filtered scheduled precursor lists whose list-only, unfragmented ions are tested for base-peak status) — https://github.com/computational-chemical-biology/regression_filter

## Examples

```
python -c "import pandas as pd; t=pd.read_csv('full_scan.csv'); t['is_base']=t.groupby('row retention time')['peak area'].transform('max')==t['peak area']"
```

## Evaluation signals

- Per-strain computed fractions equal the reported values: 33.9% for BRA006, 22.4% for BRA010, 29.7% for BRA177
- Feature counts match: 76/135/77 common SPL-DDA features and 59/84/73 list-only ions for BRA006/BRA010/BRA177
- Cross-mode feature matching is restricted to m/z tolerance 0.001 Da or 15 ppm and retention time tolerance 0.2 min (MZmine Join aligner weights: m/z 75, RT 25)
- Every flagged base-peak ion must appear in the full-scan MS1 feature table yet be absent from the DDA MS/MS fragmented precursor set

## Limitations

- DDA fragmentation is stochastic — ions with similar intensities may or may not be fragmented — so base-peak status of list-only ions reflects a snapshot of one acquisition
- Almost all SPL precursor ions were present in DDA MS1 but not all were fragmented, so absence from MS2 does not imply absence from the sample
- Low relative-abundance and low m/z ions are disproportionately captured by the SPL rather than DDA, biasing which ions can be assessed

## Evidence
