---
name: molecular-network-component-analysis
description: Use when a GNPS molecular network component (e.g. a 21-spectra cluster)
  contains unknown spectra that pairwise similarity alone cannot structurally explain,
  and you want the mined fragmentation pattern with highest support and precision
  over that component.
when_to_use_negative:
- The goal is to extract common structures shared by only two spectra — GNPS pairwise
  edges already propagate those annotations and mineMS2 pattern mining is unnecessary
- The spectra have not been processed by mineMS2 pattern mining (discretizeMzDifferences
  binning at 15 ppm and graph building), so no candidate patterns exist to score against
  the component
- Probabilistic substructure topics are preferred over exact graph-structured patterns
  — MS2LDA/MolNetEnhancer is the alternative pipeline
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: mineMS2
  role: R package providing infoPatterns (support/precision of patterns over component
    spectra) and plotPatterns for component-level pattern interpretation
  repo: https://github.com/odisce/mineMS2
- name: GNPS
  role: Source of the molecular network whose vertices are spectra and whose edges
    are high modified-cosine pairwise similarities; components are the objects to
    be explained
- name: igraph
  role: R library used to identify the largest connected component (21 spectra) of
    the GNPS network graph
- name: R
  role: 'Runtime environment: mineMS2 is an R package (C++/Boost core wrapped with
    Rcpp) and the component analysis is scripted in R'
  repo: https://github.com/odisce/mineMS2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-025-01051-y
    title: 'mineMS2: annotation of spectral libraries with exact fragmentation patterns'
  source_content_hash: sha256:94e0bbc4bcf0b285f06a59be2cca4bc53236842be6114f0d2437878de9be2088
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# molecular-network-component-analysis

## Summary

Couples mineMS2-extracted exact fragmentation patterns to GNPS molecular network components to identify the pattern that best explains a component (support/precision over its spectra) and annotate its unknown members, e.g. the 99.0683 m/z difference (C5H9ON, valine loss) explaining the 21-spectra P. nordicum fungisporin component.

## When to use

Use when a GNPS molecular network component (e.g. a 21-spectra cluster) contains unknown spectra that pairwise similarity alone cannot structurally explain, and you want the mined fragmentation pattern with highest support and precision over that component.

## When NOT to use

- The goal is to extract common structures shared by only two spectra — GNPS pairwise edges already propagate those annotations and mineMS2 pattern mining is unnecessary
- The spectra have not been processed by mineMS2 pattern mining (discretizeMzDifferences binning at 15 ppm and graph building), so no candidate patterns exist to score against the component
- Probabilistic substructure topics are preferred over exact graph-structured patterns — MS2LDA/MolNetEnhancer is the alternative pipeline

## Inputs

- GNPS molecular network (spectrum vertices plus modified-cosine similarity edges) from the MS/MS dataset
- MS/MS spectra library in MGF format (e.g. pnordicum.mgf, 51 P. nordicum spectra)
- Spectra metadata table (e.g. dataset/pnordicum_ms2_info.tsv shipped with the mineMS2 package)
- Set of mineMS2-extracted fragmentation patterns (fragmentation graphs with candidate molecular formulas per m/z difference)

## Outputs

- Connected-component statistics (number of spectra, e.g. 21) identified with igraph
- Best-explaining pattern with edge annotation (m/z difference 99.0683, closest formula C5H9ON, valine loss)
- Support and precision values for the selected pattern (19/21 component spectra; precision = 1)
- Pattern plots mapped onto the component spectra (plotPatterns)
- Summary file recording component statistics, pattern edge annotation, support, and precision

## How to apply

Load the GNPS molecular network and the mineMS2-mined patterns (previously extracted with the frequent-subgraph mining pipeline: m/z difference binning at 15 ppm, formula generation for differences of 14–200 Da, DAG fragmentation-graph building) into R. Identify the target connected component with igraph — for the P. nordicum dataset this is the 21-spectra fungisporin cluster. For each mined pattern, compute support and precision over the component's spectra using the mineMS2 infoPatterns function, and select the pattern with highest coverage/precision. Verify the pattern's edge annotation (e.g. an m/z difference of 99.0683 with closest formula C5H9ON, consistent with a valine loss) — exact-graph patterns make this interpretation direct because every m/z difference of the pattern is present in every spectrum containing it. Visualize with plotPatterns to confirm the pattern covers 19 of the 21 component spectra, check that no occurrence exists outside the component (precision = 1), and save component statistics, edge annotation, support, and precision to a summary file.

## Related tools

- **mineMS2** (R package providing infoPatterns (support/precision of patterns over component spectra) and plotPatterns for component-level pattern interpretation) — https://github.com/odisce/mineMS2
- **GNPS** (Source of the molecular network whose vertices are spectra and whose edges are high modified-cosine pairwise similarities; components are the objects to be explained)
- **igraph** (R library used to identify the largest connected component (21 spectra) of the GNPS network graph)
- **R** (Runtime environment: mineMS2 is an R package (C++/Boost core wrapped with Rcpp) and the component analysis is scripted in R) — https://github.com/odisce/mineMS2

## Examples

```
pnordicum_metadata.c <- system.file("dataset/pnordicum_ms2_info.tsv", package = "mineMS2"); pnordicum.m2l <- ms2Lib(pnordicum_mgf.c, suppInfos = pnordicum_metadata.df)
```

## Evaluation signals

- The selected pattern's edge is annotated with the expected m/z difference and closest formula: 99.0683 / C5H9ON (valine loss) for the largest fungisporin component
- Precision equals 1: the pattern occurs in no dataset spectra outside the target component
- Support matches the expectation of 19 out of 21 component spectra, confirmed by plotPatterns visualization
- The component size (21 spectra) is reproduced as the largest connected component of the GNPS network in igraph
- Component statistics, edge annotation, support, and precision are all written to the summary output file

## Limitations

- Exact pattern mining such as mineMS2 extracts a high number of patterns, making component-level selection of the best explainer necessary
- A subset of spectrum peaks must be selected due to combinatorial complexity (currently based solely on peak intensities), which may lose chemically relevant information
- No scoring system for patterns yet exists; identifying the most informative patterns is proposed as future work
- GNPS molecular networks propagate annotations between pairs of spectra but cannot extract common structures shared by more than two spectra — this is why mineMS2 coupling is needed
- Molecular formula knowledge is not required by the DAG representation, and adding formula-based options is left to future package versions

## Evidence

- [intro] coupling mineMS2 to molecular networks: "we show the potential of coupling mineMS2 to molecular network approaches such as GNPS to help interpreting the detected components and annotate unknown spectra."
- [readme] component-level pattern focus: [evidence span withheld — anchor status: not_found]
- [readme] exact-graph interpretation rationale: "the structure of mineMS2 patterns in the form of exact graphs (all m/z differences of the pattern are present in all spectra containing this pattern)"
- [intro] GNPS limitation motivating the skill: "it does not, however, enable to extract common structures between more than two spectra"
- [readme] input data provenance: [evidence span withheld — anchor status: not_found]
- [supplementary] package data loading invocation: "pnordicum_metadata.c <- system.file("dataset/pnordicum_ms2_info.tsv", package = "mineMS2")"
- [methods] fragmentation graph construction: "Each MS/MS spectrum is converted to a fragmentation graph using the set of m/z bins: a vertex is created for each fragment ion"
