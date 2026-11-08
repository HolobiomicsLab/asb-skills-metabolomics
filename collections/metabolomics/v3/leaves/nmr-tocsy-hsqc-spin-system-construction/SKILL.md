---
name: nmr-tocsy-hsqc-spin-system-construction
description: Use when you have paired TOCSY and HSQC spectra for multiple complex
  natural products extract/prefraction samples and need shared spin system features
  to cluster samples or dereplicate known scaffolds.
when_to_use_negative:
- The compound class of interest (sterols, low MW isoprenoids, lipids, volatile terpenes)
  has exclusively shielded 1H signals — 0.4% of predicted molecules are entirely shielded
  and the <2.5 ppm TOCSY quadrant is excluded
- You expect to capture every spin system in the mixture — MADByTE will not identify
  every spin system, and peak picking performs poorly in the crowded shielded region
- Input is already a processed feature table or you need atom-accurate structure prediction
  — lightweight NMR predictors lack sufficient accuracy for dereplication at scale
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3292
tools:
- name: MADByTE
  role: 'Primary platform: peak picking, spin system construction, correlation matrix
    generation, and network assembly from TOCSY/HSQC data'
  repo: https://github.com/liningtonlab/MADByTE
- name: NMRglue
  role: Visualization of 1D spectra within the MADByTE processing panels
- name: Gephi
  role: Visualization of exported graphML networks using the Force Atlas 2 layout
    algorithm
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jnatprod.0c01076
    title: Development of an NMR-Based Platform for the Direct Structural Annotation
      of Complex Natural Products Mixtures
  source_content_hash: sha256:24f9cb55c6fa3175f93fd43287170003408eef2ef059bd7d18101d136fd1b0a9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/liningtonlab/MADByTE
  tools:
  - Gephi
  - MADByTE
  - NMRglue
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# nmr-tocsy-hsqc-spin-system-construction

## Summary

MADByTE skill for constructing spin system features from paired 2D TOCSY/HSQC NMR spectra of complex mixtures, then matching features across samples to build chemical similarity networks for dereplication and spiked-compound detection.

## When to use

Use when you have paired TOCSY and HSQC spectra for multiple complex natural products extract/prefraction samples and need shared spin system features to cluster samples or dereplicate known scaffolds.

## When NOT to use

- The compound class of interest (sterols, low MW isoprenoids, lipids, volatile terpenes) has exclusively shielded 1H signals — 0.4% of predicted molecules are entirely shielded and the <2.5 ppm TOCSY quadrant is excluded
- You expect to capture every spin system in the mixture — MADByTE will not identify every spin system, and peak picking performs poorly in the crowded shielded region
- Input is already a processed feature table or you need atom-accurate structure prediction — lightweight NMR predictors lack sufficient accuracy for dereplication at scale

## Inputs

- Paired 2D TOCSY and phase-sensitive HSQC spectra (e.g. raw NMR datasets acquired in DMSO-d6 on 600 MHz spectrometers, 50% NUS rate)
- Selected NMR solvent designation for the sample set
- Optional user-supplied reference spectra library for dereplication
- Spiked prefraction sample set with node-to-prefraction assignment metadata (e.g. the 9-sample Zenodo dataset)

## Outputs

- Per-sample spin system feature tables (directed graphs of 1H nodes annotated with 13C HSQC shifts)
- Cross-sample correlation/similarity matrix with edges defined by the higher of the two asymmetric pairwise scores
- Association network exported in graphML format
- Node-to-prefraction assignment table mapping spin system nodes to samples

## How to apply

Acquire TOCSY and phase-sensitive HSQC data (e.g. on Avance III TCI/QCI 600 MHz spectrometers in DMSO-d6) and process with MADByTE via the GUI (`python madbyte_gui.py`) or the exported spin_system_construction function. Pick peaks and filter HSQC peak lists by removing solvent/water bands (±0.02 ppm in 1H), the three illegitimate HSQC regions (1H > 7.0 with 13C < 50 ppm; 1H < 7.0 with 13C > 170 ppm; 1H < 2.4 with 13C > 100 ppm), excluding the TOCSY shielded quadrant (<2.5 ppm), symmetrizing TOCSY by correcting f1 shifts against f2, and excluding TOCSY signals without corresponding HSQC cross-peaks. Build spin system features as directed graphs from each TOCSY peak table with nodes annotated by 13C shifts from HSQC (using the closest HSQC resonance when overlap yields multiple candidates). Then call correlation_matrix_generation across samples using the higher of the two asymmetric pairwise similarity scores to define edges, and generate_network while removing nodes with only a single connection to a spin system and non-reciprocal edges; export graphML for visualization in Gephi with Force Atlas 2. Correctness is judged by whether known spiked or related compounds (e.g. azithromycin/erythromycin/roxithromycin, sharing lactone junction and cladinose sugar spin systems) form coherent sub-clusters around their reference nodes.

## Related tools

- **MADByTE** (Primary platform: peak picking, spin system construction, correlation matrix generation, and network assembly from TOCSY/HSQC data) — https://github.com/liningtonlab/MADByTE
- **NMRglue** (Visualization of 1D spectra within the MADByTE processing panels)
- **Gephi** (Visualization of exported graphML networks using the Force Atlas 2 layout algorithm)

## Examples

```
conda activate madbyte; python madbyte_gui.py
```

## Evaluation signals

- Spiked prefraction samples containing the same compound (novobiocin, mupirocin, or erythromycin) form distinct sub-clusters connected to their reference compound node by at least two shared spin system features
- Related macrolides (azithromycin, erythromycin, roxithromycin) cluster via the shared lactone junction spin system with resonances at 0.76-0.80, 1.37-1.38, 1.77-1.81, and 4.75-5.13 ppm
- Network edges are reciprocal (non-reciprocal edges removed) and no node retains only a single connection to a spin system
- Filtered HSQC peak lists contain no signals within ±0.02 ppm of solvent/water or in the three illegitimate regions (1H > 7.0/13C < 50; 1H < 7.0/13C > 170; 1H < 2.4/13C > 100)
- graphML file imports cleanly into Gephi and Force Atlas 2 layout reproduces the expected sub-cluster separation (e.g. Figure 4)

## Limitations

- TOCSY cross-peaks where both F1 and F2 chemical shifts are < 2.5 ppm are excluded, precluding compounds with exclusively shielded protons (only 0.4% of predicted microbial natural products, but including sterols, lipids, and terpenes)
- MADByTE will not identify every spin system in a given mixture, and peak-picking algorithms perform poorly in the crowded shielded region
- Phase-sensitive HSQC has marginally lower signal-to-noise than absolute value HSQC due to longer acquisition; a 50% NUS rate is used as a conservative value to limit NUS processing artefacts
- When spectral overlap yields multiple candidate 13C chemical shifts, only the closest HSQC resonance value is included
- Dereplication is limited by the lack of large-scale NMR repositories; current lightweight NMR predictors lack sufficient accuracy for this task
- Installation via Anaconda Navigator fails; use the included .bat script or `conda env create -f environment.yml`

## Evidence
