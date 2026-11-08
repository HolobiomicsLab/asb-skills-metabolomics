---
name: precursor-ion-overlap-analysis
description: Use when you have a RegFilter-exported scheduled precursor list (.csv)
  and the matching MZmine MS1 feature table for the same LC-MS runs, and need to report
  the percentage of SPL ions present in MS1 and count SPL/DDA fragmented-precursor
  overlaps.
when_to_use_negative:
- The input is already a merged, post-overlap feature table or an annotated molecular
  network rather than a raw SPL csv plus MS1 feature table.
- The goal is RegFilter filtering itself (selecting features for the SPL) or the subsequent
  FBMN/annotation steps rather than evaluating SPL vs DDA overlap.
- Only one dataset is available with no paired DDA MS1 feature table, so no SPL-vs-DDA
  comparison is possible.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine
  role: Source of the full-scan MS1 feature tables and of the m/z and retention-time
    tolerances (Join aligner) used for matching SPL ions to MS1 features
- name: RegFilter
  role: Produces the scheduled precursor list whose ions are matched against the DDA
    MS1 features; its filter thresholds define the SPL under evaluation
  repo: https://github.com/computational-chemical-biology/regression_filter
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:0514cd8a4d50db48e4391ad481f4bfcb8af6dfb9d4cbed51f1250a3c305a1d0f
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

# precursor-ion-overlap-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how many RegFilter-selected scheduled precursor list (SPL) ions are recoverable in DDA full-scan MS1 feature tables and how many are common between the SPL and DDA fragmented precursors, per strain (BRA006, BRA010, BRA177). It validates that a computationally selected targeted acquisition list is grounded in the same features the DDA run detected but often failed to fragment.

## When to use

Use when you have a RegFilter-exported scheduled precursor list (.csv) and the matching MZmine MS1 feature table for the same LC-MS runs, and need to report the percentage of SPL ions present in MS1 and count SPL/DDA fragmented-precursor overlaps.

## When NOT to use

- The input is already a merged, post-overlap feature table or an annotated molecular network rather than a raw SPL csv plus MS1 feature table.
- The goal is RegFilter filtering itself (selecting features for the SPL) or the subsequent FBMN/annotation steps rather than evaluating SPL vs DDA overlap.
- Only one dataset is available with no paired DDA MS1 feature table, so no SPL-vs-DDA comparison is possible.

## Inputs

- MZmine-exported full-scan (MS1) feature table CSV per strain (BRA006, BRA010, BRA177) with row ID, row m/z, and row retention time
- RegFilter-exported scheduled precursor list (.csv) with filtered features and their m/z and retention times
- DDA fragmented precursor list (features with MS2) from the DDA run

## Outputs

- Per-strain percentage of SPL precursor ions present in the DDA MS1 feature table
- Per-strain count of features common between the RegFilter SPL and the DDA fragmented precursors
- Summary table (e.g. spl_dda_overlap_summary.csv) combining overlap percentages and common-feature counts per strain

## How to apply

Load the MZmine-exported MS1 feature tables for strains BRA006, BRA010, and BRA177 and the RegFilter SPL csv (with row m/z and retention time). Match each SPL precursor ion to the DDA MS1 feature table per strain using the same Join aligner tolerances used in preprocessing (m/z tolerance 0.001 m/z or 15 ppm; retention time tolerance 0.2 min). Compute the percentage of SPL precursor ions present in the DDA MS1 table per strain, then count features present both in the RegFilter SPL and in the DDA fragmented precursors. Assemble the per-strain overlap percentages and common-feature counts into one summary table (e.g. spl_dda_overlap_summary.csv). This step is the validation bridge between the RegFilter ion-filtering/SPL construction stage and the downstream fragmentation run: it confirms the SPL targets real, detected-but-unfragmented signals.

## Related tools

- **MZmine** (Source of the full-scan MS1 feature tables and of the m/z and retention-time tolerances (Join aligner) used for matching SPL ions to MS1 features)
- **RegFilter** (Produces the scheduled precursor list whose ions are matched against the DDA MS1 features; its filter thresholds define the SPL under evaluation) — https://github.com/computational-chemical-biology/regression_filter

## Examples

```
```python
# Match RegFilter SPL ions to MZmine MS1 features
spl = pd.read_csv('regfilter_spl.csv')
ms1 = pd.read_csv('mzmine_ms1_BRA006.csv')
# Join aligner tolerances: 0.001 m/z or 15 ppm; 0.2 min RT
overlap = spl.merge(ms1, on=['mz', 'rt'], suffixes=('_spl', '_ms1'), how='inner')
print(f"Overlap %: {100 * len(overlap) / len(spl):.1f}")
print(f"Common features: {len(overlap)}")
```
```

## Evaluation signals

- Overlap percentages reproduce 99.2% (BRA006), 99.0% (BRA010), and 100.0% (BRA177)
- Common-feature counts reproduce 76 (BRA006), 135 (BRA010), and 77 (BRA177)
- The RegFilter SPL size used for overlap is consistent with the article's reported filtered-feature counts (135 for BRA006, 219 for BRA010, 150 for BRA177)
- Matching uses the Join aligner tolerances: 0.001 m/z or 15 ppm and 0.2 min retention time

## Limitations

- The overlap validates presence in MS1 but not that the ion was fragmented in the DDA run; most SPL ions were present in DDA MS1 yet not all were fragmented.
- DDA's stochastic nature means ions with similar intensities may or may not be fragmented, so the common-feature count depends on the DDA cycle order, not on ion quality.
- Results are specific to the strains and LC-MS/MS settings described; the article does not publish a reusable script for this overlap computation.

## Evidence

- [results] SPL precursor ions overlapped DDA MS1 features by 99.2% (BRA006), 99.0% (BRA010), and 100.0% (BRA177): [evidence span withheld — anchor status: not_found]
