---
name: peptide-structural-annotation-from-fragmentation
description: Use when MS/MS spectra suspected of sharing a structural motif (e.g.
  a residue-specific neutral loss) are NOT linked in a GNPS molecular network and
  you need exact m/z-difference patterns to annotate them.
when_to_use_negative:
- The spectra of interest are already connected in the GNPS molecular network by modified
  cosine edges — pairwise network similarity suffices and pattern mining adds no linking
  information
- You need to mine patterns de novo from raw MGF/MZML spectra rather than query an
  existing mined pattern set — use the full mineMS2 pattern-mining workflow (discretizeMzDifferences,
  graph building, frequent-subgraph mining) instead
- Only a single spectrum is available — mineMS2 patterns require the m/z differences
  to be frequent, i.e. detected in at least two spectra
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3172
tools:
- name: mineMS2
  role: R package used to query the mined pattern set via findMz (ppm=15, dmz=0.01,
    type='L') and extract per-spectrum occurrences of patterns containing the m/z
    difference
  repo: https://github.com/odisce/mineMS2
- name: R
  role: Execution environment in which the mineMS2 package is wrapped (via Rcpp) and
    the querying/occurrence-extraction script is run
  repo: https://github.com/odisce/mineMS2
- name: GNPS
  role: Molecular networking platform whose modified cosine edges are used as the
    reference linkage against which unlinked spectra recovered by the pattern are
    identified
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-025-01051-y
    title: 'mineMS2: annotation of spectral libraries with exact fragmentation patterns'
  source_content_hash: sha256:ec7badf00ee99a51744ca837d3116782e718b02b58ed7fcc57a1f60fc8a04823
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/odisce/mineMS2
  tools:
  - GNPS
  - R
  - mineMS2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# peptide-structural-annotation-from-fragmentation

## Summary

This skill uses exact mineMS2 fragmentation patterns queried by a residue-specific m/z loss (e.g. 186.079, putative tryptophan loss) to structurally annotate MS/MS spectra and recover spectral similarities that GNPS molecular networking (modified cosine edges) misses, e.g. grouping tryptophan-containing tetrapeptides.

## When to use

Use when MS/MS spectra suspected of sharing a structural motif (e.g. a residue-specific neutral loss) are NOT linked in a GNPS molecular network and you need exact m/z-difference patterns to annotate them.

## When NOT to use

- The spectra of interest are already connected in the GNPS molecular network by modified cosine edges — pairwise network similarity suffices and pattern mining adds no linking information
- You need to mine patterns de novo from raw MGF/MZML spectra rather than query an existing mined pattern set — use the full mineMS2 pattern-mining workflow (discretizeMzDifferences, graph building, frequent-subgraph mining) instead
- Only a single spectrum is available — mineMS2 patterns require the m/z differences to be frequent, i.e. detected in at least two spectra

## Inputs

- Mined mineMS2 pattern set produced from the LIMS-DB dataset (output of the frequent-subgraph mining workflow)
- Target m/z difference to query (e.g. 186.079, putative tryptophan loss)
- GNPS molecular network edge table for the same spectra collection
- Spectrum annotations/metadata used to assign chemical identities (tetrapeptides containing a tryptophan residue)

## Outputs

- Filtered pattern list containing all patterns with 186.079 as a loss from the precursor
- Per-spectrum occurrence table written to mineMS2_pattern_186.079_occurrences.csv
- Structural annotation: the pattern shared by 7 spectra identified as tetrapeptides containing a tryptophan residue, 4 of which are unlinked in the GNPS network

## How to apply

Load the mined pattern set previously produced by mineMS2 from the MS/MS collection (here the LIMS-DB dataset of 622 spectra from pure authentic standards). Query all patterns containing the target m/z difference (186.079) with the findMz function using ppm=15 (the tolerance selected in the article because lower values made patterns too sensitive to small m/z variations), dmz=0.01, and type='L'. Refine the query with a selection on 'P' so that only patterns in which the m/z difference occurs as a loss from the precursor are kept, which grounds the pattern in a physical fragmentation process. Extract per-spectrum occurrences of the filtered patterns and identify the pattern shared by 7 spectra, annotated as tryptophan-containing tetrapeptides. Finally, cross-reference the spectra against the GNPS molecular network edges: 4 of the 7 spectra are not linked by modified cosine edges, demonstrating that the exact graph-structured pattern captures similarity the network missed. Write the filtered pattern list and occurrence table to CSV.

## Related tools

- **mineMS2** (R package used to query the mined pattern set via findMz (ppm=15, dmz=0.01, type='L') and extract per-spectrum occurrences of patterns containing the m/z difference) — https://github.com/odisce/mineMS2
- **R** (Execution environment in which the mineMS2 package is wrapped (via Rcpp) and the querying/occurrence-extraction script is run) — https://github.com/odisce/mineMS2
- **GNPS** (Molecular networking platform whose modified cosine edges are used as the reference linkage against which unlinked spectra recovered by the pattern are identified)

## Examples

```
patterns_trp <- findMz(patterns, mz = 186.079, ppm = 15, dmz = 0.01, type = 'L'); write.csv(occurrences, 'mineMS2_pattern_186.079_occurrences.csv')
```

## Evaluation signals

- The output CSV contains the pattern annotated as tryptophan-containing tetrapeptides with exactly 7 spectra
- 4 of the 7 spectra sharing the pattern are verified as unlinked in the GNPS molecular network, matching the reported finding
- Every retained pattern contains 186.079 as a loss from the precursor (the 'P' selection on top of type='L'), not merely as an inter-ion difference
- Query tolerances match the article's selection: ppm=15 and dmz=0.01 in findMz
- Per-spectrum occurrence counts in the CSV sum to the reported support (7 spectra) for the tetrapeptide pattern

## Limitations

- Exact pattern mining methods such as mineMS2 extract a high number of patterns, and no scoring system for patterns yet exists
- A subset of spectrum peaks must be selected due to combinatorial complexity, and the current criterion is based solely on peak intensities, which may result in a loss of chemically relevant information
- Patterns are mined from m/z differences below 200 Da for which a possible formula is found; the DAG representation does not yet require or exploit knowledge of the molecular formula

## Evidence

- [results] Querying patterns containing the 186.079 m/z difference recovers a pattern included in 7 spectra, not all linked in the GNPS network, identified as tetrapeptides containing a tryptophan residue.: [evidence span withheld — anchor status: not_found]
- [methods] Each MS/MS spectrum is converted to a fragmentation graph using the set of m/z bins: "Each MS/MS spectrum is converted to a fragmentation graph using the set of m/z bins"
- [abstract] these new fragmentation patterns capture similarities that are not extracted by existing methods: "these new fragmentation patterns capture similarities that are not extracted by existing methods"
- [results] We selected a tolerance of 15 ppm because a lower value produced patterns which were too sensitive to very small variations in m/z: "We selected a tolerance of 15 ppm because a lower value produced patterns which were too sensitive to very small variations in m/z"
- [readme] mineMS2 can be further coupled to the GNPS MS/MS molecular networking methodology to focus on patterns that best explain components of the network: [evidence span withheld — anchor status: not_found]
