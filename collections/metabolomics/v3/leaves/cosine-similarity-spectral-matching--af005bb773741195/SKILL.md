---
name: cosine-similarity-spectral-matching
description: Use when you have centroided MS/MS data plus a feature table and want
  to cluster structurally related differential features (e.g. lipid classes like N-acyl
  GPEs, GLEAs, NAEs) into molecular subnetworks.
when_to_use_negative:
- You only have MS1 feature intensities without associated MS2 scans — no spectra
  exist to compare
- The goal is formula prediction rather than clustering — use the SIRIUS-based formula/structure
  prediction module with the Seven Golden Rules filters instead
- Spectra are already matched against a reference database (e.g. GNPS/MassBank) —
  library annotation is a different task from de novo network clustering
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Metaboseek
  role: Hosts the FindMS2ScansModule, network viewer, and SaveNetworkModule for building,
    visualizing, and exporting the MS/MS network
  repo: https://github.com/mjhelf/Metaboseek
- name: MassTools
  role: Provides makeEdges()/network1() functions that compute pairwise cosine similarity
    between averaged MS2 spectra
  repo: https://github.com/mjhelf/MassTools
- name: igraph
  role: R graph package used alongside Metaboseek for network construction/analysis
    of the feature-similarity graph
  repo: https://github.com/igraph/rigraph
- name: xcms
  role: Feature detection and grouping underlying the Feature Table that feeds the
    MS/MS network
  repo: https://github.com/sneumann/xcms
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:dd1d65d7f3d7fead3ad0a420a2d4ba84207ed0b97b4ff4ef3842c1fa3e5c020f
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
yes_rate: 0.5
n_claims: 12
---

# cosine-similarity-spectral-matching

## Summary

Builds MS/MS molecular networks in Metaboseek/MassTools by averaging MS2 scans per feature, computing cosine similarity between matched-peak intensity vectors, and retaining only edges above a Cosine threshold — used here to resolve >1000 hacl-1-enriched exo-metabolome features into subnetworks SN1-SN5.

## When to use

Use when you have centroided MS/MS data plus a feature table and want to cluster structurally related differential features (e.g. lipid classes like N-acyl GPEs, GLEAs, NAEs) into molecular subnetworks.

## When NOT to use

- You only have MS1 feature intensities without associated MS2 scans — no spectra exist to compare
- The goal is formula prediction rather than clustering — use the SIRIUS-based formula/structure prediction module with the Seven Golden Rules filters instead
- Spectra are already matched against a reference database (e.g. GNPS/MassBank) — library annotation is a different task from de novo network clustering

## Inputs

- Differential exo-metabolome feature table (>1000 features, e.g. hacl-1-enriched)
- Centroided LC/MS raw data files containing MS1 and MS/MS scans
- m/z tolerance (ppm) and retention-time window settings for MS2-to-feature association

## Outputs

- Molecular network in .graphML format (molecular_network.graphml)
- Subnetwork assignments (e.g. SN1-SN5) grouping features by shared fragmentation
- Edge list with cosine similarity scores between feature spectra

## How to apply

Load the differential feature table and centroided MS data into Metaboseek's Load module, then associate MS2 spectra to features with FindMS2ScansModule using m/z tolerance in ppm and a retention-time window. Average MS2 scans per feature and compute pairwise spectrum similarities via MassTools makeEdges()/network1(): exclude peaks below the 'Noise level in %' intensity threshold (percentage of the maximum peak intensity in a scan), score 0 when fewer than 'min. peaks' match, optionally enable 'Use parent masses' so neutral losses contribute to peak matching, and calculate the cosine score between matched-peak intensity vectors. Optionally 'ignore small fragments' to exclude peaks with m/z < 100. Name the network and apply a 'Cosine threshold' so only significant comparisons survive as edges. Inspect subnetworks in the Metaboseek network viewer and export with SaveNetworkModule as .graphML (molecular_network.graphml).

## Related tools

- **Metaboseek** (Hosts the FindMS2ScansModule, network viewer, and SaveNetworkModule for building, visualizing, and exporting the MS/MS network) — https://github.com/mjhelf/Metaboseek
- **MassTools** (Provides makeEdges()/network1() functions that compute pairwise cosine similarity between averaged MS2 spectra) — https://github.com/mjhelf/MassTools
- **igraph** (R graph package used alongside Metaboseek for network construction/analysis of the feature-similarity graph) — https://github.com/igraph/rigraph
- **xcms** (Feature detection and grouping underlying the Feature Table that feeds the MS/MS network) — https://github.com/sneumann/xcms

## Examples

```
devtools::install_github("mjhelf/MassTools"); edges <- MassTools::makeEdges(MS2averaged, useParentMass=TRUE, scoreThreshold=cosine_threshold); net <- MassTools::network1(edges)
```

## Evaluation signals

- Exported molecular_network.graphml resolves into five major subnetworks matching SN1-SN5 composition (N-acyl GPEs in SN1; GLEAs in SN2-SN4; NAEs in SN5)
- Edges only exist between feature pairs with cosine similarity above the chosen Cosine threshold; pairs with fewer than 'min. peaks' matched peaks score 0
- No edges are drawn between scans whose peaks fall below the 'Noise level in %' intensity cutoff
- N-acyl GPE subnetwork nodes carry the reported 10-100-fold hacl-1 exo-metabolome enrichment annotation
- MS2-scan association counts per feature are consistent with the ppm m/z tolerance and retention-time window used in FindMS2ScansModule

## Limitations

- Metabolite annotations via this workflow are limited to confidence Level 3 (tentative structure or compound class assignments only); many detected features remain unidentified
- Spectrum similarity is computed only when at least 'min. peaks' peaks match — sparse or noisy MS2 spectra yield zero-score edges and disconnected features
- 'ignore small fragments' is described as an experimental feature that excludes peaks with m/z < 100 from the spectra

## Evidence

- [methods] Peaks at an intensity below a set percentage of the maximum peak intensity in a scan (`Noise level in %`) will be excluded: "peaks at an intensity below a set percentage of the maximum peak intensity in a scan (`Noise level in %`) will be excluded"
- [methods] The similarity score will be considered 0 if less than `min. peaks` match between two spectra: "The similarity score will be considered 0 if less than `min. peaks` match between two spectra"
- [methods] select a threshold for which comparisons to keep (above a given `Cosine threshold`): "select a threshold for which comparisons to keep (above a given `Cosine threshold`)"
- [methods] You can also `ignore small fragments`, an experimental feature that will exclude peaks with m/z < 100 m/z from the spectra: "You can also `ignore small fragments`, an experimental feature that will exclude peaks with 
m/z < 100 m/z from the spectra"
- [abstract] We here introduce Metaboseek, an open-source analysis platform designed for untargeted comparative metabolomics: "We here introduce Metaboseek, an open-source analysis platform designed for untargeted comparative metabolomics"
- [readme] It is built on functions from the xcms package, and provides a web-based graphical user interface built with shiny: "It is built on functions from the <a href = "https://github.com/sneumann/xcms">xcms</a> package, and provides a web-based graphical user interface built with <a href ="
