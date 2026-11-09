---
name: graphml-network-visualization
description: Use when a MADByTE run has produced a chemical similarity network from
  spin-system feature comparisons and you need to inspect compound groupings, identify
  shared substructures, or spot false-positive connections.
when_to_use_negative:
- The analysis target is a compound class with exclusively shielded 1H signals (sterols,
  low-MW isoprenoids, lipids, volatile terpenes), which MADByTE's TOCSY shielded-quadrant
  exclusion (<2.5 ppm in both F1 and F2) precludes from network construction
- Input is a raw 1D/2D NMR spectrum rather than an already-built spin-system similarity
  network; spectral processing and spin-system construction must happen first
- The goal is quantitative compound quantification rather than structural relationship
  visualization; the network conveys shared-feature similarity, not concentration
edam_operation: null
edam_topics: []
tools:
- name: MADByTE
  role: Generates the chemical similarity network from TOCSY/HSQC spin-system features
    and exports it in graphML format
  repo: https://github.com/liningtonlab/MADByTE
- name: Gephi
  role: Imports the graphML file and renders the network using the Force Atlas 2 layout
    algorithm to expose compound clusters
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jnatprod.0c01076
    title: Development of an NMR-Based Platform for the Direct Structural Annotation
      of Complex Natural Products Mixtures
  source_content_hash: sha256:50a6f1d0efdf5e9cc0ab49ff7e2568c7885823ebf74bfa3a54cb3fd8f0f43720
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/liningtonlab/MADByTE
  tools:
  - Gephi
  - MADByTE
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# graphml-network-visualization

## Summary

Exports a MADByTE chemical similarity network as a graphML file and renders it in Gephi using the Force Atlas 2 layout algorithm so that structurally related compounds appear as visual clusters of shared spin-system features.

## When to use

Use when a MADByTE run has produced a chemical similarity network from spin-system feature comparisons and you need to inspect compound groupings, identify shared substructures, or spot false-positive connections.

## When NOT to use

- The analysis target is a compound class with exclusively shielded 1H signals (sterols, low-MW isoprenoids, lipids, volatile terpenes), which MADByTE's TOCSY shielded-quadrant exclusion (<2.5 ppm in both F1 and F2) precludes from network construction
- Input is a raw 1D/2D NMR spectrum rather than an already-built spin-system similarity network; spectral processing and spin-system construction must happen first
- The goal is quantitative compound quantification rather than structural relationship visualization; the network conveys shared-feature similarity, not concentration

## Inputs

- MADByTE chemical similarity network exported in graphML format (from generate_network)
- Optional: reference compound annotations for the samples in the network (e.g. compound names/classes from the 17-compound Zenodo training dataset, doi:10.5281/zenodo.4317721)

## Outputs

- Gephi-rendered network layout (Force Atlas 2) showing clusters of structurally related compounds and unconnected singleton sub-networks
- Recorded compound cluster assignments usable for dereplication or bioactive-metabolite prioritization

## How to apply

Run generate_network in the MADByTE pipeline and export the resulting network in graphML format; the network's nodes are spin-system features annotated with 1H and 13C chemical shifts, and edges are drawn only between reciprocal feature matches, scored with the higher of the two pairwise similarity scores (nodes with only a single connection are removed). Load the graphML file into Gephi and apply the Force Atlas 2 layout algorithm to spatially cluster samples that share spin-system features. Record the cluster assignments from the layout and compare them against known structural relationships — e.g. the 17-compound reference set should cluster macrolides (azithromycin, erythromycin, roxithromycin), chloramphenicols (chloramphenicol, thiamphenicol), and anthracyclines (daunomycin, epirubicin) while leaving puromycin and mupirocin as unconnected singletons. Clustered compounds can share only partial substructures (e.g. the macrolides connect via the lactone-junction spin system and cladinose sugar resonances), so interpret edges as shared substructure, not whole-molecule identity.

## Related tools

- **MADByTE** (Generates the chemical similarity network from TOCSY/HSQC spin-system features and exports it in graphML format) — https://github.com/liningtonlab/MADByTE
- **Gephi** (Imports the graphML file and renders the network using the Force Atlas 2 layout algorithm to expose compound clusters)

## Examples

```
conda activate madbyte; python madbyte_gui.py  # run generate_network, export graphML, open in Gephi with Force Atlas 2
```

## Evaluation signals

- graphML file imports cleanly into Gephi and contains reciprocal, score-annotated edges between spin-system nodes (non-reciprocal edges should already be removed)
- Known structural families form separate clusters: macrolides (azithromycin/erythromycin/roxithromycin), chloramphenicols (chloramphenicol/thiamphenicol), and anthracyclines (daunomycin/epirubicin)
- Structurally dissimilar compounds (puromycin, mupirocin) remain as single unconnected sub-networks with no false-positive edges to other compounds
- Edges within a cluster are explainable by shared spin-system features (e.g. macrolide lactone-junction resonances at 0.76-0.80, 1.37-1.38, 1.77-1.81, 4.75-5.13 ppm in 1H)

## Limitations

- MADByTE does not identify every spin system in a given mixture, so network completeness depends on spin-system construction quality
- Edge interpretation reflects shared substructures, not whole-molecule identity — related analogues cluster via partial features such as the macrolide lactone junction
- Dereplication of specialist compound classes requires user-supplied reference spectra; the visualization inherits whatever reference coverage exists

## Evidence

- [results] Reference-set validation of the rendered clustering: [evidence span withheld — anchor status: not_found]
