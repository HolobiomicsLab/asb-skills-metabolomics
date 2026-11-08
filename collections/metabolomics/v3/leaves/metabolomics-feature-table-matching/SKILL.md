---
name: metabolomics-feature-table-matching
description: Use when you have a RegFilter-exported scheduled precursor list (.csv)
  and MZmine MS1 feature tables and need to compute per-strain overlap percentages
  and common-feature counts between SPL and DDA precursors.
when_to_use_negative:
- Input is still raw LC-MS/MS data (mzXML) — MZmine preprocessing (mass detection,
  ADAP chromatogram building, Join aligner) must run first to obtain feature tables
- No RegFilter-generated scheduled precursor list exists — this skill quantifies SPL-vs-DDA
  overlap only, not general feature table alignment between unrelated experiments
- The goal is MS2-level comparison (spectral pairing, molecular networking) rather
  than MS1 precursor presence/absence matching
tools:
- name: RegFilter
  role: Produces the scheduled precursor list (.csv) of statistically filtered features
    that is matched against the DDA MS1 feature table
  repo: https://github.com/computational-chemical-biology/regression_filter
- name: MZmine 2.53
  role: Preprocesses full-scan and MS/MS modes separately and exports the MS1 feature
    tables whose Join aligner tolerances (m/z 0.001 m/z or 15 ppm; RT 0.2) define
    the matching parameters
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:1c028546bd5ca48bab6ec9f4b3bb8a131a375fe137d51e56486251ede47f8973
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# metabolomics-feature-table-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Matches RegFilter-generated scheduled precursor list (SPL) ions against MZmine-exported DDA MS1 feature tables to quantify precursor overlap and count features shared between SPL and DDA fragmented precursors, per strain (BRA006, BRA010, BRA177). It validates that statistically selected ions for targeted fragmentation are actually observable in untargeted DDA full-scan data.

## When to use

Use when you have a RegFilter-exported scheduled precursor list (.csv) and MZmine MS1 feature tables and need to compute per-strain overlap percentages and common-feature counts between SPL and DDA precursors.

## When NOT to use

- Input is still raw LC-MS/MS data (mzXML) — MZmine preprocessing (mass detection, ADAP chromatogram building, Join aligner) must run first to obtain feature tables
- No RegFilter-generated scheduled precursor list exists — this skill quantifies SPL-vs-DDA overlap only, not general feature table alignment between unrelated experiments
- The goal is MS2-level comparison (spectral pairing, molecular networking) rather than MS1 precursor presence/absence matching

## Inputs

- MZmine-exported full-scan (MS1) feature table (.csv) per strain (BRA006, BRA010, BRA177) with row ID, row m/z, row retention time
- RegFilter-exported scheduled precursor list (.csv) of filtered features with m/z and retention times
- DDA fragmented precursor list per strain (features with MS2)

## Outputs

- Per-strain SPL-vs-DDA-MS1 overlap percentages (e.g. 99.2%, 99.0%, 100.0%)
- Per-strain counts of features common to SPL and DDA fragmented precursors (e.g. 76, 135, 77)
- spl_dda_overlap_summary.csv summary table

## How to apply

Load the MZmine-exported full-scan (MS1) feature tables for each strain alongside the RegFilter-exported scheduled precursor list (.csv) containing filtered features with their m/z and retention times. Match each SPL precursor ion to the DDA MS1 feature table per strain using the same tolerances as the MZmine Join aligner step (m/z tolerance 0.001 m/z or 15 ppm; retention time tolerance 0.2 min). For each strain, compute the percentage of SPL ions present as DDA MS1 features, then count features present both in the RegFilter SPL and in the DDA fragmented precursors. Assemble the per-strain overlap percentages and common-feature counts into a summary table (spl_dda_overlap_summary.csv). The rationale is that almost all SPL ions should exist in DDA MS1 even when DDA's stochastic acquisition failed to fragment them, so high overlap confirms the targeted list recovers signals the untargeted method detected but did not fragment.

## Related tools

- **RegFilter** (Produces the scheduled precursor list (.csv) of statistically filtered features that is matched against the DDA MS1 feature table) — https://github.com/computational-chemical-biology/regression_filter
- **MZmine 2.53** (Preprocesses full-scan and MS/MS modes separately and exports the MS1 feature tables whose Join aligner tolerances (m/z 0.001 m/z or 15 ppm; RT 0.2) define the matching parameters)

## Evaluation signals

- Computed overlap percentages equal 99.2% (BRA006), 99.0% (BRA010), and 100.0% (BRA177), matching the article results
- Common-feature counts equal 76 (BRA006), 135 (BRA010), and 77 (BRA177)
- Matching uses the same tolerances as the MZmine Join aligner (m/z 0.001 m/z or 15 ppm; retention time tolerance 0.2 min) so matches are consistent with the alignment step
- Summary table exports as spl_dda_overlap_summary.csv with one overlap percentage and one common-feature count per strain

## Limitations

- DDA fragmentation is stochastic: ions with similar intensities may or may not be fragmented, so a shared MS1 feature is not guaranteed to have an MS2 spectrum
- High MS1 overlap does not mean the ions were fragmented — 'almost all of the precursor ions selected in the fragmentation list were present in the MS1 of the DDA method, but not all were fragmented'
- Ions with low m/z were more frequently captured by the SPL than fragmented by DDA, so overlap statistics can mask acquisition-mode bias

## Evidence
