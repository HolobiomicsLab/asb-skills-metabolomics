---
name: natural-product-scaffold-clustering
description: Use when you have 1H, TOCSY and HSQC spectra for a panel of reference
  natural products or extract fractions and need to determine which compounds share
  structural scaffolds versus remain unconnected singletons in a chemical similarity
  network.
when_to_use_negative:
- Compound classes with exclusively shielded 1H signals — sterols, low MW isoprenoids,
  lipids, volatile terpenes — are precluded from analysis by the shielded-quadrant
  exclusion
- You have a single spectrum with no panel to compare against; MADByTE's network formation
  requires matching spin system features between multiple samples
- The goal is dereplication against a known-compound library rather than unsupervised
  scaffold grouping; use MADByTE's dereplication module instead of relying on network
  clustering
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: MADByTE
  role: Constructs spin system features from TOCSY/HSQC peak tables, generates the
    correlation matrix, and exports the chemical similarity network in graphML format
  repo: https://github.com/liningtonlab/MADByTE
- name: Gephi
  role: Visualizes the exported graphML network using the Force Atlas 2 layout algorithm
    so cluster assignments of structurally related compounds can be read off
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jnatprod.0c01076
    title: Development of an NMR-Based Platform for the Direct Structural Annotation
      of Complex Natural Products Mixtures
  source_content_hash: sha256:3d7eb6a8c9896cd445d88fab2d5e7fe15159a1bafb6b2133fcb734b31775e845
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/liningtonlab/MADByTE
  tools:
  - Gephi
  - MADByTE
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# natural-product-scaffold-clustering

## Summary

Clusters structurally related natural product compounds into sub-networks by matching TOCSY/HSQC spin system features across a sample panel with MADByTE, then visualizing the chemical similarity network in Gephi (Force Atlas 2) to verify scaffold groupings and singletons.

## When to use

Use when you have 1H, TOCSY and HSQC spectra for a panel of reference natural products or extract fractions and need to determine which compounds share structural scaffolds versus remain unconnected singletons in a chemical similarity network.

## When NOT to use

- Compound classes with exclusively shielded 1H signals — sterols, low MW isoprenoids, lipids, volatile terpenes — are precluded from analysis by the shielded-quadrant exclusion
- You have a single spectrum with no panel to compare against; MADByTE's network formation requires matching spin system features between multiple samples
- The goal is dereplication against a known-compound library rather than unsupervised scaffold grouping; use MADByTE's dereplication module instead of relying on network clustering

## Inputs

- 1H, TOCSY and HSQC NMR spectra for each compound or fraction in the panel (e.g. the 17-compound reference training dataset from Zenodo doi:10.5281/zenodo.4317721)
- NMR solvent declaration (e.g. DMSO-d6) selected in the MADByTE GUI

## Outputs

- Chemical similarity network exported in graphML format
- Spin system features and correlation matrix comparing features between all samples
- Cluster assignments of compounds observed in the Gephi Force Atlas 2 visualization

## How to apply

Download the reference training dataset (1H, TOCSY, HSQC spectra for 17 commercially available natural products and analogues, doi:10.5281/zenodo.4317721) or supply your own spectra acquired in the same NMR solvent (e.g. DMSO-d6 on a 600 MHz Avance III). Launch the MADByTE GUI (python madbyte_gui.py), select the NMR solvent from the drop-down menu, and run spin_system_construction on each spectrum, applying the documented filters: solvent/water region removal at ±0.02 ppm in 1H, removal of the three illegitimate HSQC regions (1H > 7.0 with 13C < 50; 1H < 7.0 with 13C > 170; 1H < 2.4 with 13C > 100), exclusion of the TOCSY shielded quadrant (<2.5 ppm in both F1 and F2), and dropping TOCSY signals lacking HSQC cross-peaks. Run correlation_matrix_generation to compare spin system features between all samples, removing nodes with only a single connection, dropping non-reciprocal edges, and defining each edge by the higher of the two pairwise similarity scores (A vs B or B vs A). Run generate_network and export the network in graphML format, then visualize in Gephi with the Force Atlas 2 layout and record cluster assignments. Rationale: shared spin system features (e.g. the macrolide lactone junction spin system at 0.76-0.80, 1.37-1.38, 1.77-1.81, 4.75-5.13 ppm plus the cladinose sugar) drive edge formation, so structurally related scaffolds co-cluster while unrelated compounds stay disconnected.

## Related tools

- **MADByTE** (Constructs spin system features from TOCSY/HSQC peak tables, generates the correlation matrix, and exports the chemical similarity network in graphML format) — https://github.com/liningtonlab/MADByTE
- **Gephi** (Visualizes the exported graphML network using the Force Atlas 2 layout algorithm so cluster assignments of structurally related compounds can be read off)

## Examples

```
conda activate madbyte; python madbyte_gui.py
```

## Evaluation signals

- Macrolides (azithromycin, erythromycin, roxithromycin), chloramphenicols (chloramphenicol, thiamphenicol) and anthracyclines (daunomycin, epirubicin) each form distinct clusters
- Puromycin and mupirocin remain as single sub-networks with no false-positive connections to other compounds in the test set
- All retained edges are reciprocal and scored by the higher of the two pairwise similarity scores; nodes with only a single connection have been removed
- The exported network is valid graphML that loads in Gephi under the Force Atlas 2 layout

## Limitations

- MADByTE will not identify every spin system in a given mixture
- Peak-picking algorithms perform poorly in the crowded shielded region, which is why TOCSY cross-peaks with both F1 and F2 < 2.5 ppm are excluded
- Spectral overlap can yield multiple candidate 13C chemical shifts; values of the closest HSQC resonance are used, which may misannotate nodes
- Specialist compound classes (sterols, low MW isoprenoids, lipids, volatile terpenes) with exclusively shielded 1H signals cannot be analyzed without user-supplied reference spectra

## Evidence
