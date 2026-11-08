---
name: open-modification-search-result-filtering
description: Use when assigning FDR to SSMs from an open modification spectral library
  search, where modified matches must be evaluated separately from unmodified ones
  because decoy mass differences are randomly distributed while true modified matches
  cluster at shared modification masses.
when_to_use_negative:
- Standard (closed) search results or the unmodified peptide identifications from
  the first cascade-search level — these use the conventional target–decoy FDR calculation
  without subgrouping
- Subgroups containing fewer than 20 SSMs — these must be merged into the pooled residual
  group rather than filtered individually, or the FDR will be underestimated
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: ANN-SoLo
  role: Open spectral library search engine whose two-phase target–decoy FDR calculation
    over subgrouped SSMs this skill reproduces
  repo: https://github.com/bittremieux/ANN-SoLo
- name: pandas
  role: Load the SSM results table, compute the charge-normalized precursor mass difference,
    group SSMs, and write subgroup_fdr.tsv
- name: Pyteomics
  role: Alternative to pandas for loading and parsing the SSM results table
- name: NumPy
  role: Numerical computation of mass differences and subgroup assignments
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:6b0dc1b4a224121949b81eb7ac48ec0259cb6aca470e07a8166fbd6480bc0376
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# open-modification-search-result-filtering

## Summary

Estimates FDR for open-modification spectral library search results by partitioning spectrum–spectrum matches (SSMs) into subgroups based on their charge-normalized precursor mass difference and applying target–decoy FDR filtering per subgroup, as implemented in ANN-SoLo.

## When to use

Use when assigning FDR to SSMs from an open modification spectral library search, where modified matches must be evaluated separately from unmodified ones because decoy mass differences are randomly distributed while true modified matches cluster at shared modification masses.

## When NOT to use

- Standard (closed) search results or the unmodified peptide identifications from the first cascade-search level — these use the conventional target–decoy FDR calculation without subgrouping
- Subgroups containing fewer than 20 SSMs — these must be merged into the pooled residual group rather than filtered individually, or the FDR will be underestimated

## Inputs

- Open-search SSM results table containing SSM score, query precursor mass, library precursor mass, precursor charge, and target/decoy label (e.g. TSV readable with pandas/Pyteomics)

## Outputs

- Per-subgroup FDR estimates (group mass difference, target/decoy counts, FDR) written to subgroup_fdr.tsv
- SSMs filtered at the chosen FDR threshold, with modified matches evaluated within their mass-difference subgroups

## How to apply

Load the open-search SSM results table (SSM score, query and library precursor masses, precursor charge, target/decoy label) with pandas or Pyteomics. Compute each SSM's charge-normalized precursor mass difference between the query spectra and library spectra. Greedily form subgroups: repeatedly seed a subgroup with the highest-scoring remaining SSM and assign it together with all SSMs whose mass difference falls within a 0.1 Da range of the seed, until every SSM is assigned. Merge any subgroup containing fewer than 20 SSMs into a single pooled residual group, since FDR filtering is only done for subgroups that contain at least 20 SSMs. For each subgroup (and the residual group), estimate FDR as the number of decoy SSMs divided by the number of target SSMs under the concatenated target–decoy strategy, then filter SSMs at the desired FDR threshold. Note that FDR estimation happens in two phases after both levels of the cascade search — unmodified peptides are handled first by the standard search, and this subgroup-based procedure applies to the modified matches from the open search level.

## Related tools

- **ANN-SoLo** (Open spectral library search engine whose two-phase target–decoy FDR calculation over subgrouped SSMs this skill reproduces) — https://github.com/bittremieux/ANN-SoLo
- **pandas** (Load the SSM results table, compute the charge-normalized precursor mass difference, group SSMs, and write subgroup_fdr.tsv)
- **Pyteomics** (Alternative to pandas for loading and parsing the SSM results table)
- **NumPy** (Numerical computation of mass differences and subgroup assignments)

## Examples

```
import pandas as pd; df = pd.read_csv('ssms.tsv', sep='\t'); df['mass_diff'] = (df['precursor_mz_query'] - df['precursor_mz_library']) * df['precursor_charge']
```

## Evaluation signals

- Every SSM is assigned to exactly one subgroup (greedy seed-based grouping within 0.1 Da mass-difference windows is exhaustive)
- All subgroups subjected to FDR filtering contain at least 20 SSMs; smaller groups are pooled into a single residual group
- Per-subgroup FDR equals decoy SSM count divided by target SSM count, consistent with the concatenated target–decoy strategy
- Decoy mass differences are randomly distributed while true modified matches cluster at shared modification masses — grouped target SSMs should show clustering at modification masses
- Written subgroup_fdr.tsv contains group mass difference, target/decoy counts, and FDR for every subgroup including the residual group

## Limitations

- The actual FDR might be underestimated when too small groups are used
- It is a standing research question whether alternative approaches are needed for the accurate FDR estimation of open searches

## Evidence

- [methods] Subgroup FDR rationale: "they are grouped based on their charge-normalized precursor mass difference between the query spectra and library spectra"
- [methods] Minimum subgroup size: "FDR filtering is only done for subgroups that contain at least 20 SSMs"
- [methods] Two-phase target-decoy FDR: "False discovery rates are calculated in two phases after both levels of the cascade search using the target–decoy strategy"
- [discussion] Underestimation caveat: "Caution has to be observed though as the actual FDR might be underestimated when too small groups are used [27]."
- [readme] Cascade FDR control from README: "while strictly controlling the false discovery rate and the shifted dot product score to sensitively match modified spectra to their unmodified counterpart"
