---
name: molecular-network-vs-pattern-occurrence-comparison
description: Use when MS/MS spectra suspected to share a structural motif (e.g. a
  common neutral loss such as tryptophan loss 186.079) are NOT all connected in a
  GNPS molecular network and you need a graph-pattern-based similarity check.
when_to_use_negative:
- The spectra of interest are already fully connected in the GNPS molecular network,
  so the network itself already exposes the similarity
- You need probabilistic fragment/loss distributions rather than exact graph-structured
  patterns — MS2LDA is the complementary approach
- The input is raw MS/MS spectra rather than an already-mined pattern set; pattern
  mining must be run first (a separate skill)
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mineMS2
  role: R package used to query mined patterns (findMz with ppm=15, dmz=0.01, type='L'
    and 'P' selection) and extract per-spectrum occurrences of the m/z difference
  repo: https://github.com/odisce/mineMS2
- name: GNPS
  role: Provides the molecular network whose edges (modified cosine score) are compared
    against pattern occurrence co-membership to identify unlinked spectra
- name: R
  role: Environment in which the mineMS2 pattern query, occurrence extraction, and
    CSV export are executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-025-01051-y
    title: 'mineMS2: annotation of spectral libraries with exact fragmentation patterns'
  source_content_hash: sha256:6df5b7bc5581fda74e32a1cb7b70f1ea4b40b7afc50d5cff62caf8b886885365
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/odisce/mineMS2
  tools:
  - GNPS
  - R
  - mineMS2
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# molecular-network-vs-pattern-occurrence-comparison

## Summary

Compares mineMS2 pattern occurrence sets against GNPS molecular network connectivity to reveal spectral similarities that network edges (modified cosine score) miss, e.g. a 186.079 Da tryptophan-loss pattern shared by tetrapeptide spectra only partially linked in the network.

## When to use

Use when MS/MS spectra suspected to share a structural motif (e.g. a common neutral loss such as tryptophan loss 186.079) are NOT all connected in a GNPS molecular network and you need a graph-pattern-based similarity check.

## When NOT to use

- The spectra of interest are already fully connected in the GNPS molecular network, so the network itself already exposes the similarity
- You need probabilistic fragment/loss distributions rather than exact graph-structured patterns — MS2LDA is the complementary approach
- The input is raw MS/MS spectra rather than an already-mined pattern set; pattern mining must be run first (a separate skill)

## Inputs

- Mined pattern set from mineMS2 (e.g. output of task_002 on the LIMS-DB dataset of 622 MS/MS spectra)
- GNPS molecular network (spectrum nodes and edges) for the same spectra
- Target m/z difference value (e.g. 186.079, putative tryptophan loss)

## Outputs

- Filtered pattern list containing the queried m/z difference
- Per-spectrum occurrence table for each filtered pattern
- CSV file such as mineMS2_pattern_186.079_occurrences.csv
- Count of pattern-sharing spectra that are unlinked in the GNPS network

## How to apply

Load the mined pattern set previously produced from the dataset of interest (e.g. the LIMS-DB 622-spectrum collection mined with mineMS2). Query all patterns containing the target m/z difference using the findMz function with ppm=15 and dmz=0.01 and type='L' (the 15 ppm tolerance was selected because lower values made patterns too sensitive to small m/z variations). Refine the query with a selection on 'P' to keep only patterns in which the m/z difference occurs as a loss from the precursor. Extract the occurrences of the filtered patterns, tabulate which spectra contain each pattern, and cross-reference the spectra against the GNPS network edges to count how many pattern-sharing spectra are unlinked in the network. In the reported case, the 186.079 pattern was found in 7 tryptophan-containing tetrapeptide spectra, 4 of which are not linked in the GNPS network, demonstrating that exact graph-structured patterns capture similarities that pairwise cosine-based networking does not.

## Related tools

- **mineMS2** (R package used to query mined patterns (findMz with ppm=15, dmz=0.01, type='L' and 'P' selection) and extract per-spectrum occurrences of the m/z difference) — https://github.com/odisce/mineMS2
- **GNPS** (Provides the molecular network whose edges (modified cosine score) are compared against pattern occurrence co-membership to identify unlinked spectra)
- **R** (Environment in which the mineMS2 pattern query, occurrence extraction, and CSV export are executed)

## Examples

```
findMz(patterns, mz = 186.079, ppm = 15, dmz = 0.01, type = 'L')
```

## Evaluation signals

- The filtered pattern list contains a pattern supported by exactly 7 spectra, annotated as tryptophan-containing tetrapeptides
- Of those 7 spectra, 4 are not linked in the GNPS molecular network — the unlinked count must be reproducible
- All retained patterns contain 186.079 as a loss from the precursor (type='L' query followed by 'P' selection), not merely as a difference between fragment ions
- The output CSV (e.g. mineMS2_pattern_186.079_occurrences.csv) contains both the filtered pattern list and the per-spectrum occurrence table
- The ppm/dmz parameters used are 15 ppm and dmz=0.01, matching the selected mineMS2 binning tolerance

## Limitations

- Exact pattern mining extracts a high number of patterns, so a targeted query on a specific m/z difference is needed to stay tractable
- mineMS2 requires selecting a subset of spectrum peaks (currently based solely on peak intensities), which may lose chemically relevant information
- No scoring system for patterns or database of patterns of interest with a dedicated query metric yet exists
- GNPS molecular networks propagate annotations between pairs but cannot extract common structures shared by more than two spectra, which is precisely the gap this comparison exploits

## Evidence

- [other] Querying patterns containing the 186.079 m/z difference recovers a pattern included in 7 spectra, not all linked in the GNPS network, identified as tetrapeptides containing a tryptophan residue.: [evidence span withheld — anchor status: not_found]
- [intro] In molecular networks, as generated by the GNPS platform, each vertex is a spectrum and an edge indicates a high pairwise similarity according to the modified cosine score: "In molecular networks, as generated by the GNPS platform [27], each vertex is a spectrum and an edge indicates a high pairwise similarity according to the modified cosine score"
- [intro] it does not, however, enable to extract common structures between more than two spectra: "it does not, however, enable to extract common structures between more than two spectra"
- [intro] the specific nature of the detected patterns (as being exact and structured as graphs) enables to capture similarities that are not detected by existing methods: "the specific nature of the detected patterns (as being exact and structured as graphs) enables to capture similarities that are not detected by existing methods"
- [results] We selected a tolerance of 15 ppm because a lower value produced patterns which were too sensitive to very small variations in m/z: "We selected a tolerance of 15 ppm because a lower value produced patterns which were too sensitive to very small variations in m/z"
