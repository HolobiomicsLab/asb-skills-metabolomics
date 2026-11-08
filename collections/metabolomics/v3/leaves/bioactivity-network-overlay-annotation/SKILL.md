---
name: bioactivity-network-overlay-annotation
description: Use when a MADByTE chemical similarity network of spin system features
  must be annotated with bioactivity screening data to locate clusters of correlated
  active samples (e.g., 34 BioMAP-active prefractions) for prioritization.
when_to_use_negative:
- The goal is dereplication of known scaffolds alone, with no bioactivity screening
  data to overlay (use MADByTE's dereplication module against an internal library
  instead)
- Input is raw 1H/TOCSY/HSQC spectra rather than processed spin system features —
  the spectra must first pass through MADByTE peak picking and feature generation
- The sample set lacks paired screening results, so no bioactivity attributes can
  be encoded as node annotations
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0153
tools:
- name: MADByTE
  role: Generates the spin system chemical similarity association network from TOCSY/HSQC
    feature data that is subsequently annotated with bioactivity attributes and exported
    as graphML
  repo: https://github.com/liningtonlab/MADByTE
- name: Gephi
  role: Visualizes the annotated graphML network using the Force Atlas 2 algorithm
    to reveal bioactivity hotspot clusters
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jnatprod.0c01076
    title: Development of an NMR-Based Platform for the Direct Structural Annotation
      of Complex Natural Products Mixtures
  source_content_hash: sha256:7d945c30ca5967994c5b1393d5a9b56a178be4249ce97b9aaf0e0a4466bf347a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/liningtonlab/MADByTE
  tools:
  - Gephi
  - MADByTE
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# bioactivity-network-overlay-annotation

## Summary

Overlays biological screening results (e.g., BioMAP antimicrobial data) as node attributes onto a MADByTE spin system chemical similarity network to identify bioactivity hotspots among prefractionated natural products extracts and prioritize samples for NMR-guided isolation.

## When to use

Use when a MADByTE chemical similarity network of spin system features must be annotated with bioactivity screening data to locate clusters of correlated active samples (e.g., 34 BioMAP-active prefractions) for prioritization.

## When NOT to use

- The goal is dereplication of known scaffolds alone, with no bioactivity screening data to overlay (use MADByTE's dereplication module against an internal library instead)
- Input is raw 1H/TOCSY/HSQC spectra rather than processed spin system features — the spectra must first pass through MADByTE peak picking and feature generation
- The sample set lacks paired screening results, so no bioactivity attributes can be encoded as node annotations

## Inputs

- MADByTE spin system feature table / chemical similarity association network (85-sample prefractionated microbial natural products library)
- BioMAP antimicrobial screening results (15 clinically relevant bacterial pathogens) for 34 active prefractions
- Sample metadata (prefraction IDs, isolate identifiers, e.g., extract 2108 / RL12_176_HVF_A)

## Outputs

- Annotated graphML network file with bioactivity node attributes for the 34 BioMAP-active prefractions
- Gephi Force Atlas 2 network visualization reproducing the Figure 6A bioactivity hotspot layout

## How to apply

Download the deposited MADByTE dataset (Zenodo DOI 10.5281/zenodo.4317721) and run the MADByTE 'generate_network' module on the spin system feature data to build the full association network. Restrict the network to samples flagged active in the bioactivity screen (here, the 34 prefractions active in the BioMAP panel of 15 clinically relevant bacterial pathogens). Overlay the screening results as node attributes, encoding narrow-to-broad spectrum activity with a consistent color scheme. Identify bioactivity hotspot clusters (e.g., prefractions 2108D/2108E from extract 2108, isolate RL12_176_HVF_A) and export the annotated network in graphML format. Visualize in Gephi with the Force Atlas 2 algorithm to reproduce the published layout (Figure 6A).

## Related tools

- **MADByTE** (Generates the spin system chemical similarity association network from TOCSY/HSQC feature data that is subsequently annotated with bioactivity attributes and exported as graphML) — https://github.com/liningtonlab/MADByTE
- **Gephi** (Visualizes the annotated graphML network using the Force Atlas 2 algorithm to reveal bioactivity hotspot clusters)

## Examples

```
conda activate madbyte && python madbyte_gui.py
```

## Evaluation signals

- Exported graphML is a valid graph containing exactly the 34 BioMAP-active prefraction nodes with bioactivity attributes attached
- The 2108D/2108E hotspot (extract 2108, isolate RL12_176_HVF_A) is present as a connected cluster, matching Figure 6A
- Narrow-to-broad spectrum activity is encoded consistently via node attributes/color coding across the 15 pathogens
- Force Atlas 2 visualization in Gephi reproduces the hotspot topology reported in the paper

## Limitations

- MADByTE will not identify every spin system in a given mixture, so hotspot definition depends on the features that were detected
- TOCSY cross-peaks where both F1 and F2 shifts are < 2.5 ppm are excluded, and compound classes with exclusively shielded 1H signals (sterols, lipids, volatile terpenes) are precluded from analysis
- Edges are removed if not reciprocal, and nodes with only a single connection to a spin system are removed, which can fragment weakly connected bioactivity clusters

## Evidence
