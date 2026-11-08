---
name: molecular-network-construction-ms2-spectra
description: Use when a differential LC/MS feature table plus centroided MS/MS raw
  data are available and structurally related differential features need to be grouped
  by MS2 spectral similarity into subnetworks for class-level annotation.
when_to_use_negative:
- Input is only a peak/intensity table without associated MS2 scans — no spectra exist
  to compute cosine similarities
- The goal is statistical differential analysis of features rather than structural
  clustering of MS2 spectra
- Molecular formula prediction/filtering (Seven Golden Rules) is needed rather than
  spectral networking
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Metaboseek
  role: Platform for loading data, associating MS2 scans to features (FindMS2ScansModule),
    network viewing, and .graphML export (SaveNetworkModule)
  repo: https://github.com/mjhelf/Metaboseek
- name: MassTools
  role: R package whose makeEdges()/network1() functions compute pairwise MS2 spectrum
    similarity edges
  repo: https://github.com/mjhelf/MassTools
- name: igraph
  role: R package used with MassTools to build and manipulate the spectral similarity
    network
  repo: https://github.com/igraph/rigraph
- name: xcms
  role: Feature detection and grouping underlying the feature table fed into networking
  repo: https://github.com/sneumann/xcms
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:f3723819c767f57a27e83fd31ee4dd9dcfae6044e3af2b40217df1ec0f011778
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjhelf/Metaboseek
  tools:
  - MassTools
  - Metaboseek
  - igraph
  - xcms
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# molecular-network-construction-ms2-spectra

## Summary

Construct an MS/MS molecular network in Metaboseek by averaging per-feature MS2 scans and linking features whose spectra exceed a cosine similarity threshold, enabling clustered annotation of related metabolites (e.g. the hacl-1 exo-metabolome subnetworks SN1-SN5).

## When to use

Use when a differential LC/MS feature table plus centroided MS/MS raw data are available and structurally related differential features need to be grouped by MS2 spectral similarity into subnetworks for class-level annotation.

## When NOT to use

- Input is only a peak/intensity table without associated MS2 scans — no spectra exist to compute cosine similarities
- The goal is statistical differential analysis of features rather than structural clustering of MS2 spectra
- Molecular formula prediction/filtering (Seven Golden Rules) is needed rather than spectral networking

## Inputs

- Differential feature table (e.g. >1000 hacl-1-enriched exo-metabolome features)
- Centroided LC/MS raw data files with MS2 scans
- Parameters: m/z tolerance (ppm), retention-time window, 'Noise level in %', 'min. peaks', 'Cosine threshold', optional 'Use parent masses'

## Outputs

- Molecular network graph exported as .graphML (molecular_network.graphml)
- Identified subnetworks of spectrally related features (e.g. SN1-SN5)
- Per-feature averaged MS2 spectra and pairwise cosine similarity edges

## How to apply

Load the feature table and centroided MS data into Metaboseek's Load module, then associate MS2 scans with features via the FindMS2ScansModule using m/z tolerance in ppm and a retention-time window, averaging scans per feature. Compute pairwise spectrum similarities with the MassTools makeEdges()/network1() workflow: exclude peaks below the 'Noise level in %' intensity threshold, score 0 when fewer than 'min. peaks' match, optionally enable 'Use parent masses' so neutral losses contribute, and use the cosine score between matched-peak intensity vectors. Apply a 'Cosine threshold' to retain only significant edges and remove weak comparisons before viewing. Inspect the graph in the Metaboseek network viewer (SHIFT+click to zoom into subgraphs, color by default groups) to identify subnetworks, and export via the SaveNetworkModule to .graphML (molecular_network.graphml).

## Related tools

- **Metaboseek** (Platform for loading data, associating MS2 scans to features (FindMS2ScansModule), network viewing, and .graphML export (SaveNetworkModule)) — https://github.com/mjhelf/Metaboseek
- **MassTools** (R package whose makeEdges()/network1() functions compute pairwise MS2 spectrum similarity edges) — https://github.com/mjhelf/MassTools
- **igraph** (R package used with MassTools to build and manipulate the spectral similarity network) — https://github.com/igraph/rigraph
- **xcms** (Feature detection and grouping underlying the feature table fed into networking) — https://github.com/sneumann/xcms

## Examples

```
devtools::install_github("mjhelf/Metaboseek"); devtools::install_github("mjhelf/MassTools")  # then run MassTools makeEdges()/network1() and export via Metaboseek SaveNetworkModule
```

## Evaluation signals

- The exported molecular_network.graphml resolves into the expected number of major subnetworks (five for the hacl-1 exo-metabolome, SN1-SN5)
- Subnetwork composition matches expected chemical classes: SN1 = N-acyl glycophosphoethanolamides, SN2-SN4 = N-acyl glycoglycerophosphoethanolamides, SN5 = N-acyl ethanolamides
- Edges below the chosen 'Cosine threshold' are absent from the exported graph
- Spectrum pairs matching fewer than 'min. peaks' receive a similarity score of 0 and produce no edge
- Nodes annotated with differential enrichment values (e.g. 10-100-fold hacl-1 enrichment) render correctly in the network viewer

## Limitations

- Peaks below the 'Noise level in %' of maximum scan intensity are excluded, which can drop genuine low-intensity fragments
- 'ignore small fragments' is an experimental feature that excludes peaks below m/z 100 and may remove informative fragments
- Network edges reflect spectral similarity only; annotations remain at confidence Level 3 (tentative structure or compound class) and many clustered features may remain unidentified

## Evidence

- [readme] Metaboseek is an interactive tool to analyze and browse your mass spectrometry data.: "Metaboseek is an interactive tool to analyze and browse your mass spectrometry data."
- [methods] Noise level filter: "peaks at an intensity below a set percentage of the maximum peak 
intensity in a scan (`Noise level in %`) will be excluded"
- [methods] Min peaks rule: "The similarity score
will be considered 0 if less than `min. peaks` match between two spectra"
- [methods] Cosine threshold: "select a threshold for which comparisons to keep (above a given 
`Cosine threshold`)"
- [methods] ignore small fragments: "You can also
`ignore small fragments`, an experimental feature that will exclude peaks with 
m/z < 100 m/z from the spectra"
