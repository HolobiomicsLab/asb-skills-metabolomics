---
name: graph-based-msms-spectral-representation
description: Use when you need to convert an MGF-format MS/MS spectral collection
  into fragmentation graphs (graphs of m/z differences) so that subsequent frequent-subgraph
  mining can extract exact fragmentation patterns shared across spectra.
when_to_use_negative:
- Spectra are already represented as molecular networks (e.g. GNPS components) — this
  skill builds per-spectrum graphs, not between-spectrum similarity networks
- You need probabilistic fragment/loss motifs such as MS2LDA topics rather than exact
  graph-structured patterns, or the pattern goal is ranking structures rather than
  capturing shared m/z-difference structure
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mineMS2
  role: R package providing ms2Lib (builds the ms2Lib object from MGF + metadata)
    and discretizeMzDifferences (bins m/z differences and converts them to candidate
    formulas) to construct the fragmentation graphs
  repo: https://github.com/odisce/mineMS2
- name: R
  role: runtime environment in which the mineMS2 package (written in C++ with Boost,
    wrapped via Rcpp) is invoked to build and store the graphs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-025-01051-y
    title: 'mineMS2: annotation of spectral libraries with exact fragmentation patterns'
  source_content_hash: sha256:105b99c03e4a47295f5a15ffd6c67d3c1534924f3ab4b6affa43f9fd03eec576
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/odisce/mineMS2
  tools:
  - R
  - mineMS2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# graph-based-msms-spectral-representation

## Summary

Represent each MS/MS spectrum in a collection as a fragmentation graph (DAG) whose vertices are ion peaks and whose edges are binned m/z differences annotated with candidate molecular formulas — the representation that mineMS2's frequent-subgraph mining operates on, enabling exact, structured pattern detection across spectra.

## When to use

Use when you need to convert an MGF-format MS/MS spectral collection into fragmentation graphs (graphs of m/z differences) so that subsequent frequent-subgraph mining can extract exact fragmentation patterns shared across spectra.

## When NOT to use

- Spectra are already represented as molecular networks (e.g. GNPS components) — this skill builds per-spectrum graphs, not between-spectrum similarity networks
- You need probabilistic fragment/loss motifs such as MS2LDA topics rather than exact graph-structured patterns, or the pattern goal is ranking structures rather than capturing shared m/z-difference structure

## Inputs

- MS/MS spectra in MGF format (e.g. pnordicum_ms2_spectra.mgf)
- supplementary spectrum metadata as a TSV data frame (e.g. pnordicum_ms2_info.tsv)
- parameters: maxFrags=15 (15 most intense fragment ions per spectrum), dmz=0.007, ppm=15

## Outputs

- ms2Lib R object containing one fragmentation graph (DAG of ion-peak vertices and binned m/z-difference edges with candidate molecular formulas) per spectrum
- saved .rds file (e.g. fragmentation_graphs_pnordicum.rds via saveRDS)

## How to apply

Load the MGF spectra (e.g. pnordicum_ms2_spectra.mgf) plus any supplementary metadata TSV into R and build the ms2Lib object with mineMS2's ms2Lib function. Run discretizeMzDifferences to derive a set of disjoint m/z bins from the full set of differences between the 15 most intense fragment ions (maxFrags=15) of each spectrum, using dmz=0.007 for low-m/z tolerance and 15 ppm tolerance — the paper selected 15 ppm because lower values produced patterns too sensitive to very small m/z variations. Generate candidate neutral formulas for all m/z differences between 14 and 200 Da, discarding differences with no possible formula, and assign a formula when there is a unique match. Finally, build the fragmentation graph for each spectrum: create a vertex for each fragment ion and add an edge for each binned m/z difference, storing the ms2Lib object (saveRDS) for downstream mining. This graph representation keeps exact structure — every m/z difference of a pattern is present in all spectra containing it — which makes the resulting patterns chemically interpretable in a way that MS2LDA's distribution-based motifs are not.

## Related tools

- **mineMS2** (R package providing ms2Lib (builds the ms2Lib object from MGF + metadata) and discretizeMzDifferences (bins m/z differences and converts them to candidate formulas) to construct the fragmentation graphs) — https://github.com/odisce/mineMS2
- **R** (runtime environment in which the mineMS2 package (written in C++ with Boost, wrapped via Rcpp) is invoked to build and store the graphs)

## Examples

```
pnordicum_metadata.c <- system.file("dataset/pnordicum_ms2_info.tsv", package = "mineMS2"); pnordicum.m2l <- ms2Lib(pnordicum_mgf.c, suppInfos = pnordicum_metadata.df)
```

## Evaluation signals

- Each spectrum in the MGF yields exactly one fragmentation graph inside the returned ms2Lib object (e.g. 51 spectra → 51 graphs)
- Edge labels consist of binned m/z differences that fall within the 14–200 Da window and carry a possible neutral formula; differences with no formula are absent
- Labeling error rate in triangles is below 1%, supporting Properties 1 and 2 of the graph construction
- A 15 ppm tolerance is applied for m/z-difference binning; lower tolerances should make patterns overly sensitive to tiny m/z shifts
- Graph structure is a DAG (acyclic) over the selected (up to 15) most intense fragment ions per spectrum

## Limitations

- Peak selection is currently based solely on peak intensities (the 15 most intense ions), which may lose chemically relevant information
- Combinatorial complexity forces selecting a subset of spectrum peaks before graph construction
- Option to exploit known molecular formula knowledge in the DAG representation is not yet implemented
- Exact pattern mining on the resulting graphs extracts a high number of patterns; no scoring system for pattern informativeness exists yet
- Performance bound: up to ~3,000 spectra in a few hours on a standard laptop

## Evidence

- [methods] Graph representation of spectra: "Each MS/MS spectrum is converted to a fragmentation graph using the set of m/z bins: a vertex is created for each fragment ion"
- [methods] m/z difference binning parameters: "A set of disjoint bins is derived from the full set of m/z differences between the 15 most intense fragment ions from each spectrum"
- [methods] Formula generation window: "Possible neutral formulas are generated for all the m/z differences between 14 and 200."
- [results] Tolerance rationale: "We selected a tolerance of 15 ppm because a lower value produced patterns which were too sensitive to very small variations in m/z"
- [readme] Interpretability advantage of exact graphs: "The candidate molecular formulas for each m/z difference are computed to help interpretation (below 200 Da)."
- [readme] Graph definition in README: "Each pattern is a graph with ion peaks as nodes and m/z differences as edges."
