---
name: network-threshold-tuning-and-simplification
description: Use when pairwise MS/MS spectrum similarities for molecular features
  must be converted into a graph and weak edges removed to reveal subnetworks of related
  metabolites (e.g., N-acyl GPEs, GLEAs, NAEs).
when_to_use_negative:
- The input is already a curated network or .graphML file and no new MS2 spectrum
  similarity comparisons are needed
- Only a single MS1 feature table without associated MS2 scan data is available, since
  cosine edges cannot be computed
- The goal is formula-level filtering of predicted molecular formulas (Seven Golden
  Rules element/valence/H-C/NOPS-C ratio filters) rather than spectrum-based network
  simplification
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: Metaboseek
  role: Shiny GUI platform that hosts the FindMS2ScansModule (MS2-to-feature mapping),
    the network viewer, and the SaveNetworkModule for .graphML export
  repo: https://github.com/mjhelf/Metaboseek
- name: MassTools
  role: R package supplying the makeEdges()/network1() workflow that averages MS2
    scans per feature and computes cosine similarity scores between matched-peak intensity
    vectors
  repo: https://github.com/mjhelf/MassTools
- name: igraph
  role: R network library used alongside Metaboseek/MassTools to construct and manipulate
    the filtered molecular network graph
  repo: https://github.com/igraph/rigraph
- name: xcms
  role: Underlying feature detection and grouping package on which Metaboseek builds
    the feature table consumed by the network workflow
  repo: https://github.com/sneumann/xcms
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:d823ff249c8876e7b279f4a622dde427aa4858d5e3a9af541ffc11b0ca4ee0ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjhelf/Metaboseek
  tools:
  - MassTools
  - Metaboseek
  - igraph
  - xcms
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# network-threshold-tuning-and-simplification

## Summary

Tuning cosine-similarity thresholds and peak-matching filters when building MS/MS molecular networks in Metaboseek, so that the differential hacl-1 exo-metabolome features resolve into clean subnetworks (SN1-SN5) rather than an over-connected hairball.

## When to use

Use when pairwise MS/MS spectrum similarities for molecular features must be converted into a graph and weak edges removed to reveal subnetworks of related metabolites (e.g., N-acyl GPEs, GLEAs, NAEs).

## When NOT to use

- The input is already a curated network or .graphML file and no new MS2 spectrum similarity comparisons are needed
- Only a single MS1 feature table without associated MS2 scan data is available, since cosine edges cannot be computed
- The goal is formula-level filtering of predicted molecular formulas (Seven Golden Rules element/valence/H-C/NOPS-C ratio filters) rather than spectrum-based network simplification

## Inputs

- Differential exo-metabolome feature table (>1000 hacl-1-enriched features)
- Centroided LC-MS/MS raw data files associated with the feature table
- MS2 scans mapped to features with m/z (ppm) and retention-time window settings

## Outputs

- Molecular network of MS/MS similarity edges filtered by 'Cosine threshold'
- Five major subnetworks SN1-SN5 (N-acyl GPEs SN1; GLEA SN2-SN4; NAEs SN5)
- molecular_network.graphml exported via the SaveNetworkModule

## How to apply

After associating MS2 scans to features with the FindMS2ScansModule (m/z tolerance in ppm plus a retention-time window), average MS2 scans per feature and compute pairwise cosine scores with the MassTools makeEdges()/network1() workflow. Exclude peaks below the 'Noise level in %' intensity threshold relative to the maximum peak intensity in each scan, and set the score to 0 when fewer than 'min. peaks' peaks match between two spectra; optionally enable 'Use parent masses' so neutral losses contribute to peak matching. Name the network and apply a 'Cosine threshold' so only comparisons above that score are kept as edges, which simplifies the network by removing weak, nonspecific similarities. Inspect the resulting graph in the Metaboseek network viewer (SHIFT+click to zoom into subgraphs) to confirm the differential hacl-1 features separate into subnetworks such as SN1-SN5 before exporting via the SaveNetworkModule in .graphML format.

## Related tools

- **Metaboseek** (Shiny GUI platform that hosts the FindMS2ScansModule (MS2-to-feature mapping), the network viewer, and the SaveNetworkModule for .graphML export) — https://github.com/mjhelf/Metaboseek
- **MassTools** (R package supplying the makeEdges()/network1() workflow that averages MS2 scans per feature and computes cosine similarity scores between matched-peak intensity vectors) — https://github.com/mjhelf/MassTools
- **igraph** (R network library used alongside Metaboseek/MassTools to construct and manipulate the filtered molecular network graph) — https://github.com/igraph/rigraph
- **xcms** (Underlying feature detection and grouping package on which Metaboseek builds the feature table consumed by the network workflow) — https://github.com/sneumann/xcms

## Examples

```
devtools::install_github("mjhelf/MassTools"); library(MassTools); net <- network1(makeEdges(MS2features, "Cosine threshold" = 0.7))
```

## Evaluation signals

- Every retained edge in the exported network has a cosine score above the chosen 'Cosine threshold'
- No edge connects two spectra sharing fewer than the 'min. peaks' matched peaks (score is 0 in that case)
- The filtered graph of differential hacl-1 features resolves into five major subnetworks (SN1-SN5) matching the reported composition: N-acyl GPEs (SN1), GLEAs (SN2-SN4), and NAEs (SN5)
- N-acyl GPE subnetwork nodes carry the reported 10-100-fold hacl-1 exo-metabolome enrichment annotation
- The exported molecular_network.graphML loads correctly as a valid .graphML network file

## Limitations

- Similarity scores are set to 0 when fewer than 'min. peaks' peaks match, which can disconnect features with sparse MS2 spectra
- 'ignore small fragments' is described as an experimental feature that excludes peaks below 100 m/z and may alter edge scores
- Metabolite annotations remain at confidence Level 3 (tentative structure or compound class), so subnetwork composition assignments like SN1-SN5 are class-level rather than fully resolved structures

## Evidence

- [methods] Noise level in % peak intensity filter: "peaks at an intensity below a set percentage of the maximum peak intensity in a scan (`Noise level in %`) will be excluded"
- [methods] min. peaks matching rule: "The similarity score will be considered 0 if less than `min. peaks` match between two spectra"
- [methods] Cosine threshold edge filtering: "select a threshold for which comparisons to keep (above a given `Cosine threshold`)"
- [readme] Metaboseek platform built on xcms: "It is built on functions from the <a href = "https://github.com/sneumann/xcms">xcms</a> package, and provides a web-based graphical user interface built with"
