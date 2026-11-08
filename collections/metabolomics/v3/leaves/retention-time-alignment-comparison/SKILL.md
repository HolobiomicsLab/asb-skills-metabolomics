---
name: retention-time-alignment-comparison
description: Use when you have an MZmine-exported full-scan (MS1) feature table and
  a RegFilter-generated SPL .csv for the same LC-MS run and need to quantify what
  fraction of targeted precursor ions exist as DDA MS1 features or were actually fragmented
  in DDA.
when_to_use_negative:
- Input is already a DDA MS2 peak list with no RegFilter SPL — there is nothing to
  align against and the overlap metric is undefined
- The full-scan and DDA runs were not processed with the same MZmine parameters (m/z
  0.001 m/z or 15 ppm; RT 0.2), since the Join aligner tolerances are what make the
  overlap match valid
- You need to assess spectral identity of fragments, not precursor-level presence/absence
  — this skill only compares m/z and RT coordinates
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: RegFilter
  role: Generates the scheduled precursor list (SPL) of filtered features that is
    aligned against the DDA MS1 feature table; repo provides the notebook building_spl_and_assessing_XIC_for_all_ions.ipynb
    for the filtering algorithm
  repo: https://github.com/computational-chemical-biology/regression_filter
- name: MZmine 2.53
  role: Preprocessed full scan and MS/MS data separately and performed the Join aligner
    whose tolerances (m/z 0.001 m/z or 15 ppm; RT 0.2 min) define the matching criteria
    for the overlap comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:648c247231b03a4c44898568577d74f5318d7d7211691ec1d786b563447066a0
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
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# retention-time-alignment-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Matches RegFilter-selected scheduled precursor list (SPL) ions against DDA MS1 feature tables using MZmine's Join aligner tolerances (m/z 0.001 or 15 ppm; RT 0.2 min) to quantify overlap and common-feature counts per strain, verifying that RegFilter covers chromatographic signals missed by DDA.

## When to use

Use when you have an MZmine-exported full-scan (MS1) feature table and a RegFilter-generated SPL .csv for the same LC-MS run and need to quantify what fraction of targeted precursor ions exist as DDA MS1 features or were actually fragmented in DDA.

## When NOT to use

- Input is already a DDA MS2 peak list with no RegFilter SPL — there is nothing to align against and the overlap metric is undefined
- The full-scan and DDA runs were not processed with the same MZmine parameters (m/z 0.001 m/z or 15 ppm; RT 0.2), since the Join aligner tolerances are what make the overlap match valid
- You need to assess spectral identity of fragments, not precursor-level presence/absence — this skill only compares m/z and RT coordinates

## Inputs

- MZmine-exported full-scan (MS1) feature table (.csv) for each actinomycete strain (BRA006, BRA010, BRA177)
- RegFilter-exported scheduled precursor list (.csv) containing filtered features with m/z and retention times
- DDA fragmented precursor list (MS2-associated precursor m/z / RT pairs)

## Outputs

- Per-strain percentage of SPL precursor ions present in the DDA MS1 feature table (expected 99.2%, 99.0%, 100.0%)
- Per-strain count of features common between the SPL and DDA fragmented precursors (expected 76, 135, 77)
- Summary table (spl_dda_overlap_summary.csv)

## How to apply

Load the MZmine-exported full-scan (MS1) feature table for each strain (BRA006, BRA010, BRA177) and the RegFilter-exported scheduled precursor list (.csv) containing filtered features with their m/z and retention times. Match each SPL precursor ion to the DDA MS1 feature table per strain using the tolerances adopted in the MZmine Join aligner step: m/z tolerance 0.001 m/z or 15 ppm, retention time tolerance 0.2 min. Compute the percentage of SPL precursor ions present in the DDA MS1 feature table per strain — the article reports 99.2% (BRA006), 99.0% (BRA010), and 100.0% (BRA177). Then count features present both in the RegFilter SPL and in the DDA fragmented precursors (76, 135, and 77 for BRA006, BRA010, and BRA177 respectively). Rationale: because almost all SPL-selected precursor ions were present in DDA MS1 but not all were fragmented, this comparison isolates whether RegFilter's coverage gain comes from targeted acquisition rather than mere feature detection.

## Related tools

- **RegFilter** (Generates the scheduled precursor list (SPL) of filtered features that is aligned against the DDA MS1 feature table; repo provides the notebook building_spl_and_assessing_XIC_for_all_ions.ipynb for the filtering algorithm) — https://github.com/computational-chemical-biology/regression_filter
- **MZmine 2.53** (Preprocessed full scan and MS/MS data separately and performed the Join aligner whose tolerances (m/z 0.001 m/z or 15 ppm; RT 0.2 min) define the matching criteria for the overlap comparison)

## Examples

```
from regfilter import build_spl; build_spl(input_csv='features_fullscan.csv', npts=5, output_csv='spl.csv')
```

## Evaluation signals

- Computed overlap percentages equal 99.2% (BRA006), 99.0% (BRA010), and 100.0% (BRA177) as reported in the article results
- Common-feature counts equal 76 (BRA006), 135 (BRA010), and 77 (BRA177) matching the article's reported values
- Every SPL precursor match must satisfy the Join aligner tolerances: m/z ≤ 0.001 m/z (or 15 ppm) and RT ≤ 0.2 min
- Per-strain SPL-to-DDA-MS1 overlap should be near-complete (≥99%) because the article states precursor ions selected in the fragmentation list were almost all present in DDA MS1 but not all fragmented
- spl_dda_overlap_summary.csv exports one row per strain with columns for overlap percentage and common-feature count

## Limitations

- DDA's stochastic nature means ions with similar intensities may or may not be fragmented, so common-feature counts vary between replicates and reflect fragmentation chance, not signal quality
- RegFilter was able to select ions of low relative abundance, so overlap percentages can be inflated when the DDA MS1 feature table detects features that DDA never fragmented
- Tolerances are inherited from MZmine's Join aligner (0.001 m/z or 15 ppm; RT 0.2 min); runs preprocessed with different alignment parameters will yield non-comparable overlap statistics

## Evidence

- [results] SPL precursor ions overlapped DDA MS1 features by 99.2% (BRA006), 99.0% (BRA010), and 100.0% (BRA177): [evidence span withheld — anchor status: not_found]
- [results] 76, 135, and 77 features, respectively, were present both in the SPL and in DDA fragmented precursors: [evidence span withheld — anchor status: not_found]
- [readme] RegFilter notebook is the entry point for generating the SPL before alignment: [evidence span withheld — anchor status: not_found]
