---
name: subgroup-fdr-stratification
description: Reference for controlling FDR in open searches where modifications shift
  precursor masses and a single global target-decoy FDR would underestimate error
  rates.
when_to_use_negative:
- The search is a standard (narrow-window, unmodified-peptide) library search where
  a single global FDR is appropriate rather than mass-difference subgroups.
- Subgroups are too small (fewer than 20 SSMs per group), as the article warns the
  actual FDR may be underestimated.
- Only matches within a single narrow precursor mass window are analyzed, making mass-difference
  stratification unnecessary.
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ANN-SoLo
  role: Generates the target-decoy SSMs and charge-normalized precursor mass differences
    that are subgrouped and FDR-filtered; applies the subgroup FDR strategy for the
    open-search level of the cascade search.
  repo: https://github.com/bittremieux/ANN-SoLo
- name: SpectraST
  role: Used to prepare the spectral library (removing duplicates) so the 1:1 target-decoy
    ratio that underpins subgroup FDR estimation is in place before searching.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:74ca41d4fa989642b03bf456e05b7fce1b0817b1ceeef08d1108c8b202e5f0ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/ANN-SoLo
  tools:
  - ANN-SoLo
  - Python
  - SpectraST
  - msconvert
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# subgroup-fdr-stratification

## Summary

A false discovery rate estimation strategy for open modification spectral library searches in which SSMs are grouped by their charge-normalized precursor mass difference between query and library spectra, and FDR is computed within each subgroup before filtering.

## When to use

Reference for controlling FDR in open searches where modifications shift precursor masses and a single global target-decoy FDR would underestimate error rates.

## When NOT to use

- The search is a standard (narrow-window, unmodified-peptide) library search where a single global FDR is appropriate rather than mass-difference subgroups.
- Subgroups are too small (fewer than 20 SSMs per group), as the article warns the actual FDR may be underestimated.
- Only matches within a single narrow precursor mass window are analyzed, making mass-difference stratification unnecessary.

## Inputs

- Query spectra matched by ANN-SoLo from an open spectral library search (MGF-converted raw data from e.g. PRIDE PXD001468)
- Target-decoy SSM results from a two-level cascade search, including per-SSM precursor mass differences and charges
- Subgroup definition parameters (e.g. 0.1 Da charge-normalized precursor mass difference bins)
- Minimum subgroup size threshold (e.g. 20 SSMs)
- FDR threshold (e.g. 0.01)

## Outputs

- Per-subgroup FDR estimates for each 0.1 Da charge-normalized precursor mass difference bin
- Filtered list of confident SSMs below the FDR threshold across both cascade search levels
- Final identification counts (e.g. ann_solo_hek293_idents.csv in the source card workflow)
- Diagnostic record of which subgroups had fewer than 20 SSMs and were excluded from FDR filtering

## How to apply

After running both levels of the ANN-SoLo cascade search, group target and decoy SSMs by the charge-normalized precursor mass difference between the query spectrum and the matched library spectrum (e.g. 0.1 Da subgroups on the HEK293 open search). For each subgroup, estimate the false discovery rate as the ratio of decoy SSMs to target SSMs, applying this only to subgroups containing at least 20 SSMs. Merge confident SSMs (at 1% FDR) from both the level-1 narrow-window search and the level-2 shifted-dot-product open search and retain matches below the FDR threshold. Because the article cautions that too-small groups can underestimate FDR, verify subgroup counts before accepting results. This strategy assumes decoy spectra are appended 1:1 to the library via the shuffle-and-reposition method so decoy/target ratios are meaningful.

## Related tools

- **ANN-SoLo** (Generates the target-decoy SSMs and charge-normalized precursor mass differences that are subgrouped and FDR-filtered; applies the subgroup FDR strategy for the open-search level of the cascade search.) — https://github.com/bittremieux/ANN-SoLo
- **SpectraST** (Used to prepare the spectral library (removing duplicates) so the 1:1 target-decoy ratio that underpins subgroup FDR estimation is in place before searching.)

## Examples

```
python ann_solo/psm_filter.py --input ann_solo_hek293_ssms.csv --fdr 0.01 --subgroup-mass-diff 0.1 --min-subgroup-size 20
```

## Evaluation signals

- Final confident SSM set is consistent with identifications independently deposited at PRIDE PXD009861.
- Every retained SSM has a subgroup with at least 20 SSMs, or is excluded (and this exclusion is recorded).
- Target SSM count exceeds decoy SSM count at the chosen FDR threshold within each subgroup.
- Estimated overall FDR across the combined level-1 and level-2 results is at or below 1%.
- For the HEK293 dataset, the resulting identification count exceeds those reported for SpectraST and MSFragger.

## Limitations

- Actual FDR may be underestimated when subgroups are too small (fewer than 20 SSMs); caution is advised and alternative approaches are still being researched.
- Accurate FDR estimation for open searches remains an open research question; the target-decoy strategy used here is not guaranteed to be fully calibrated for mass-shift-based searches.
- Requires decoy spectra appended in a 1:1 ratio to the target library (e.g. via shuffle-and-reposition) for the decoy/target ratio to be interpretable.
- The strategy was validated on specific datasets (iPRG2012, HEK293 with MassIVE-KB library version 2017/11/27) and may not transfer directly to other library compositions or instrument types.

## Evidence

- [methods] Subgroup FDR by charge-normalized mass difference: "they are grouped based on their charge-normalized precursor mass difference between the query spectra and library spectra"
