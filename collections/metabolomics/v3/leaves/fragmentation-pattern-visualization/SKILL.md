---
name: fragmentation-pattern-visualization
description: Use when a GNPS molecular network component (e.g. the 21-spectra fungisporin
  cluster of P. nordicum) must be visually interpreted with a mineMS2 pattern, showing
  the exact m/z-difference graph and which component spectra contain it.
when_to_use_negative:
- When the goal is de novo mining of fragmentation patterns rather than visualizing/interpreting
  patterns already extracted by mineMS2 (use the mineMS2 frequent-subgraph mining
  workflow instead)
- When patterns are probabilistic distributions of fragments and losses (e.g. MS2LDA
  motifs) rather than exact graphs — such patterns cannot be drawn as an exact m/z-difference
  graph with identical support across spectra
- 'When the input is a single spectrum without a spectra collection: mineMS2 m/z differences
  must be frequent (detected in at least two spectra)'
edam_topics:
- http://edamontology.org/topic_3336
tools:
- name: mineMS2
  role: R package providing the plotPatterns function to visualize pattern structure
    mapped onto component spectra
  repo: https://github.com/odisce/mineMS2
- name: R
  role: Runtime environment in which the mineMS2 package and its visualization functions
    are executed
  repo: https://github.com/odisce/mineMS2
- name: GNPS
  role: Source of the molecular network whose components are being visually interpreted
    with the mined patterns
- name: igraph
  role: Used to identify the largest connected component (21 spectra) whose spectra
    are then mapped in the pattern plot
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-025-01051-y
    title: 'mineMS2: annotation of spectral libraries with exact fragmentation patterns'
  source_content_hash: sha256:d50210ea9525052f84aff880c1632126b0869beae7b470a1138fa1886acfcae9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/odisce/mineMS2
  tools:
  - GNPS
  - R
  - igraph
  - mineMS2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# fragmentation-pattern-visualization

## Summary

Visualize mineMS2 exact fragmentation patterns — graphs with ion peaks as nodes and m/z differences as edges — mapped onto the spectra of a GNPS molecular network component, to chemically interpret the component (e.g. the valine-loss pattern with m/z difference 99.0683, closest formula C5H9ON, explaining the 21-spectra P. nordicum fungisporin cluster).

## When to use

Use when a GNPS molecular network component (e.g. the 21-spectra fungisporin cluster of P. nordicum) must be visually interpreted with a mineMS2 pattern, showing the exact m/z-difference graph and which component spectra contain it.

## When NOT to use

- When the goal is de novo mining of fragmentation patterns rather than visualizing/interpreting patterns already extracted by mineMS2 (use the mineMS2 frequent-subgraph mining workflow instead)
- When patterns are probabilistic distributions of fragments and losses (e.g. MS2LDA motifs) rather than exact graphs — such patterns cannot be drawn as an exact m/z-difference graph with identical support across spectra
- When the input is a single spectrum without a spectra collection: mineMS2 m/z differences must be frequent (detected in at least two spectra)

## Inputs

- MS/MS spectra in .mgf format with supplementary metadata (.tsv), e.g. pnordicum_mgf.c and pnordicum_metadata.df loaded via ms2Lib
- mineMS2-mined fragmentation patterns (graph objects with m/z-difference edges and candidate molecular formulas)
- GNPS molecular network component membership (e.g. the 21-spectra component from the P. nordicum dataset)

## Outputs

- Pattern plots showing the exact m/z-difference graph mapped onto the component spectra (plotPatterns output)
- Verification summary file containing component statistics, pattern edge annotation (e.g. 99.0683 / C5H9ON valine loss), support (19/21) and precision (1)

## How to apply

Build the ms2Lib object from the .mgf spectra with supplementary metadata (e.g. system.file('dataset/pnordicum_ms2_info.tsv', package='mineMS2')), then use the mineMS2 plotPatterns function to draw the pattern structure mapped onto the component spectra. Because mineMS2 patterns are exact graphs — every m/z difference of the pattern is present in all spectra containing it — the plot lets you verify chemically interpretable edges such as the 99.0683 difference annotated with closest formula C5H9ON (valine loss). For the validation workflow, confirm visually that the plotted pattern covers 19 of the 21 component spectra and that its occurrences do not extend elsewhere in the dataset (precision = 1). Save the pattern plots together with component statistics, pattern edge annotation, support and precision into a summary file.

## Related tools

- **mineMS2** (R package providing the plotPatterns function to visualize pattern structure mapped onto component spectra) — https://github.com/odisce/mineMS2
- **R** (Runtime environment in which the mineMS2 package and its visualization functions are executed) — https://github.com/odisce/mineMS2
- **GNPS** (Source of the molecular network whose components are being visually interpreted with the mined patterns)
- **igraph** (Used to identify the largest connected component (21 spectra) whose spectra are then mapped in the pattern plot)

## Examples

```
devtools::install_github("odisce/mineMS2"); pnordicum.m2l <- ms2Lib(pnordicum_mgf.c, suppInfos = pnordicum_metadata.df); plotPatterns(best_pattern, pnordicum.m2l)
```

## Evaluation signals

- The plotted pattern's edge with m/z difference 99.0683 is annotated with closest molecular formula C5H9ON, consistent with a valine loss
- The visualized pattern is present in 19 of the 21 spectra of the target GNPS component (support = 19/21)
- The pattern has precision = 1: no occurrence of the visualized pattern in any dataset spectrum outside the component
- Because patterns are exact graphs, every m/z difference shown in the plot is present in all supporting spectra — the plot must reflect identical edge support across the 19 spectra
- The pattern plots and verification results (component statistics, edge annotation, support, precision) are saved into a summary file

## Limitations

- mineMS2 exact pattern mining extracts a high number of patterns, so the pattern to visualize must first be selected (e.g. by highest coverage/precision over the component)
- Peak selection is needed to limit combinatorial complexity and is currently based solely on peak intensities, which may result in a loss of chemically relevant information
- Candidate molecular formulas for m/z differences are only computed below 200 Da
- No scoring system for patterns yet exists; identifying the most informative patterns is proposed as future work

## Evidence

- [readme] Each pattern is a graph with ion peaks as nodes and m/z differences as edges.: "Each **pattern is a graph** with **ion peaks as nodes** and **m/z differences as edges**."
- [readme] Structure of mineMS2 patterns in the form of exact graphs facilitates their chemical interpretation.: "the **structure of *mineMS2* patterns in the form of exact graphs** (all m/z differences of the pattern are present in all spectra containing this pattern) **facilitates their chemical"
- [readme] mineMS2 can be further coupled to the GNPS MS/MS molecular networking methodology.: [evidence span withheld — anchor status: external_doc]
- [readme] Candidate molecular formulas for each m/z difference are computed to help interpretation (below 200 Da).: "The candidate **molecular formulas for each m/z difference** are computed to help interpretation (below 200 Da)."
- [readme] The included dataset consists of 51 spectra from the untargeted study of the secondary metabolism of Penicillium nordicum.: [evidence span withheld — anchor status: external_doc]
- [abstract] The mineMS2 software is publicly available as an R package.: "The mineMS2 software is publicly available as an R package"
