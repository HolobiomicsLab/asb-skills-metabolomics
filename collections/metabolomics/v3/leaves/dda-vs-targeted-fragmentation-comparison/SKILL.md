---
name: dda-vs-targeted-fragmentation-comparison
description: Use when evaluating how many ions missed or not fragmented in DDA mode
  would be captured by a targeted scheduled precursor list derived from full-scan
  MS1 features.
when_to_use_negative:
- Input data is already an MS/MS spectral library or annotated network (FBMN/GNPS
  output) rather than unprocessed MZmine MS1 feature tables and a RegFilter SPL
- The comparison targets ions already fragmented in DDA mode and needs no targeting,
  so overlap accounting adds no information beyond DDA coverage itself
- Sample set has no paired full-scan and DDA acquisitions for the same strains, so
  SPL precursors cannot be matched to an MS1 reference table
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZmine
  role: Used to export the full-scan MS1 feature tables and align features across
    samples, providing the m/z and RT values against which SPL precursor ions are
    matched.
- name: RegFilter
  role: Generates the scheduled precursor list (.csv) of filtered features whose overlap
    with DDA MS1 and DDA fragmented precursors is being quantified.
  repo: https://github.com/computational-chemical-biology/regression_filter
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:107eb32c4ea2723e1e95425a50c8eeec92d290fcd96a55b6e51adfd0cee4f429
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
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# dda-vs-targeted-fragmentation-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compares an untargeted DDA acquisition against a RegFilter-generated scheduled precursor list (SPL) by matching SPL m/z–RT entries to MZmine MS1 feature tables, quantifying precursor overlap and common fragmented features per strain.

## When to use

Use when evaluating how many ions missed or not fragmented in DDA mode would be captured by a targeted scheduled precursor list derived from full-scan MS1 features.

## When NOT to use

- Input data is already an MS/MS spectral library or annotated network (FBMN/GNPS output) rather than unprocessed MZmine MS1 feature tables and a RegFilter SPL
- The comparison targets ions already fragmented in DDA mode and needs no targeting, so overlap accounting adds no information beyond DDA coverage itself
- Sample set has no paired full-scan and DDA acquisitions for the same strains, so SPL precursors cannot be matched to an MS1 reference table

## Inputs

- MZmine-exported full-scan (MS1) feature table (.csv) per strain
- RegFilter-exported scheduled precursor list (.csv) containing filtered features with m/z and retention times
- DDA fragmented precursor lists per strain (from MS/MS mode preprocessing)

## Outputs

- Per-strain percentage of SPL precursor ions present in DDA MS1 feature table
- Per-strain count of features common to the SPL and DDA fragmented precursors
- Summary table (e.g. spl_dda_overlap_summary.csv) with overlap percentages and common-feature counts

## How to apply

Load the MZmine-exported MS1 feature tables for strains BRA006, BRA010, and BRA177 together with the RegFilter .csv scheduled precursor list, then match each SPL precursor ion to the MS1 table using the MZmine Join aligner tolerances (m/z tolerance 0.001 m/z or 15 ppm; retention time tolerance 0.2 min). Compute the percentage of SPL precursors present in DDA MS1 per strain and count features present both in the SPL and in DDA fragmented precursors. The article reports overlap percentages of 99.2% (BRA006), 99.0% (BRA010), and 100.0% (BRA177), and 76, 135, and 77 common features respectively — these values serve as reference expectations when reproducing the comparison. Rationale: DDA's stochastic nature means ions with similar intensities may or may not be fragmented, so the overlap quantifies how much targeted acquisition covers beyond DDA. Export the per-strain summary as a single table for downstream annotation-level comparison.

## Related tools

- **MZmine** (Used to export the full-scan MS1 feature tables and align features across samples, providing the m/z and RT values against which SPL precursor ions are matched.)
- **RegFilter** (Generates the scheduled precursor list (.csv) of filtered features whose overlap with DDA MS1 and DDA fragmented precursors is being quantified.) — https://github.com/computational-chemical-biology/regression_filter

## Examples

```
python building_spl_and_assessing_XIC_for_all_ions.ipynb  # then match SPL precursors to MZmine MS1 table with m/z tolerance 0.001 m/z (15 ppm) and RT tolerance 0.2 min
```

## Evaluation signals

- Computed SPL-to-MS1 overlap percentages should equal 99.2% (BRA006), 99.0% (BRA010), and 100.0% (BRA177) when the article's tolerances (0.001 m/z or 15 ppm; 0.2 min RT) are used.
- Common-feature counts between SPL and DDA fragmented precursors should equal 76 (BRA006), 135 (BRA010), and 77 (BRA177).
- Feature matching must use the same tolerances as the MZmine Join aligner step — m/z 0.001 m/z or 15 ppm, retention time 0.2 min — otherwise counts will shift.
- Total features detected in full scan mode should be higher than in DDA mode, providing a sanity check that the full-scan feature table is complete.
- Output summary table should contain one row per strain with both overlap percentage and common-feature count, matching the article's three-strain (BRA006, BRA010, BRA177) structure.

## Limitations

- The comparison depends on the stochastic nature of DDA: ions with similar intensities may or may not be fragmented, so overlap percentages can vary across acquisition runs.
- DDA's limitation of missing low-abundance ions means SPL coverage of low m/z features is higher, so overlap percentages do not directly reflect SPL quality alone.
- Values (99.2%, 99.0%, 100.0% overlap; 76/135/77 common features) are specific to the article's three actinomycete strains (BRA006, BRA010, BRA177) and their MZmine/RegFilter settings; other datasets require recalibrated expectations.

## Evidence

- [readme] RegFilter SPL .csv export format and generation workflow.: [evidence span withheld — anchor status: not_found]
