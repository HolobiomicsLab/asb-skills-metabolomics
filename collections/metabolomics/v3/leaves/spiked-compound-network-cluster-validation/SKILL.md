---
name: spiked-compound-network-cluster-validation
description: Use when TOCSY/HSQC datasets for spiked or control spiked samples must
  be shown to cluster correctly in a MADByTE association network, e.g. to prove the
  platform detects non-native compounds in complex mixtures.
when_to_use_negative:
- Samples contain only shielded-proton compound classes (sterols, low MW isoprenoids,
  lipids, volatile terpenes) whose TOCSY cross-peaks with both F1 and F2 < 2.5 ppm
  are excluded by MADByTE
- The goal is identifying every spin system in a mixture — MADByTE will not identify
  every spin system, so absence from the network is not proof of absence
- Input is already a processed spin system feature table or network; re-running the
  TOCSY/HSQC peak-picking pipeline is unnecessary
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3292
tools:
- name: MADByTE
  role: Peak picking, spin system construction, correlation matrix generation, and
    association network assembly for the 9 spiked samples
  repo: https://github.com/liningtonlab/MADByTE
- name: Gephi
  role: Visualization of the exported graphML network using the Force Atlas 2 layout
    to confirm spiked-sample sub-clustering
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jnatprod.0c01076
    title: Development of an NMR-Based Platform for the Direct Structural Annotation
      of Complex Natural Products Mixtures
  source_content_hash: sha256:173edf9dfc0127f2a63888b75aa598a30d37d0871b071ae3f32ef3b16f12e9b1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/liningtonlab/MADByTE
  tools:
  - Gephi
  - MADByTE
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# spiked-compound-network-cluster-validation

## Summary

Validates MADByTE chemical similarity network construction by confirming that non-native reference compounds spiked into prefractionated extracts (novobiocin, mupirocin, erythromycin) form sample sub-clusters connected to their reference compound by shared spin system features.

## When to use

Use when TOCSY/HSQC datasets for spiked or control spiked samples must be shown to cluster correctly in a MADByTE association network, e.g. to prove the platform detects non-native compounds in complex mixtures.

## When NOT to use

- Samples contain only shielded-proton compound classes (sterols, low MW isoprenoids, lipids, volatile terpenes) whose TOCSY cross-peaks with both F1 and F2 < 2.5 ppm are excluded by MADByTE
- The goal is identifying every spin system in a mixture — MADByTE will not identify every spin system, so absence from the network is not proof of absence
- Input is already a processed spin system feature table or network; re-running the TOCSY/HSQC peak-picking pipeline is unnecessary

## Inputs

- Raw paired TOCSY/HSQC NMR datasets for 9 spiked prefraction samples (Zenodo doi:10.5281/zenodo.4317721)
- Reference compound spectra for novobiocin, mupirocin, and erythromycin (e.g. from the training dataset of 17 commercial natural products)
- NMR solvent selection (DMSO-d6) for each dataset

## Outputs

- graphML-format chemical similarity network of spin system nodes across the 9 spiked samples
- Node-to-prefraction assignment table mapping each spin system node to its sample
- Gephi Force Atlas 2 layout confirming sub-clusters of spiked prefractions around each reference compound

## How to apply

Download the raw TOCSY/HSQC data for the 9 spiked prefraction samples from Zenodo (doi:10.5281/zenodo.4317721), then run MADByTE (via `python madbyte_gui.py` or the exported spin_system_construction function) applying the standard filters: solvent/water bands at ±0.02 ppm in 1H, the three illegitimate HSQC regions (1H > 7.0 with 13C < 50; 1H < 7.0 with 13C > 170; 1H < 2.4 with 13C > 100), exclusion of the TOCSY shielded quadrant <2.5 ppm, and removal of TOCSY signals lacking HSQC cross-peaks. Call correlation_matrix_generation across all 9 samples, using the higher of the two asymmetric pairwise similarity scores to define edges, then generate_network while removing nodes with only a single connection to a spin system and non-reciprocal edges. Export the network as graphML, keep the node-to-prefraction assignment table, and import into Gephi with the Force Atlas 2 layout. Correct application is judged by samples spiked with the same compound clustering together and connecting to the reference compound node (as in Figure 4), with macrolide spikes sharing the lactone junction spin system (four resonances at 0.76–0.80, 1.37–1.38, 1.77–1.81, 4.75–5.13 ppm).

## Related tools

- **MADByTE** (Peak picking, spin system construction, correlation matrix generation, and association network assembly for the 9 spiked samples) — https://github.com/liningtonlab/MADByTE
- **Gephi** (Visualization of the exported graphML network using the Force Atlas 2 layout to confirm spiked-sample sub-clustering)

## Examples

```
conda activate madbyte; python madbyte_gui.py
```

## Evaluation signals

- Samples spiked with the same compound (novobiocin, mupirocin, or erythromycin) form distinct sub-clusters in the network, with most connected to the reference compound by at least two shared spin system features
- Network is exported as valid graphML with a node-to-prefraction assignment table available for every node
- Single-connection nodes and non-reciprocal edges are absent from the final network
- Edges derive from the higher of the two asymmetric pairwise similarity scores (A vs B or B vs A)
- Gephi Force Atlas 2 layout reproduces the Figure 4 clustering pattern

## Limitations

- MADByTE will not identify every spin system in a given mixture; the platform excludes TOCSY cross-peaks where chemical shifts in both F1 and F2 are < 2.5 ppm
- Peak-picking algorithms perform poorly in the crowded shielded region of the spectrum
- Dereplication is limited by the lack of large-scale NMR repositories; current lightweight NMR predictors lack sufficient accuracy

## Evidence

- [results] samples containing each spiked compound formed sub-clusters: [evidence span withheld — anchor status: not_found]
