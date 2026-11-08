---
name: list-only-ion-fraction-quantification
description: Use when MZmine full-scan and DDA MS/MS feature tables plus a RegFilter
  SPL exist and you need to quantify coverage losses — e.g. 'what fraction of list-only
  ions were MS1 base peaks that DDA failed to fragment?'
when_to_use_negative:
- When the goal is annotation or molecular networking of the fragmented features (FBMN/GNPS2,
  SIRIUS, ChemWalker) rather than quantifying DDA coverage loss.
- When the input is raw instrument files that have not yet been converted (MSConvert
  to .mzXML) and preprocessed in MZmine — the join/flagging logic requires aligned
  feature tables.
- When the SPL ions already have DDA MS2 spectra (common features) — the skill targets
  only list-only ions lacking MS2 fragmentation.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine 2.53
  role: Provides the preprocessed full-scan and MS/MS feature tables and the Join
    aligner tolerances (m/z 0.001 m/z or 15 ppm; RT 0.2) used to flag list-only ions
- name: RegFilter
  role: Produces the scheduled precursor list (SPL) features whose unfragmented fraction
    is being quantified
  repo: https://github.com/computational-chemical-biology/regression_filter
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:992e500e6596ecd0ef6de11fb9725c35ae8fd3db3502035376f62c99d7e0f764
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# list-only-ion-fraction-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how many RegFilter scheduled-precursor-list (SPL) ions detected in DDA MS1 full scan were nonetheless not fragmented by DDA — specifically, the fraction that were base peaks at their retention point. For strains BRA006, BRA010, and BRA177 this yielded 33.9%, 22.4%, and 29.7%, respectively, exposing the stochastic fragmentation gap of DDA acquisition.

## When to use

Use when MZmine full-scan and DDA MS/MS feature tables plus a RegFilter SPL exist and you need to quantify coverage losses — e.g. 'what fraction of list-only ions were MS1 base peaks that DDA failed to fragment?'

## When NOT to use

- When the goal is annotation or molecular networking of the fragmented features (FBMN/GNPS2, SIRIUS, ChemWalker) rather than quantifying DDA coverage loss.
- When the input is raw instrument files that have not yet been converted (MSConvert to .mzXML) and preprocessed in MZmine — the join/flagging logic requires aligned feature tables.
- When the SPL ions already have DDA MS2 spectra (common features) — the skill targets only list-only ions lacking MS2 fragmentation.

## Inputs

- MZmine-preprocessed full-scan feature table (.csv with row ID, row m/z, row retention time, peak area) per strain
- MZmine DDA MS/MS feature table per strain (BRA006, BRA010, BRA177)
- RegFilter-filtered SPL feature list per strain (135 BRA006, 219 BRA010, 150 BRA177 features)

## Outputs

- list_only_ion_fractions.csv with per-strain fraction of list-only ions that were full-scan base peaks but not fragmented by DDA
- Common-feature counts per strain (76 BRA006, 135 BRA010, 77 BRA177) and list-only counts (59, 84, 73)

## How to apply

Load MZmine 2.53-preprocessed full-scan feature tables (.csv with row ID, row m/z, row retention time, peak area) and the DDA MS/MS feature tables for each strain, plus the RegFilter-filtered SPL features (135 for BRA006, 219 for BRA010, 150 for BRA177). Match SPL precursors against the DDA fragmented precursor set using Join aligner tolerances (m/z 0.001 Da or 15 ppm; retention time 0.2 min) to recover common features (76, 135, 77 per strain); the remainder present in DDA MS1 are the list-only ions (59, 84, 73). For each list-only ion, look up its full-scan peak area and test whether it was the highest-intensity (base peak) ion at its retention point. Compute the per-strain fraction of list-only ions that were full-scan base peaks yet unfragmented (33.9% BRA006, 22.4% BRA010, 29.7% BRA177) and export, e.g. list_only_ion_fractions.csv. The rationale is that DDA's stochastic nature skips ions of comparable intensity, so base-peak status among unfragmented list-only ions quantifies the severity of that loss.

## Related tools

- **MZmine 2.53** (Provides the preprocessed full-scan and MS/MS feature tables and the Join aligner tolerances (m/z 0.001 m/z or 15 ppm; RT 0.2) used to flag list-only ions)
- **RegFilter** (Produces the scheduled precursor list (SPL) features whose unfragmented fraction is being quantified) — https://github.com/computational-chemical-biology/regression_filter

## Examples

```
frac = df.groupby('strain').apply(lambda g: (g['is_base_peak'] & g['list_only']).sum()/g['list_only'].sum()*100).round(1); frac.to_csv('list_only_ion_fractions.csv')
```

## Evaluation signals

- Per-strain fractions must equal 33.9% (BRA006), 22.4% (BRA010), and 29.7% (BRA177).
- Common-feature counts must be 76 (BRA006), 135 (BRA010), 77 (BRA177) when joining SPL to DDA fragmented precursors with m/z 0.001 Da or 15 ppm and RT 0.2 min tolerances.
- List-only counts must be 59 (BRA006), 84 (BRA010), 73 (BRA177), and common + list-only must not exceed the SPL totals (135, 219, 150).
- Each list-only ion must be resolvable to a full-scan peak area in the MZmine table to test base-peak status; ions absent from DDA MS1 must be excluded from the fraction denominator.
- Output CSV must contain one fraction per strain with values in [0, 1] (reported as percentages here).

## Limitations

- DDA's stochastic nature means ions with similar intensities may or may not be fragmented, so the base-peak fraction fluctuates run to run and cannot be treated as deterministic.
- Ions with low m/z are more frequently captured by the SPL than fragmented by DDA, so the metric is biased toward low-mass features.
- RegFilter can select ions of low relative abundance; only list-only ions present in DDA MS1 are scored — features absent from MS1 entirely are not captured by this fraction.

## Evidence
