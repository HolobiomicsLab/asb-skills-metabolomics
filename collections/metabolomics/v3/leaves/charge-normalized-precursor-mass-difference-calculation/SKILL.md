---
name: charge-normalized-precursor-mass-difference-calculation
description: Use when analyzing open-search SSM results where modified peptide matches
  must be grouped by their precursor mass shift to enable per-group target–decoy FDR
  estimation, especially when deciding whether each subgroup is large enough (≥20
  SSMs) to support separate FDR filtering.
when_to_use_negative:
- When FDR is already available for a standard (closed) search without modification
  mass shifts, since grouping by mass difference only helps when true modified matches
  cluster at shared modification masses.
- When individual subgroups would be very small and the total set of open-search SSMs
  lacks enough target/decoy pairs to meet the ≥20 SSM threshold, as the actual FDR
  might be underestimated when too small groups are used.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ANN-SoLo
  role: Open modification spectral library search engine whose SSM output (query/library
    precursor masses, charge, target/decoy label) supplies the input to the charge-normalized
    mass difference calculation and subsequent subgroup FDR estimation.
  repo: https://github.com/bittremieux/ANN-SoLo
- name: pandas
  role: Python library used to load the SSM table, compute the mass differences, form
    subgroups, and write the per-subgroup FDR estimates to subgroup_fdr.tsv.
- name: Pyteomics
  role: Python library used to read the SSM results table and its mass/charge fields.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:d3cdcefecde385803ee6075f838075e75d38b31dadabf0576d041286e3e00f0e
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
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# charge-normalized-precursor-mass-difference-calculation

## Summary

Computes the charge-normalized precursor mass difference between query and library spectra for each spectrum–spectrum match (SSM), then greedily forms subgroups of SSMs sharing mass differences within a 0.1 Da window so that false discovery rates can be estimated separately for modified peptide matches.

## When to use

Use when analyzing open-search SSM results where modified peptide matches must be grouped by their precursor mass shift to enable per-group target–decoy FDR estimation, especially when deciding whether each subgroup is large enough (≥20 SSMs) to support separate FDR filtering.

## When NOT to use

- When FDR is already available for a standard (closed) search without modification mass shifts, since grouping by mass difference only helps when true modified matches cluster at shared modification masses.
- When individual subgroups would be very small and the total set of open-search SSMs lacks enough target/decoy pairs to meet the ≥20 SSM threshold, as the actual FDR might be underestimated when too small groups are used.

## Inputs

- open-search SSM results table with columns for SSM score, query precursor mass, library precursor mass, precursor charge, and target/decoy label
- mass spectra searched with the open modification spectral library search engine (e.g. ANN-SoLo output on iPRG2012 or HEK293 data)

## Outputs

- subgroup_fdr.tsv listing per-subgroup mass difference, target and decoy SSM counts, and estimated FDR
- subgroup assignments mapping each SSM to its mass-difference-based group

## How to apply

Load the open-search SSM results table (SSM score, query and library precursor masses, precursor charge, target/decoy label) with pandas or Pyteomics, then compute each SSM's charge-normalized precursor mass difference between the query and library spectra. Greedily form subgroups by repeatedly selecting the highest-scoring remaining SSM as a seed and collecting all SSMs whose mass difference falls within 0.1 Da of that seed, until every SSM is assigned. Merge any subgroup containing fewer than 20 SSMs into a residual group, because FDR filtering is only applied to subgroups with at least 20 SSMs. For each subgroup (and the pooled residual group), estimate the FDR as the number of decoy SSMs divided by the number of target SSMs under the concatenated target–decoy strategy, and write the per-group mass difference, target/decoy counts, and FDR to subgroup_fdr.tsv. The rationale is that decoy mass differences are randomly distributed while true modified matches cluster at shared modification masses, so grouping by mass difference concentrates true matches and makes decoy/target ratios informative per group.

## Related tools

- **ANN-SoLo** (Open modification spectral library search engine whose SSM output (query/library precursor masses, charge, target/decoy label) supplies the input to the charge-normalized mass difference calculation and subsequent subgroup FDR estimation.) — https://github.com/bittremieux/ANN-SoLo
- **pandas** (Python library used to load the SSM table, compute the mass differences, form subgroups, and write the per-subgroup FDR estimates to subgroup_fdr.tsv.)
- **Pyteomics** (Python library used to read the SSM results table and its mass/charge fields.)

## Examples

```
import pandas as pd; df = pd.read_csv('open_search_ssms.tsv', sep='\t'); df['mdiff'] = (df['query_mass'] - df['library_mass']) / df['charge']; pd.to_csv('subgroup_fdr.tsv', sep='\t')
```

## Evaluation signals

- Every SSM in the output subgroup_fdr.tsv has a single assigned subgroup, with no SSMs left unassigned after greedy seed selection and 0.1 Da window assignment.
- All reported subgroups, except the pooled residual group, contain at least 20 SSMs, matching the threshold at which FDR filtering is applied.
- Per-subgroup decoy counts divided by target counts reproduce the reported FDR values, consistent with the concatenated target–decoy strategy.
- The charge-normalized precursor mass difference for each SSM falls within 0.1 Da of its subgroup seed, and mass differences for decoy SSMs should appear randomly distributed across subgroups rather than clustered.

## Limitations

- FDR filtering is only applied to subgroups containing at least 20 SSMs, so very small groups are merged into a residual pool and do not receive their own FDR estimate.
- The actual FDR may be underestimated when too-small subgroups are used.
- It is a standing research question whether alternative approaches are needed for accurate FDR estimation of open searches.

## Evidence

- [methods] For FDR estimation in the open search, SSMs are partitioned into subgroups based on their charge-normalized precursor mass difference between query and library spectra: "they are grouped based on their charge-normalized precursor mass difference between the query spectra and library spectra"
- [methods] Subgroup FDR filtering threshold of at least 20 SSMs per group: "FDR filtering is only done for subgroups that contain at least 20 SSMs"
- [discussion] Caution about FDR underestimation in small subgroups: "Caution has to be observed though as the actual FDR might be underestimated when too small groups are used [27]."
- [discussion] Unresolved question of accurate FDR estimation for open searches: "It is a standing research question whether alternative approaches are needed for the accurate FDR estimation of open searches [20, 40]."
- [readme] Open-source code availability of ANN-SoLo: "The software is available as open-source under the Apache 2.0 license."
