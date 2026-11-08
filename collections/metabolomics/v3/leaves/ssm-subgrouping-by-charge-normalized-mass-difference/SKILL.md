---
name: ssm-subgrouping-by-charge-normalized-mass-difference
description: 'Use when FDR must be estimated for open modification search SSMs whose
  mass shifts span many values: group matches by charge-normalized mass difference,
  estimate decoy/target FDR per group, and merge groups with fewer than 20 SSMs.'
when_to_use_negative:
- Standard (closed) searches with a single or small predefined set of mass shifts
  — FDR can be estimated directly on the full SSM set without subgrouping
- Subgroups are needed with fewer than 20 SSMs and cannot be pooled — the paper warns
  the actual FDR may be underestimated when too-small groups are used
- Percolator-style semisupervised rescoring is the goal rather than target–decoy per-subgroup
  FDR estimation
tools:
- name: ANN-SoLo
  role: Source of the open-search SSM results (score, precursor masses, charge, target/decoy
    label) that are subgrouped and FDR-estimated
  repo: https://github.com/bittremieux/ANN-SoLo
- name: pandas
  role: Load the SSM table, compute charge-normalized mass differences, and write
    subgroup_fdr.tsv
- name: Pyteomics
  role: Parsing and handling of the SSM results table with target/decoy labels
- name: NumPy
  role: Vectorized computation of charge-normalized mass differences and subgroup
    membership windows
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:8d6634b2fe8be3b34b51870186f80ed10912908d67504398cce619c5c47b0eaa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/ANN-SoLo
  tools:
  - ANN-SoLo
  - NumPy
  - Pyteomics
  - Python
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9047619047619048
n_claims: 42
---

# ssm-subgrouping-by-charge-normalized-mass-difference

## Summary

Partitions open-search spectrum–spectrum matches (SSMs) into subgroups by charge-normalized precursor mass difference between query and library spectra so that target–decoy FDR can be estimated per subgroup, exploiting the fact that true modified matches cluster at shared modification masses while decoy mass differences are randomly distributed.

## When to use

Use when FDR must be estimated for open modification search SSMs whose mass shifts span many values: group matches by charge-normalized mass difference, estimate decoy/target FDR per group, and merge groups with fewer than 20 SSMs.

## When NOT to use

- Standard (closed) searches with a single or small predefined set of mass shifts — FDR can be estimated directly on the full SSM set without subgrouping
- Subgroups are needed with fewer than 20 SSMs and cannot be pooled — the paper warns the actual FDR may be underestimated when too-small groups are used
- Percolator-style semisupervised rescoring is the goal rather than target–decoy per-subgroup FDR estimation

## Inputs

- Open-search SSM results table containing SSM score, query precursor mass, library precursor mass, precursor charge, and concatenated target–decoy label per match (e.g., TSV produced by ANN-SoLo open search)

## Outputs

- Per-subgroup FDR estimates written to subgroup_fdr.tsv, with columns for group mass difference, target and decoy SSM counts, and FDR
- Subgroup assignment of every SSM (mass-difference subgroups plus a pooled residual group for subgroups with fewer than 20 SSMs)

## How to apply

Load the open-search SSM results (SSM score, query and library precursor masses, precursor charge, target/decoy label) with pandas/Pyteomics, then compute each SSM's charge-normalized precursor mass difference between the query and library spectra. Greedily form subgroups: repeatedly seed a subgroup with the highest-scoring unassigned SSM and assign all remaining SSMs whose mass difference falls within a 0.1 Da window of the seed, until every SSM is assigned. Merge any subgroup with fewer than 20 SSMs into a single pooled residual group, since FDR filtering is only done for subgroups containing at least 20 SSMs. For each subgroup (and the residual group), estimate FDR from concatenated target–decoy labels as decoy SSM count divided by target SSM count. The grouping rationale is that decoy mass differences are randomly distributed while true modified matches cluster at shared modification masses, so per-subgroup decoy enrichment reflects false matches within each mass-shift class.

## Related tools

- **ANN-SoLo** (Source of the open-search SSM results (score, precursor masses, charge, target/decoy label) that are subgrouped and FDR-estimated) — https://github.com/bittremieux/ANN-SoLo
- **pandas** (Load the SSM table, compute charge-normalized mass differences, and write subgroup_fdr.tsv)
- **Pyteomics** (Parsing and handling of the SSM results table with target/decoy labels)
- **NumPy** (Vectorized computation of charge-normalized mass differences and subgroup membership windows)

## Examples

```
import pandas as pd; df = pd.read_csv('ssms.tsv', sep='\t'); df['mass_diff'] = (df['precursor_mass_query'] - df['precursor_mass_library']) / df['charge']
```

## Evaluation signals

- Every SSM is assigned to exactly one subgroup after the greedy 0.1 Da mass-difference window grouping (no matches left unassigned)
- All subgroups retained for FDR filtering contain at least 20 SSMs; every subgroup below this size is merged into a single residual group
- Each subgroup's reported FDR equals its decoy SSM count divided by its target SSM count from the concatenated target–decoy labels
- The mass difference for each subgroup is charge-normalized, i.e., computed from (query precursor mass − library precursor mass) divided by precursor charge
- subgroup_fdr.tsv contains, per row, the group mass difference, target/decoy counts, and FDR, and FDR values fall within [0, 1] for well-populated subgroups

## Limitations

- The actual FDR might be underestimated when too-small subgroups are used; hence the 20-SSM minimum and residual-group pooling
- Accurate FDR estimation for open searches is a standing research question — alternative approaches may be needed
- Subgrouping is designed for the open-search regime where mass shifts are unbounded; it presumes true modified matches cluster at shared modification masses, which may not hold for very sparse or heterogeneous modification landscapes

## Evidence

- [methods] Subgrouping basis: "they are grouped based on their charge-normalized precursor mass difference between the query spectra and library spectra"
- [methods] Minimum subgroup size for FDR filtering: "FDR filtering is only done for subgroups that contain at least 20 SSMs"
- [methods] Target–decoy FDR framework: "False discovery rates are calculated in two phases after both levels of the cascade search using the target–decoy strategy"
- [discussion] Small-group FDR bias caveat: "Caution has to be observed though as the actual FDR might be underestimated when too small groups are used [27]."
- [readme] FDR control in cascade open search (README): "This is combined with a cascade search strategy to maximize the number of identified unmodified and modified spectra while strictly controlling the false discovery rate"
