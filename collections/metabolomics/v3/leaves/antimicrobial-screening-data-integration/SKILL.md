---
name: antimicrobial-screening-data-integration
description: Use when a MADByTE chemical similarity network from an NMR platform (TOCSY/HSQC
  spin system features) of a prefractionated extract library must be annotated with
  antimicrobial screening results to identify which network clusters concentrate activity.
when_to_use_negative:
- The sample set has no antimicrobial (or other) screening measurements, so no node
  attributes can be overlaid and no bioactivity hotspots can be defined
- The similarity network is not built from MADByTE spin system features (e.g., input
  is already a feature table or an MS/MS molecular network) and the BioMAP attribute
  schema does not map onto the nodes
- The compounds of interest belong to specialist classes with exclusively shielded
  1H signals (sterols, low MW isoprenoids, lipids, volatile terpenes), which the MADByTE
  TOCSY/HSQC pipeline precludes from analysis
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: MADByTE
  role: Runs the 'generate_network' module on spin system feature data to build the
    chemical similarity association network that receives the BioMAP activity attributes
  repo: https://github.com/liningtonlab/MADByTE
- name: Gephi
  role: Visualizes the exported graphML bioactivity-annotated network using the Force
    Atlas 2 algorithm to reproduce the Figure 6A layout
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jnatprod.0c01076
    title: Development of an NMR-Based Platform for the Direct Structural Annotation
      of Complex Natural Products Mixtures
  source_content_hash: sha256:2b9348708cab7ade571dc567d1f6e46271938973258d3b1a40e7bcc4de5d7fb9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/liningtonlab/MADByTE
  tools:
  - Gephi
  - MADByTE
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# antimicrobial-screening-data-integration

## Summary

Overlay BioMAP antimicrobial screening data (15 clinically relevant bacterial pathogens) as node attributes onto a MADByTE chemical similarity association network built from spin system features, to localize bioactivity hotspots among prefractionated natural products extracts and guide NMR-directed isolation.

## When to use

Use when a MADByTE chemical similarity network from an NMR platform (TOCSY/HSQC spin system features) of a prefractionated extract library must be annotated with antimicrobial screening results to identify which network clusters concentrate activity.

## When NOT to use

- The sample set has no antimicrobial (or other) screening measurements, so no node attributes can be overlaid and no bioactivity hotspots can be defined
- The similarity network is not built from MADByTE spin system features (e.g., input is already a feature table or an MS/MS molecular network) and the BioMAP attribute schema does not map onto the nodes
- The compounds of interest belong to specialist classes with exclusively shielded 1H signals (sterols, low MW isoprenoids, lipids, volatile terpenes), which the MADByTE TOCSY/HSQC pipeline precludes from analysis

## Inputs

- MADByTE dataset for the 85-sample prefractionated microbial natural products library (Zenodo DOI 10.5281/zenodo.4317721)
- Spin system feature data derived from TOCSY and HSQC spectra (600 MHz Avance III TCI/QCI, DMSO-d6)
- BioMAP antimicrobial screening results for 15 clinically relevant bacterial pathogens
- List of BioMAP-active prefraction IDs (34 prefractions)

## Outputs

- Annotated chemical similarity network restricted to 34 BioMAP-active prefraction nodes, with bioactivity attributes
- graphML file suitable for Gephi visualization (Force Atlas 2 layout, reproducing Figure 6A)
- Identified bioactivity hotspot cluster(s) (e.g., prefractions 2108D and 2108E)

## How to apply

Download the deposited MADByTE dataset for the 85-sample prefractionated microbial natural products library from Zenodo (DOI 10.5281/zenodo.4317721) and run the MADByTE 'generate_network' module on the spin system feature data to build the full association network between samples. Restrict the network to the 34 prefractions flagged as active in the BioMAP screen (15 clinically relevant pathogens), then overlay the BioMAP results as node attributes, encoding narrow-to-broad spectrum activity with the paper's color coding. Inspect the annotated network for bioactivity hotspots — here the 2108D/2108E prefractions of extract 2108 (isolate RL12_176_HVF_A), which share spin system features that later yielded collismycin A via NMR-guided isolation. Export the annotated network in graphML format and visualize in Gephi with the Force Atlas 2 algorithm to reproduce the Figure 6A layout.

## Related tools

- **MADByTE** (Runs the 'generate_network' module on spin system feature data to build the chemical similarity association network that receives the BioMAP activity attributes) — https://github.com/liningtonlab/MADByTE
- **Gephi** (Visualizes the exported graphML bioactivity-annotated network using the Force Atlas 2 algorithm to reproduce the Figure 6A layout)

## Examples

```
conda activate madbyte; python madbyte_gui.py
```

## Evaluation signals

- The exported graphML parses as a valid graph and contains exactly the 34 BioMAP-active prefraction nodes
- Every node carries bioactivity attributes encoding narrow-to-broad spectrum activity across the 15 pathogens
- The 2108D/2108E hotspot (extract 2108, isolate RL12_176_HVF_A) is present as a connected, attribute-annotated cluster as reported in Figure 6A
- Force Atlas 2 rendering in Gephi reproduces the Figure 6A layout, with hotspot prefractions clustering together
- The network is traceable to the full 85-sample association network before restriction to the 34 active prefractions

## Limitations

- MADByTE will not identify every spin system in a given mixture, so some shared structural features underlying a hotspot may be absent from the network
- TOCSY cross-peaks where both F1 and F2 chemical shifts are < 2.5 ppm are excluded, precluding dereplication of compound classes with exclusively shielded 1H signals (sterols, low MW isoprenoids, lipids, volatile terpenes) unless the user supplies reference spectra
- Nodes containing only a single connection to a spin system are removed, and edges between nodes are removed if they are not reciprocal, so weakly supported similarities are pruned before bioactivity overlay

## Evidence

- [results] BioMAP panel definition: "antimicrobial screening data from our in-house BioMAP screening platform, comprising 15 clinically relevant bacterial pathogens"
- [results] graphML export and Gephi visualization: "The resulting networks were exported in graphML format and processed in Gephi for visualization using the Force Atlas 2 algorithm"
- [abstract] Network generation from matched spin system features: "matches spin system features between samples to create a chemical similarity network for a given sample set"
- [results] Dataset size of prefractionated library: "we acquired data for 85 samples from our prefractionated microbial natural products library"
- [readme] MADByTE GUI launcher (README): "Ensure your virtual environment is activated and run the launcher script.

conda activate madbyte
python madbyte_gui.py"
