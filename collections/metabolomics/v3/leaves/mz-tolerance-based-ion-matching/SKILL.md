---
name: mz-tolerance-based-ion-matching
description: Use when verifying that ions on a curated scheduled precursor list are
  present as detected MS1 features in an independent feature table, and when counting
  features shared with a DDA fragmented-precursor set.
when_to_use_negative:
- When the input table lacks m/z and retention-time columns (e.g., an MS2-only export
  filtered to 'Only with MS2'), the tolerance-based match cannot be computed
- When the MZmine preprocessing used different aligner/deconvolution tolerances —
  reusing 0.001 m/z or 15 ppm and 0.2 min would then be inconsistent with the feature
  definitions
- When the goal is molecular annotation (GNPS-FBMN, SIRIUS) rather than validating
  SPL coverage against MS1
tools:
- name: MZmine 2.53
  role: 'produces the MS1 feature tables and defines the matching tolerances (Join
    aligner: 0.001 m/z or 15 ppm, RT 0.2 min) reused for SPL matching'
- name: RegFilter
  role: generates the scheduled precursor list (.csv) of filtered features that is
    matched against the DDA MS1 features
  repo: https://github.com/computational-chemical-biology/regression_filter
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:c7d56328eee93974e3daf83562ef1d7a9f51fa2b35800f39450fef73b69fedae
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
n_claims: 15
---

# mz-tolerance-based-ion-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Matches ions from a RegFilter-generated scheduled precursor list (SPL) against MZmine full-scan (MS1) feature tables using fixed m/z and retention-time tolerances (0.001 m/z or 15 ppm; 0.2 min) to quantify SPL-to-DDA-MS1 overlap and count features shared between the SPL and DDA fragmented precursors.

## When to use

Use when verifying that ions on a curated scheduled precursor list are present as detected MS1 features in an independent feature table, and when counting features shared with a DDA fragmented-precursor set.

## When NOT to use

- When the input table lacks m/z and retention-time columns (e.g., an MS2-only export filtered to 'Only with MS2'), the tolerance-based match cannot be computed
- When the MZmine preprocessing used different aligner/deconvolution tolerances — reusing 0.001 m/z or 15 ppm and 0.2 min would then be inconsistent with the feature definitions
- When the goal is molecular annotation (GNPS-FBMN, SIRIUS) rather than validating SPL coverage against MS1

## Inputs

- MZmine-exported full-scan (MS1) feature table (.csv with row ID, row m/z, row retention time)
- RegFilter-exported scheduled precursor list (.csv of filtered features with m/z and retention time)
- DDA fragmented precursor list per strain

## Outputs

- Per-strain percentage of SPL precursor ions present in the DDA MS1 feature table
- Per-strain count of features present both in the SPL and in DDA fragmented precursors
- spl_dda_overlap_summary.csv combining overlap percentages and common-feature counts

## How to apply

Load the MZmine-exported MS1 feature table (.csv exporting row ID, row m/z, row retention time) and the RegFilter-exported SPL (.csv) per strain (BRA006, BRA010, BRA177). Match each SPL ion to MS1 features with the same tolerances used in the MZmine Join aligner step — m/z tolerance 0.001 m/z or 15 ppm (weight 75) and retention-time tolerance 0.2 min (weight 25) — so matching is consistent with the upstream preprocessing. Compute the percentage of SPL ions present in the MS1 table and the count of features present both in the SPL and in the DDA fragmented precursors. Rationale: DDA is stochastic and misses ions present in MS1 (especially low-abundance and low-m/z ions), so overlap quantification confirms the SPL is targeting real, detectable signals rather than noise. Assemble per-strain overlap percentages and common-feature counts into spl_dda_overlap_summary.csv.

## Related tools

- **MZmine 2.53** (produces the MS1 feature tables and defines the matching tolerances (Join aligner: 0.001 m/z or 15 ppm, RT 0.2 min) reused for SPL matching)
- **RegFilter** (generates the scheduled precursor list (.csv) of filtered features that is matched against the DDA MS1 features) — https://github.com/computational-chemical-biology/regression_filter

## Examples

```
matched = spl.apply(lambda r: ((abs(ms1.mz - r.mz) / r.mz * 1e6 <= 15) & (abs(ms1.rt - r.rt) <= 0.2)).any(), axis=1)
```

## Evaluation signals

- Overlap percentages must equal 99.2% (BRA006), 99.0% (BRA010), and 100.0% (BRA177) when the article's tolerances are used
- Common-feature counts between SPL and DDA fragmented precursors must equal 76 (BRA006), 135 (BRA010), and 77 (BRA177)
- SPL input sizes must be 135 (BRA006), 219 (BRA010), and 150 (BRA177) filtered features before matching
- Matching tolerances must be identical to the MZmine Join aligner settings (m/z 0.001 m/z or 15 ppm; RT 0.2 min)
- Output summary table contains one row per strain with both overlap percentage and common-feature count columns

## Limitations

- Presence in MS1 does not guarantee fragmentation: 'almost all of the precursor ions selected in the fragmentation list were present in the MS1 of the DDA method, but not all were fragmented'
- DDA's stochastic nature means ions with similar intensities may or may not be fragmented, so common-feature counts understate true detectability
- Matching is only valid if the same tolerances used in MZmine preprocessing are reused; the article does not supply a DOI or code for the matching step itself

## Evidence

- [results] 76 features, 135 features, and 77 features for strains BRA006, BRA010, and BRA177, respectively, were present both in the scheduled precursors list (SPL) and in the fragmented: [evidence span withheld — anchor status: not_found]
