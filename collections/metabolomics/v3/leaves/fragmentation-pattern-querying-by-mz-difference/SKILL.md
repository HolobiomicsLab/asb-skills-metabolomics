---
name: fragmentation-pattern-querying-by-mz-difference
description: Use when a mined mineMS2 pattern set must be searched for a specific
  m/z difference or loss (e.g. 186.079 Da) to recover spectra sharing that fragmentation,
  including similarities missed by GNPS network edges.
when_to_use_negative:
- Input is raw MS/MS spectra rather than an already-mined pattern set — de novo frequent-subgraph
  mining of patterns is the separate upstream mineMS2 step
- The goal is to mine all frequent fragmentation patterns or to select patterns explaining
  GNPS components in bulk, rather than to search for one specific m/z difference
- The queried difference should not be restricted to a precursor loss (the 'P' refinement
  would wrongly discard patterns where the difference occurs between ions of distinct
  fragmentation paths)
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: mineMS2
  role: R package whose mined pattern set is queried with findMz (ppm=15, dmz=0.01,
    type='L' then 'P') to retrieve patterns containing the target m/z difference and
    their spectrum occurrences
  repo: https://github.com/odisce/mineMS2
- name: GNPS
  role: Reference molecular network used to determine which spectra containing the
    queried pattern are not linked by network edges
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-025-01051-y
    title: 'mineMS2: annotation of spectral libraries with exact fragmentation patterns'
  source_content_hash: sha256:2f8c177d8036a6583d8329f41a854d16f12072745f33db4b20fb9f270d47afe2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/odisce/mineMS2
  tools:
  - GNPS
  - R
  - mineMS2
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# fragmentation-pattern-querying-by-mz-difference

## Summary

Query an already-mined mineMS2 pattern set for a specific m/z difference (e.g. the 186.079 Da putative tryptophan loss) using findMz, refine to true precursor losses, and extract per-spectrum occurrences to reveal spectral similarities not captured by the GNPS molecular network.

## When to use

Use when a mined mineMS2 pattern set must be searched for a specific m/z difference or loss (e.g. 186.079 Da) to recover spectra sharing that fragmentation, including similarities missed by GNPS network edges.

## When NOT to use

- Input is raw MS/MS spectra rather than an already-mined pattern set — de novo frequent-subgraph mining of patterns is the separate upstream mineMS2 step
- The goal is to mine all frequent fragmentation patterns or to select patterns explaining GNPS components in bulk, rather than to search for one specific m/z difference
- The queried difference should not be restricted to a precursor loss (the 'P' refinement would wrongly discard patterns where the difference occurs between ions of distinct fragmentation paths)

## Inputs

- Mined mineMS2 pattern set (fragmentation-graph patterns of m/z differences) produced from the LIMS-DB dataset
- GNPS molecular network edge list for cross-checking which spectra sharing the pattern are not linked
- Target m/z difference value (e.g. 186.079)

## Outputs

- Filtered list of mineMS2 patterns containing the queried m/z difference as a loss from the precursor
- Per-spectrum occurrence table for the selected pattern(s), e.g. mineMS2_pattern_186.079_occurrences.csv

## How to apply

Load the mined pattern set produced from the LIMS-DB dataset (622 MS/MS spectra from pure authentic standards) with the mineMS2 R package. Query all patterns containing the target m/z difference using the findMz function with parameters ppm=15 (the tolerance selected in the paper because lower values were too sensitive to small m/z variations), dmz=0.01 and type='L' (local search for the difference anywhere in the pattern graph). Refine the query by selecting on 'P' so that only patterns in which the m/z difference occurs as a loss from the precursor are kept — mineMS2 m/z differences are edges between ion-peak nodes and include neutral losses as well as differences between ions of distinct fragmentation paths. Extract the occurrences of the filtered patterns, identifying spectra carrying the pattern (e.g. the tetrapeptide pattern shared by 7 spectra, 4 of which are unlinked in the GNPS molecular network). Write the filtered pattern list and per-spectrum occurrence table to a CSV output.

## Related tools

- **mineMS2** (R package whose mined pattern set is queried with findMz (ppm=15, dmz=0.01, type='L' then 'P') to retrieve patterns containing the target m/z difference and their spectrum occurrences) — https://github.com/odisce/mineMS2
- **GNPS** (Reference molecular network used to determine which spectra containing the queried pattern are not linked by network edges)

## Examples

```
hits <- findMz(minedPatterns, mz = 186.079, ppm = 15, dmz = 0.01, type = "L")  # then subset type == "P" for precursor-loss patterns
```

## Evaluation signals

- Output contains the tetrapeptide pattern with the 186.079 loss supported by 7 spectra
- 4 of those 7 spectra are confirmed as not linked in the GNPS molecular network
- Query used type='L' with ppm=15 and dmz=0.01, followed by the 'P' selection restricting the difference to a loss from the precursor
- Result table distinguishes the local ('L') hits from the precursor-loss ('P') subset
- Output CSV (e.g. mineMS2_pattern_186.079_occurrences.csv) lists the filtered patterns and per-spectrum occurrences

## Limitations

- Exact pattern mining methods such as mineMS2 extract a high number of patterns, and no scoring system for patterns yet exists
- There is no database of patterns of interest with a dedicated query metric; strategies to facilitate structural interpretation of patterns are planned future work
- A subset of spectrum peaks must be selected due to combinatorial complexity, currently based solely on peak intensities, which may lose chemically relevant information

## Evidence

- [readme] Each pattern is a graph with ion peaks as nodes and m/z differences as edges: "Each **pattern is a graph** with **ion peaks as nodes** and **m/z differences as edges**. These m/z differences can be **any difference between the m/z values of two peaks"
- [intro] GNPS networks cannot extract common structures shared by more than two spectra: "it does not, however, enable to extract common structures between more than two spectra"
- [results] 15 ppm tolerance selected for m/z difference binning: "We selected a tolerance of 15 ppm because a lower value produced patterns which were too sensitive to very small variations in m/z"
- [abstract] mineMS2 available as an R package: "The mineMS2 software is publicly available as an R package"
