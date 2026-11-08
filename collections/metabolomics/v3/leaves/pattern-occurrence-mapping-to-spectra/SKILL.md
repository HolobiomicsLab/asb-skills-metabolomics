---
name: pattern-occurrence-mapping-to-spectra
description: Use when you have mined fragmentation patterns (mineMS2 pattern set)
  and need to find which patterns contain a specific m/z difference or loss and which
  spectra share them, especially to flag spectra that are not connected in a GNPS
  molecular network.
when_to_use_negative:
- Input is a feature table or annotated peak list without individual MS/MS spectra
  — occurrence mapping requires spectra-level pattern memberships
- Goal is probabilistic motif discovery à la MS2LDA (distributions of fragments/losses)
  rather than exact graph-structured pattern querying
- You need to rank or score patterns by informativeness — the article notes no scoring
  system for patterns yet exists
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_3292
tools:
- name: mineMS2
  role: R package providing findMz to query mined patterns for a target m/z difference
    and extract their per-spectrum occurrences
  repo: https://github.com/odisce/mineMS2
- name: GNPS
  role: Source of the molecular network edge list used to determine which spectra
    sharing a pattern are not linked in the network
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-025-01051-y
    title: 'mineMS2: annotation of spectral libraries with exact fragmentation patterns'
  source_content_hash: sha256:4fb43cc9e82de53fc6e038cc30dc372b536a96d711dad9ebca632ea88c42b9d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/odisce/mineMS2
  tools:
  - GNPS
  - R
  - mineMS2
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Pattern occurrence mapping to spectra

## Summary

Queries a mineMS2-mined pattern set for a target m/z difference (e.g. the 186.079 Da tryptophan loss) and maps the filtered patterns' occurrences onto individual MS/MS spectra to reveal similarities invisible to the GNPS molecular network.

## When to use

Use when you have mined fragmentation patterns (mineMS2 pattern set) and need to find which patterns contain a specific m/z difference or loss and which spectra share them, especially to flag spectra that are not connected in a GNPS molecular network.

## When NOT to use

- Input is a feature table or annotated peak list without individual MS/MS spectra — occurrence mapping requires spectra-level pattern memberships
- Goal is probabilistic motif discovery à la MS2LDA (distributions of fragments/losses) rather than exact graph-structured pattern querying
- You need to rank or score patterns by informativeness — the article notes no scoring system for patterns yet exists

## Inputs

- Mined mineMS2 pattern set (fragmentation-graph patterns of m/z differences) from an MS/MS spectra collection, e.g. the LIMS-DB dataset of 622 MS/MS spectra
- Target m/z difference value (e.g. 186.079 Da) with query tolerances (ppm=15, dmz=0.01) and type selector (e.g. 'L' for loss, refined with 'P')
- GNPS molecular network edge list (from the GNPS platform) for the same spectra, to compare pattern co-occurrence against network links

## Outputs

- Filtered list of patterns containing the queried m/z difference (optionally restricted to precursor losses)
- Per-spectrum occurrence table mapping each filtered pattern to the spectra that contain it
- Exported occurrence file (e.g. mineMS2_pattern_186.079_occurrences.csv) and count of spectra unlinked in the GNPS network

## How to apply

Load the mined pattern set produced from an MS/MS spectra collection (e.g. the 622-spectrum LIMS-DB dataset mined with the mineMS2 R package), then query all patterns containing the target m/z difference with the findMz function using ppm=15, dmz=0.01 and type='L' for losses; the 15 ppm tolerance was chosen because lower values made patterns too sensitive to small m/z variations. Refine the query with a selection on 'P' so only patterns in which the m/z difference occurs as a loss from the precursor are kept. Extract the occurrences of the filtered patterns to build a per-spectrum occurrence table, identify the spectra sharing each pattern (e.g. a pattern in 7 spectra annotated as tryptophan-containing tetrapeptides), and cross-check those spectra against GNPS molecular network edges to count how many are unlinked. Assemble the filtered pattern list and occurrence table and export them (e.g. to a CSV such as mineMS2_pattern_186.079_occurrences.csv).

## Related tools

- **mineMS2** (R package providing findMz to query mined patterns for a target m/z difference and extract their per-spectrum occurrences) — https://github.com/odisce/mineMS2
- **GNPS** (Source of the molecular network edge list used to determine which spectra sharing a pattern are not linked in the network)

## Examples

```
findMz(minedPatterns, mz = 186.079, ppm = 15, dmz = 0.01, type = "L")
```

## Evaluation signals

- The queried pattern (e.g. the tryptophan-loss 186.079 pattern) is recovered with the expected support — 7 spectra, of which 4 are unlinked in the GNPS molecular network
- After the 'P' selection, only patterns in which the m/z difference occurs as a loss from the precursor remain in the output
- The occurrence table contains one row per spectrum-pattern pair and matches the filtered pattern list exported to CSV
- Query used the article-validated tolerance parameters ppm=15 and dmz=0.01 with type='L'
- Filtered patterns are exact graphs: all m/z differences of each pattern are present in every spectrum containing that pattern

## Limitations

- Exact pattern mining methods such as mineMS2 extract a high number of patterns, so queries may return many hits needing refinement
- A subset of spectrum peaks must be selected due to combinatorial complexity, and the current criterion is based solely on peak intensities, which may lose chemically relevant information
- m/z differences in patterns include differences between ions on distinct fragmentation paths, not only neutral losses, so occurrence does not always imply the same physical fragmentation process

## Evidence

- [readme] Each pattern is a graph with ion peaks as nodes and m/z differences as edges.: "Each **pattern is a graph** with **ion peaks as nodes** and **m/z differences as edges**."
- [readme] coupled to the GNPS MS/MS molecular networking methodology: "coupled to the GNPS MS/MS molecular networking"
- [results] We selected a tolerance of 15 ppm because a lower value produced patterns which were too sensitive to very small variations in m/z: "We selected a tolerance of 15 ppm because a lower value produced patterns which were too sensitive to very small variations in m/z"
- [methods] A set of disjoint bins is derived from the full set of m/z differences between the 15 most intense fragment ions from each spectrum: "A set of disjoint bins is derived from the full set of m/z differences between the 15 most intense fragment ions from each spectrum"
