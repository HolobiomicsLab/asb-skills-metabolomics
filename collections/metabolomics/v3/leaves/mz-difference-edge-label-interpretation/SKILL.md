---
name: mz-difference-edge-label-interpretation
description: 'Use when fragmentation-graph patterns from mineMS2 must be chemically
  annotated: each edge label is a discretized m/z difference (14–200 Da) whose candidate
  neutral formulas must be generated and interpreted.'
when_to_use_negative:
- When the analyst expects edge labels to be exclusively precursor neutral losses
  — mineMS2 labels also include differences between ions from distinct fragmentation
  paths
- When exact formula assignment is required rather than candidate formula generation
  — the method only proposes possible neutral formulas for differences between 14
  and 200 Da
- When a probabilistic distribution of fragments/losses (e.g. an MS2LDA topic) is
  the target object instead of an exact graph of m/z differences
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mineMS2
  role: R package that discretizes m/z differences, generates candidate formulas for
    edge labels, and outputs the closed patterns to be interpreted
  repo: https://github.com/odisce/mineMS2
- name: Rcpp
  role: Wraps the C++ (Boost) mining/annotation core into the mineMS2 R package used
    to produce the labeled patterns
  repo: https://github.com/odisce/mineMS2
- name: Boost
  role: C++ library used to implement the graph algorithms that generate the labeled
    fragmentation graphs and patterns
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-025-01051-y
    title: 'mineMS2: annotation of spectral libraries with exact fragmentation patterns'
  source_content_hash: sha256:ab4d2b3ca600c3b1754626a13a15f479d2818821365de487f0266e28bcbb4d8a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/odisce/mineMS2
  tools:
  - Boost
  - R
  - Rcpp
  - mineMS2
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# m/z difference edge-label interpretation

## Summary

Interprets the edge labels of mineMS2 fragmentation-graph patterns — discretized m/z differences (binned at 15 ppm) annotated with candidate neutral-loss molecular formulas — to chemically explain fragmentation patterns shared by spectra collections.

## When to use

Use when fragmentation-graph patterns from mineMS2 must be chemically annotated: each edge label is a discretized m/z difference (14–200 Da) whose candidate neutral formulas must be generated and interpreted.

## When NOT to use

- When the analyst expects edge labels to be exclusively precursor neutral losses — mineMS2 labels also include differences between ions from distinct fragmentation paths
- When exact formula assignment is required rather than candidate formula generation — the method only proposes possible neutral formulas for differences between 14 and 200 Da
- When a probabilistic distribution of fragments/losses (e.g. an MS2LDA topic) is the target object instead of an exact graph of m/z differences

## Inputs

- Mined closed fragmentation patterns from mineClosedSubgraphs, reported as pattern id + edge labels (discretized m/z differences) + supporting spectra (support)
- Fragmentation graphs (ms2Lib objects) built from MS/MS spectra with m/z bins at 15 ppm tolerance
- Spectrum metadata (e.g. pnordicum_ms2_info.tsv) for tracing pattern support to spectra

## Outputs

- Chemically annotated pattern table: each edge label paired with candidate neutral-loss molecular formula (< 200 Da)
- Structural interpretation of patterns/components (e.g. explanation of ChemOnt concepts or GNPS molecular network components)

## How to apply

First recall how edge labels arise: m/z differences between the 15 most intense fragment ions of each spectrum are binned (disjoint bins, 15 ppm tolerance) and differences between 14 and 200 Da for which no possible neutral formula can be generated are discarded, so every retained edge label carries a candidate formula. Note that labels are not restricted to neutral losses from the precursor — they also include m/z differences between ions belonging to distinct fragmentation paths, which the README stresses may be specific to particular molecules. When interpreting a mined closed pattern, read its edge labels as an exact, structured graph (all differences present in all supporting spectra), associate each label with its candidate neutral-loss formula below 200 Da, and use the shared-label topology to propose structural interpretation of the molecular network component or unknown spectra. If coupling to GNPS molecular networks, prioritize the patterns whose edge labels best explain the component of interest rather than the full pattern list.

## Related tools

- **mineMS2** (R package that discretizes m/z differences, generates candidate formulas for edge labels, and outputs the closed patterns to be interpreted) — https://github.com/odisce/mineMS2
- **Rcpp** (Wraps the C++ (Boost) mining/annotation core into the mineMS2 R package used to produce the labeled patterns) — https://github.com/odisce/mineMS2
- **Boost** (C++ library used to implement the graph algorithms that generate the labeled fragmentation graphs and patterns)

## Examples

```
pnordicum_m2l <- ms2Lib(pnordicum_mgf.c, suppInfos = pnordicum_metadata.df); pats <- mineClosedSubgraphs(pnordicum_m2l, sizeMin = 1, count = 2)
```

## Evaluation signals

- Every retained edge label has at least one candidate neutral formula and lies in the 14–200 Da range (differences outside this range or without possible formulas were discarded)
- Edge-label discretization used the 15 ppm tolerance; re-deriving bins at lower tolerance should not be attempted since lower values made patterns too sensitive to small m/z variations
- Interpreted patterns are closed: no superpattern exists with identical support over the same spectra
- Support counts of the interpreted pattern match re-counting of its labeled occurrences across the source spectra (e.g. the 51 P. nordicum spectra)
- The interpretation treats the pattern as an exact graph — all m/z differences of the pattern are present in all spectra containing it

## Limitations

- The high number of patterns extracted by exact mining makes exhaustive interpretation difficult; no scoring system for patterns yet exists
- Peak selection (currently based solely on peak intensities) may lose chemically relevant information, affecting which m/z differences become edge labels
- Molecular formula knowledge is not used in the DAG representation, limiting label interpretation (option could be added in future versions)

## Evidence

- [methods] Formula generation: "m/z differences between 14 and 200 Da for which no possible formula is found are discarded."
- [readme] Edge label semantics: "These m/z differences can be any difference between the m/z values of two peaks of a spectrum, provided that they are frequent (i.e. detected in at least two spectra)."
- [readme] Distinct-path differences: "They therefore include not only neutral losses but also m/z differences between ions that belong to distinct fragmentation paths of the precursor"
- [results] Tolerance selection: "We selected a tolerance of 15 ppm because a lower value produced patterns which were too sensitive to very small variations in m/z"
- [readme] Exactness aids interpretation: "the structure of mineMS2 patterns in the form of exact graphs (all m/z differences of the pattern are present in all spectra containing this pattern) facilitates their chemical interpretation."
