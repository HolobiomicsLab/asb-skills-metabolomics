---
name: mass-spectrometry-feature-tracking
description: Use when MZmine full-scan (.csv) feature tables and DDA MS/MS tables
  are available and the goal is to quantify which targeted-list ions were detected
  in MS1 but lacked MS2 fragmentation.
when_to_use_negative:
- Input is raw vendor data (.raw/.mzXML) rather than an already-preprocessed MZmine
  feature table — preprocessing with MZmine 2.53 must happen first.
- The DDA run already fragmented all targeted precursors, leaving no list-only ions
  to characterize.
- The goal is general molecular networking or annotation (FBMN, SIRIUS, ChemWalker)
  rather than auditing which SPL ions DDA failed to fragment.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine 2.53
  role: Preprocesses full-scan and MS/MS data separately and exports feature tables
    (.csv with row ID, row m/z, row retention time, peak area) whose Join aligner
    tolerances (m/z 0.001 Da or 15 ppm; RT 0.2 min) define the feature-matching step.
- name: RegFilter
  role: Generates the scheduled precursor list (SPL) of filtered features (135 BRA006,
    219 BRA010, 150 BRA177) whose ions are tracked against the DDA fragmented set.
  repo: https://github.com/computational-chemical-biology/regression_filter
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:58b1500e4b862617b027a1a45f12d1fb017e0786f4a9b429df695ed88b9fa36e
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
  - MZmine 2.53
  - RegFilter
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# mass-spectrometry-feature-tracking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tracks individual metabolomic features across full-scan MS1 and DDA MS/MS acquisition using MZmine feature tables, quantifying ions present in full scan but never fragmented by DDA (list-only ions) and whether they were base peaks. This quantifies how much biological signal DDA's stochastic precursor selection leaves behind.

## When to use

Use when MZmine full-scan (.csv) feature tables and DDA MS/MS tables are available and the goal is to quantify which targeted-list ions were detected in MS1 but lacked MS2 fragmentation.

## When NOT to use

- Input is raw vendor data (.raw/.mzXML) rather than an already-preprocessed MZmine feature table — preprocessing with MZmine 2.53 must happen first.
- The DDA run already fragmented all targeted precursors, leaving no list-only ions to characterize.
- The goal is general molecular networking or annotation (FBMN, SIRIUS, ChemWalker) rather than auditing which SPL ions DDA failed to fragment.

## Inputs

- MZmine full-scan feature table (.csv with row ID, row m/z, row retention time, peak area) per strain
- DDA MS/MS feature table per strain
- RegFilter-filtered scheduled precursor list (SPL) features per strain (135 for BRA006, 219 for BRA010, 150 for BRA177)

## Outputs

- list_only_ion_fractions.csv with per-strain fractions of list-only ions that were full-scan base peaks missed by DDA (33.9% BRA006, 22.4% BRA010, 29.7% BRA177)
- Counts of common (SPL ∩ fragmented) and list-only features per strain

## How to apply

Load the MZmine-preprocessed full-scan feature tables (row ID, row m/z, row retention time, peak area) and the DDA MS/MS feature tables for each strain (BRA006, BRA010, BRA177). Match the RegFilter scheduled precursor list (SPL) precursors against the DDA fragmented precursor set using Join aligner tolerances — m/z tolerance 0.001 Da or 15 ppm and retention time tolerance 0.2 min — to recover common features (76 BRA006, 135 BRA010, 77 BRA177). Flag list-only ions: SPL features present in DDA MS1 but lacking MS2 fragmentation (59 BRA006, 84 BRA010, 73 BRA177). For each list-only ion, query its full-scan peak area in the MZmine table to determine whether it was the base peak (highest-intensity ion) at its retention point. Compute per-strain fractions (33.9% BRA006, 22.4% BRA010, 29.7% BRA177) and export them to list_only_ion_fractions.csv. The rationale is that DDA's stochastic precursor selection misses ions of similar intensity, so tracking features across MS1/MS2 reveals the scale of missed biological signals.

## Related tools

- **MZmine 2.53** (Preprocesses full-scan and MS/MS data separately and exports feature tables (.csv with row ID, row m/z, row retention time, peak area) whose Join aligner tolerances (m/z 0.001 Da or 15 ppm; RT 0.2 min) define the feature-matching step.)
- **RegFilter** (Generates the scheduled precursor list (SPL) of filtered features (135 BRA006, 219 BRA010, 150 BRA177) whose ions are tracked against the DDA fragmented set.) — https://github.com/computational-chemical-biology/regression_filter

## Examples

```
import pandas as pd; spl = pd.read_csv('SPL_BRA006.csv'); dda = pd.read_csv('dda_ms2_BRA006.csv'); common = spl.merge(dda, on=['row m/z','row retention time'], how='inner'); list_only = spl[~spl['row ID'].isin(common['row ID'])]
```

## Evaluation signals

- The per-strain fractions must equal 33.9% (BRA006), 22.4% (BRA010), and 29.7% (BRA177) to validate the computation.
- Common-feature counts must be 76 (BRA006), 135 (BRA010), and 77 (BRA177) after Join aligner matching at 0.001 Da or 15 ppm m/z and 0.2 min RT tolerance.
- List-only ion counts must equal 59 (BRA006), 84 (BRA010), and 73 (BRA177).
- Every matched pair must satisfy both m/z tolerance (0.001 Da or 15 ppm) and retention time tolerance (0.2 min).
- Base-peak assignment must rely on full-scan peak area in the MZmine table (highest-intensity ion at the retention point).

## Limitations

- DDA's stochastic nature means ions with similar intensities may or may not be fragmented, so the list-only set partly reflects acquisition randomness rather than true absence.
- Almost all SPL-selected precursors were present in DDA MS1, so MS1 detection is not the bottleneck — the characterization targets fragmentation failure specifically.
- Ions with low m/z were more frequently captured by the SPL than fragmented by DDA, so results may be biased toward low-mass features.
- RegFilter was able to select ions of low relative abundance, so list-only ions are often weak signals where base-peak determination is sensitive to peak-picking thresholds (MS1 noise level 1E3, min peak height 3E3 in MZmine).

## Evidence
