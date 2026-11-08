---
name: target-decoy-fdr-estimation
description: Use when filtering SSMs from a library search (standard or open/modified)
  to a confident set at 1% FDR, especially when modified matches must be evaluated
  separately from unmodified ones via target-decoy decoy-to-target ratios.
when_to_use_negative:
- When only unmodified/standard-search SSMs are being analyzed, the charge-normalized
  mass-difference subgrouping is unnecessary — a single pooled target-decoy FDR estimate
  suffices.
- When subgroups are too small (fewer than 20 SSMs even after pooling), FDR may be
  underestimated and results should be interpreted with caution rather than treated
  as reliable.
- When using a search engine that does not append decoy spectra (1:1 shuffle-and-reposition),
  the decoy/target ratio cannot be computed and this skill does not apply.
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ANN-SoLo
  role: Search engine whose two-level cascade output SSMs are FDR-estimated and filtered
    at 1% using the target-decoy and subgroup FDR strategies
  repo: https://github.com/bittremieux/ANN-SoLo
- name: Python
  role: Implementation language for subgroup FDR computation (pandas/NumPy/Pyteomics)
    and writing per-subgroup FDR tables
- name: SpectraST
  role: Used to prepare the target-decoy library by removing duplicates and appending
    1:1 shuffle-and-reposition decoy spectra prior to FDR-based filtering
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  - task_007
  - task_008
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:067019055f86cf64b6aa3f70b71d7cdd06ddce965ce660270ba315c6abd62cd2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/ANN-SoLo
  tools:
  - ANN-SoLo
  - MS-GF+
  - NumPy
  - ProteoWizard
  - Pyteomics
  - Python
  - SpectraST
  - msconvert
  - pandas
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# target-decoy-fdr-estimation

## Summary

Estimate and control the false discovery rate of spectrum–spectrum matches (SSMs) from spectral library searches — including open modification searches — using the target–decoy strategy, with subgroup FDR estimation for open-search matches grouped by charge-normalized precursor mass difference.

## When to use

Use when filtering SSMs from a library search (standard or open/modified) to a confident set at 1% FDR, especially when modified matches must be evaluated separately from unmodified ones via target-decoy decoy-to-target ratios.

## When NOT to use

- When only unmodified/standard-search SSMs are being analyzed, the charge-normalized mass-difference subgrouping is unnecessary — a single pooled target-decoy FDR estimate suffices.
- When subgroups are too small (fewer than 20 SSMs even after pooling), FDR may be underestimated and results should be interpreted with caution rather than treated as reliable.
- When using a search engine that does not append decoy spectra (1:1 shuffle-and-reposition), the decoy/target ratio cannot be computed and this skill does not apply.

## Inputs

- Open-search SSM results table with SSM score, query and library precursor masses, precursor charge, and target/decoy label (TSV/CSV loaded with pandas/Pyteomics)
- Standard-search SSM results with target-decoy labels from cascade level 1 (precursor mass window search)
- Target-decoy spectral library with decoys appended 1:1 via shuffle-and-reposition (e.g. MassIVE-KB 2017/11/27, or NIST human HCD 2016/09/12 concatenated with the Selevsek yeast library)

## Outputs

- Confident SSM set filtered at 1% FDR (per cascade level and combined)
- Per-subgroup FDR table (group mass difference, target/decoy counts, FDR) written to subgroup_fdr.tsv
- Identification counts for downstream comparison (e.g. ann_solo_hek293_idents.csv, iprg2012_open_search_identifications.csv)

## How to apply

After both levels of a cascade search (e.g. ANN-SoLo's 5–20 ppm level 1 followed by the 300–500 Da open-search level 2), calculate FDRs using the target–decoy strategy, with decoy spectra appended to the library in a 1:1 ratio via the shuffle-and-reposition method. For standard-search SSMs, estimate FDR as the number of decoy SSMs divided by the number of target SSMs and filter at the chosen threshold (1% FDR in the ANN-SoLo workflows). For open-search SSMs, first compute each match's charge-normalized precursor mass difference between query and library spectra, then greedily partition SSMs into subgroups: repeatedly seed a subgroup with the highest-scoring remaining SSM and include all SSMs within a 0.1 Da range of the seed, until every SSM is assigned. Apply FDR filtering per subgroup, but only for subgroups containing at least 20 SSMs, merging smaller subgroups into a single pooled residual group — the rationale being that decoy mass differences are randomly distributed while true modified matches cluster at shared modification masses. Retain confident SSMs below the FDR threshold from both cascade levels and export the counts for downstream comparison (e.g. against the deposited PXD009861 results).

## Related tools

- **ANN-SoLo** (Search engine whose two-level cascade output SSMs are FDR-estimated and filtered at 1% using the target-decoy and subgroup FDR strategies) — https://github.com/bittremieux/ANN-SoLo
- **Python** (Implementation language for subgroup FDR computation (pandas/NumPy/Pyteomics) and writing per-subgroup FDR tables)
- **SpectraST** (Used to prepare the target-decoy library by removing duplicates and appending 1:1 shuffle-and-reposition decoy spectra prior to FDR-based filtering)

## Examples

```
import pandas as pd; df = pd.read_csv('ssm_results.tsv', sep='\t'); df['mass_diff'] = (df['query_mass'] - df['library_mass']) / df['charge']; df.to_csv('subgroup_fdr.tsv', sep='\t', index=False)
```

## Evaluation signals

- Every retained SSM passes the 1% FDR threshold, and FDR is computed as decoy SSMs divided by target SSMs within each subgroup.
- All SSMs are assigned to a subgroup: seeds are selected greedily by score and subgroups span 0.1 Da of charge-normalized precursor mass difference; no SSM is left ungrouped.
- Subgroups with fewer than 20 SSMs are merged into a single residual group before FDR filtering; FDR is never filtered on sub-sub-20-Spec groups individually.
- Open-search identification counts at 1% FDR are consistent with the deposited results at PXD009861 (e.g. ~25% increase over a standard 20 ppm search on iPRG2012, 45% total increase with the shifted dot product cascade).
- FDRs are calculated after both levels of the cascade search, and confident SSMs from both levels are combined only after filtering.

## Limitations

- The actual FDR might be underestimated when too-small groups are used, so caution must be observed with small subgroups.
- Accurate FDR estimation for open modification searches remains a standing research question; alternative approaches may be needed.
- FDR estimation for open-search level 2 requires the subgroup strategy; applying a single global FDR across all mass differences can mis-control the rate because decoy mass differences are randomly distributed while true modified matches cluster at shared modification masses.

## Evidence

- [methods] False discovery rates are calculated in two phases after both levels of the cascade search using the target–decoy strategy: "False discovery rates are calculated in two phases after both levels of the cascade search using the target–decoy strategy"
- [methods] decoy spectra were added in a 1:1 ratio using the shuffle-and-reposition method: "decoy spectra were added in a 1:1 ratio using the shuffle-and-reposition method [41]"
- [methods] they are grouped based on their charge-normalized precursor mass difference between the query spectra and library spectra: "they are grouped based on their charge-normalized precursor mass difference between the query spectra and library spectra"
- [methods] FDR filtering is only done for subgroups that contain at least 20 SSMs: "FDR filtering is only done for subgroups that contain at least 20 SSMs"
- [discussion] Caution has to be observed though as the actual FDR might be underestimated when too small groups are used: "Caution has to be observed though as the actual FDR might be underestimated when too small groups are used [27]."
- [readme] decoy mass differences are randomly distributed while true modified matches cluster at shared modification masses: "while strictly controlling the false discovery rate and the shifted dot product score to sensitively match modified spectra to their unmodified counterpart"
