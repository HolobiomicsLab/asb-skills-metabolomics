---
name: chemical-similarity-network-generation
description: Use when you have TOCSY/HSQC spectra for a sample set (e.g., prefractionated
  extract library) and need to cluster samples by shared structural features to dereplicate
  known scaffolds or locate bioactivity hotspots.
when_to_use_negative:
- The compounds of interest have exclusively shielded 1H signals (e.g., sterols, low
  MW isoprenoids, lipids, volatile terpenes) — the <2.5 ppm TOCSY shielded quadrant
  is excluded, so these classes are precluded unless user-supplied reference spectra
  are added
- You have a single spectrum rather than a sample set — the network step matches spin
  system features BETWEEN samples and requires multiple samples
- Input is already a molecular fingerprint/MS/MS spectral library similarity table
  rather than NMR spin system features — MADByTE operates on TOCSY/HSQC-derived spin
  systems
edam_operation: null
edam_topics: []
tools:
- name: MADByTE
  role: Provides spin_system_construction, correlation_matrix_generation, and generate_network
    functions that build the chemical similarity networks; run via the GUI (madbyte_gui.py)
  repo: https://github.com/liningtonlab/MADByTE
- name: Gephi
  role: Visualization of exported graphML networks using the Force Atlas 2 layout
    algorithm
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_007
  - task_003
  - task_005
  source_papers:
  - doi: 10.1021/acs.jnatprod.0c01076
    title: Development of an NMR-Based Platform for the Direct Structural Annotation
      of Complex Natural Products Mixtures
  source_content_hash: sha256:72451f9293992a6cb70f7b1a588564da73c4bc4f4af1f911d15f8698eb329311
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/liningtonlab/MADByTE
  tools:
  - Gephi
  - MADByTE
  - correlation_matrix_generation
  - generate_network
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# chemical-similarity-network-generation

## Summary

MADByTE's network generation converts pairwise spin-system similarity scores (derived from paired TOCSY/HSQC spectra) into full association, similarity, and hybrid chemical similarity networks exported as graphML for dereplication and bioactivity-guided prioritization of natural product mixtures.

## When to use

Use when you have TOCSY/HSQC spectra for a sample set (e.g., prefractionated extract library) and need to cluster samples by shared structural features to dereplicate known scaffolds or locate bioactivity hotspots.

## When NOT to use

- The compounds of interest have exclusively shielded 1H signals (e.g., sterols, low MW isoprenoids, lipids, volatile terpenes) — the <2.5 ppm TOCSY shielded quadrant is excluded, so these classes are precluded unless user-supplied reference spectra are added
- You have a single spectrum rather than a sample set — the network step matches spin system features BETWEEN samples and requires multiple samples
- Input is already a molecular fingerprint/MS/MS spectral library similarity table rather than NMR spin system features — MADByTE operates on TOCSY/HSQC-derived spin systems

## Inputs

- Paired 2D TOCSY and phase-sensitive HSQC spectra per sample (e.g., 600 MHz Avance III data in DMSO-d6)
- Spin system correlation/similarity matrix from correlation_matrix_generation
- Optional: reference compound spectra (e.g., 17-compound training set from Zenodo 10.5281/zenodo.4317721) for hybrid networks

## Outputs

- Full association network (graphML)
- Similarity network (graphML)
- Hybrid network linking library samples to reference compounds (graphML)
- Node-to-sample cluster assignments interpretable in Gephi (Force Atlas 2 layout)

## How to apply

First construct spin system features per sample with spin_system_construction: pick TOCSY/HSQC peaks, select the NMR solvent, apply the documented filters (solvent/water bands at ±0.02 ppm in 1H, three illegitimate HSQC regions, exclusion of the <2.5 ppm TOCSY shielded quadrant, and TOCSY signals lacking HSQC cross-peaks), symmetrize TOCSY f1 against f2, and build directed-graph features annotated with 13C shifts. Then run correlation_matrix_generation to score spin system features between all sample pairs, using the higher of the two directional similarity scores (A vs B or B vs A) to define edges. Prune by removing nodes with only a single connection to a spin system and dropping non-reciprocal edges, then run generate_network to produce the three named outputs — the full association network, the similarity network, and the hybrid network (samples linked to reference compounds). Export each network in graphML format and visualize in Gephi with the Force Atlas 2 layout to record cluster assignments.

## Related tools

- **MADByTE** (Provides spin_system_construction, correlation_matrix_generation, and generate_network functions that build the chemical similarity networks; run via the GUI (madbyte_gui.py)) — https://github.com/liningtonlab/MADByTE
- **Gephi** (Visualization of exported graphML networks using the Force Atlas 2 layout algorithm)

## Examples

```
conda activate madbyte; python madbyte_gui.py
```

## Evaluation signals

- Each of the three .graphml output files (full association, similarity, hybrid) exists and parses as a valid graphML graph
- Node/edge counts and edge weights correspond to the input similarity matrix entries after pruning of single-connection nodes and non-reciprocal edges
- Reproduce the 17-compound reference validation: macrolides (azithromycin/erythromycin/roxithromycin), chloramphenicols (chloramphenicol/thiamphenicol), and anthracyclines (daunomycin/epirubicin) cluster; puromycin and mupirocin remain unconnected singletons
- Spiked-prefraction check: novobiocin-, mupirocin-, and erythromycin-spiked samples sub-cluster by compound identity, each connected to its reference compound by at least two shared spin system features
- Hybrid-network dereplication check: library samples 1565C/1565D are placed in the novobiocin-linked region (confirmed by UPLC-MS against a commercial standard)

## Limitations

- MADByTE will not identify every spin system in a given mixture
- TOCSY cross-peaks where both F1 and F2 are < 2.5 ppm are excluded, so compound classes with exclusively shielded 1H signals (sterols, lipids, volatile terpenes) cannot be dereplicated unless user reference spectra are supplied
- Peak picking performs poorly in the crowded shielded region of the spectrum
- Spectral overlap can yield multiple candidate 13C chemical shifts; the closest HSQC resonance value is used, which may introduce annotation ambiguity
- Dereplication is limited by the lack of large-scale NMR spectral repositories; lightweight NMR predictors lack sufficient accuracy for reference generation

## Evidence
