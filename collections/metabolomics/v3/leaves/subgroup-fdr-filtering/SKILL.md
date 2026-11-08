---
name: subgroup-fdr-filtering
description: Use when filtering open-search SSMs (e.g. shifted dot product matches
  with large precursor mass shifts) where a single global FDR would be inflated or
  underpowered for modified peptides; requires target and decoy SSM counts per mass-difference
  group.
when_to_use_negative:
- Standard closed searches (single narrow precursor mass window) where a single global
  target-decoy FDR suffices and no mass-difference subgroups exist
- Subgroups that are intentionally kept smaller than ~20 SSMs — the article warns
  the actual FDR may be underestimated for too-small groups, so results there are
  unreliable
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: ANN-SoLo
  role: Implements the two-phase target-decoy FDR calculation with subgroup FDR filtering
    of level-2 SSMs by charge-normalized precursor mass difference (0.1 Da ranges,
    20-SSM minimum) after the cascade open search
  repo: https://github.com/bittremieux/ANN-SoLo
- name: Python
  role: Runtime used to implement ANN-SoLo's FDR estimation and to aggregate retained
    1% FDR identifications into results tables such as iprg2012_open_search_identifications.csv
  repo: https://github.com/bittremieux/ANN-SoLo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:bd210ace68dae103fc4ab0f18600cbd8ca7c6d22003011fe766b061266a2c8a1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/ANN-SoLo
  tools:
  - ANN-SoLo
  - Python
  - SpectraST
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# subgroup-fdr-filtering

## Summary

Subgroup FDR filtering for open modification spectral library searches: target-decoy FDR is estimated separately for SSMs grouped by charge-normalized precursor mass difference, so that modified-peptide matches at large mass shifts are not dominated by the unmodified-peptide FDR.

## When to use

Use when filtering open-search SSMs (e.g. shifted dot product matches with large precursor mass shifts) where a single global FDR would be inflated or underpowered for modified peptides; requires target and decoy SSM counts per mass-difference group.

## When NOT to use

- Standard closed searches (single narrow precursor mass window) where a single global target-decoy FDR suffices and no mass-difference subgroups exist
- Subgroups that are intentionally kept smaller than ~20 SSMs — the article warns the actual FDR may be underestimated for too-small groups, so results there are unreliable

## Inputs

- Target and decoy spectrum-spectrum matches (SSMs) from ANN-SoLo cascade level-2 open search with charge-normalized precursor mass differences
- SSMs from cascade level 1 (standard 20 ppm search) for the first FDR phase
- Search parameters record (0.1 Da subgroup ranges, 1% FDR threshold, minimum 20 SSMs per subgroup)

## Outputs

- List of SSMs retained at 1% subgroup FDR (confidently identified spectra)
- Per-subgroup (0.1 Da mass-difference range) FDR estimates and residual-group pooled FDR
- Identification counts table, e.g. iprg2012_open_search_identifications.csv

## How to apply

After both cascade levels of the ANN-SoLo search (level 1 at 20 ppm precursor mass window, level 2 at 300 Da window with shifted dot product scoring and 0.02 Da fragment mass tolerance), calculate FDRs using the target-decoy strategy in two phases. Group level-2 SSMs by the charge-normalized precursor mass difference between query and library spectra in 0.1 Da ranges, applying the subgroup FDR strategy so modified matches are evaluated against decoys within their own mass-shift regime. Only subgroups containing at least 20 SSMs are FDR-filtered individually; smaller subgroups are pooled into a residual group before filtering. Retain SSMs passing the 1% FDR threshold and count confident identifications (e.g. against the ~25% open-search gain over a standard 20 ppm search on iPRG2012).

## Related tools

- **ANN-SoLo** (Implements the two-phase target-decoy FDR calculation with subgroup FDR filtering of level-2 SSMs by charge-normalized precursor mass difference (0.1 Da ranges, 20-SSM minimum) after the cascade open search) — https://github.com/bittremieux/ANN-SoLo
- **Python** (Runtime used to implement ANN-SoLo's FDR estimation and to aggregate retained 1% FDR identifications into results tables such as iprg2012_open_search_identifications.csv) — https://github.com/bittremieux/ANN-SoLo

## Evaluation signals

- Every retained SSM passes the 1% FDR threshold within its 0.1 Da charge-normalized mass-difference subgroup
- All subgroups with fewer than 20 SSMs were pooled into the residual group before FDR filtering — no subgroup below the 20-SSM minimum was filtered independently
- FDR was computed after both cascade levels (level 1 and level 2), not only for the open-search level
- On iPRG2012, total 1% FDR identifications from the open search should reproduce the reported ~25% increase over a standard 20 ppm search, with additional identifications corresponding to modified peptides

## Limitations

- The actual FDR might be underestimated when too-small subgroups are used
- It is a standing research question whether alternative approaches are needed for the accurate FDR estimation of open searches
- Subgrouping applies to level-2 SSMs (open search); level-1 unmodified identifications are filtered in the first FDR phase without mass-difference subgroups

## Evidence

- [methods] Subgroup FDR strategy groups level-2 SSMs by mass difference: "they are grouped based on their charge-normalized precursor mass difference between the query spectra and library spectra"
- [methods] 20-SSM minimum per subgroup: "FDR filtering is only done for subgroups that contain at least 20 SSMs"
- [methods] Two-phase target-decoy FDR after cascade: "False discovery rates are calculated in two phases after both levels of the cascade search using the target–decoy strategy"
- [discussion] Caution on underestimated FDR for small groups: "Caution has to be observed though as the actual FDR might be underestimated when too small groups are used [27]."
- [results] Expected ~25% open-search gain on iPRG2012: "ANN-SoLo achieves a 25 % increase in identifications when performing an open search compared to a standard search"
