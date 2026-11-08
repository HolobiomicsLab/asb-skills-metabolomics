---
name: nmr-mixture-dereplication-via-network-matching
description: Use when you have TOCSY and HSQC spectra for a sample set of microbial
  extracts/prefractions and need to dereplicate known compound families (e.g., novobiocin
  in prefractions 1565C/1565D) or group samples into regions of related chemical space.
when_to_use_negative:
- Analyzing compound classes with exclusively shielded 1H signals (sterols, low MW
  isoprenoids, lipids, volatile terpenes) unless user-supplied reference spectra for
  those classes are available
- Mixtures dominated by signals in the crowded shielded region (<2.5 ppm), where peak-picking
  algorithms perform poorly and TOCSY cross-peaks with both F1 and F2 < 2.5 ppm are
  excluded
- Tasks requiring exhaustive identification of every spin system in a mixture — MADByTE
  will not identify every spin system
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3292
tools:
- name: MADByTE
  role: 'Core platform: spin_system_construction, correlation_matrix_generation, generate_network,
    and dereplication module comparing network spin systems against a user-supplied
    internal library'
  repo: https://github.com/liningtonlab/MADByTE
- name: NMRglue
  role: Visualization of 1D spectra within MADByTE's visualization panels
- name: Gephi
  role: Rendering of exported graphML networks using the Force Atlas 2 algorithm
- name: NMRShiftDB2
  role: Source of stereo-aware HOSE code model for NMR prediction supporting the dereplication
    module
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jnatprod.0c01076
    title: Development of an NMR-Based Platform for the Direct Structural Annotation
      of Complex Natural Products Mixtures
  source_content_hash: sha256:49ba298fc071ea3137077b264a69f54f95d9187360a87a4c5940f0b077df5c9b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/liningtonlab/MADByTE
  tools:
  - Gephi
  - MADByTE
  - NMRShiftDB2
  - NMRglue
  - generate_network
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9545454545454546
n_claims: 22
---

# nmr-mixture-dereplication-via-network-matching

## Summary

A MADByTE-based dereplication workflow that combines TOCSY and HSQC NMR spectra of complex natural product mixtures, builds directed-graph spin system features annotated with 13C shifts, matches features across samples to assemble a chemical similarity network against reference compounds, and exports the hybrid network in graphML for dereplication and metabolite prioritization.

## When to use

Use when you have TOCSY and HSQC spectra for a sample set of microbial extracts/prefractions and need to dereplicate known compound families (e.g., novobiocin in prefractions 1565C/1565D) or group samples into regions of related chemical space.

## When NOT to use

- Analyzing compound classes with exclusively shielded 1H signals (sterols, low MW isoprenoids, lipids, volatile terpenes) unless user-supplied reference spectra for those classes are available
- Mixtures dominated by signals in the crowded shielded region (<2.5 ppm), where peak-picking algorithms perform poorly and TOCSY cross-peaks with both F1 and F2 < 2.5 ppm are excluded
- Tasks requiring exhaustive identification of every spin system in a mixture — MADByTE will not identify every spin system

## Inputs

- TOCSY spectra for each sample (e.g., the 85-sample Linington lab prefractionated microbial natural products library)
- HSQC spectra (phase-sensitive) for each sample
- Reference compound spectra for the dereplication module (e.g., 17 commercially available natural products and analogues; predicted spectra from Natural Products Atlas entries)
- NMR solvent selection (e.g., DMSO-d6) via GUI drop-down menu

## Outputs

- Directed-graph spin system features per sample with 13C-annotated nodes
- Correlation matrix of pairwise spin system similarities across samples
- Hybrid chemical similarity network in graphML format linking samples and reference compounds
- Dereplication assignments (e.g., 1565C/1565D mapped to the novobiocin-linked region)

## How to apply

Download the 85-sample prefraction library dataset (TOCSY and HSQC spectra) from Zenodo 10.5281/zenodo.4317721 and launch the MADByTE GUI via 'python madbyte_gui.py' inside the 'madbyte' conda environment. Run the exported 'spin_system_construction' function on each sample: pick TOCSY/HSQC peaks, filter solvent and water bands (±0.02 ppm in 1H) plus the three illegitimate HSQC regions (1H > 7.0 with 13C < 50 ppm; 1H < 7.0 with 13C > 170 ppm; 1H < 2.4 with 13C > 100 ppm), exclude the shielded TOCSY quadrant (<2.5 ppm in both F1 and F2), symmetrize TOCSY f1 shifts against f2, drop TOCSY signals lacking HSQC cross-peaks, and build directed-graph spin systems whose nodes are 1H signals annotated with 13C shifts from the HSQC. Run 'correlation_matrix_generation' to match spin system features between all sample pairs, defining edges with the higher of the two pairwise similarity scores and removing single-connection nodes and non-reciprocal edges. Then run 'generate_network' to assemble the hybrid network linking library samples to reference compounds (e.g., from a user-supplied internal dereplication library) and export it in graphML for visualization in Gephi with the Force Atlas 2 algorithm. Validate by confirming the network separates samples into distinct chemical-space regions and places samples 1565C/1565D in the novobiocin-linked group, confirmed by UPLC-MS against a commercial novobiocin standard.

## Related tools

- **MADByTE** (Core platform: spin_system_construction, correlation_matrix_generation, generate_network, and dereplication module comparing network spin systems against a user-supplied internal library) — https://github.com/liningtonlab/MADByTE
- **NMRglue** (Visualization of 1D spectra within MADByTE's visualization panels)
- **Gephi** (Rendering of exported graphML networks using the Force Atlas 2 algorithm)
- **NMRShiftDB2** (Source of stereo-aware HOSE code model for NMR prediction supporting the dereplication module)

## Examples

```
conda activate madbyte; python madbyte_gui.py
```

## Evaluation signals

- Reproduced network separates the 85 samples into distinct regions of related chemical space connected to reference compounds
- Samples 1565C and 1565D are placed in the novobiocin-linked region of the hybrid network
- 1565C/1565D novobiocin assignment is confirmed by UPLC-MS against a commercial novobiocin standard
- Related macrolides (azithromycin, erythromycin, roxithromycin) cluster via shared spin system features such as the lactone junction (resonances at 0.76-0.80, 1.37-1.38, 1.77-1.81, 4.75-5.13 ppm)
- Filtered peak lists contain no signals in solvent/water bands (±0.02 ppm) or the three illegitimate HSQC regions

## Limitations

- MADByTE will not identify every spin system in a given mixture
- TOCSY cross-peaks where both F1 and F2 < 2.5 ppm are excluded, precluding analysis of sterols, low MW isoprenoids, lipids, and volatile terpenes unless users supply reference spectra
- Dereplication is limited by the lack of large-scale NMR repositories; current 'lightweight' NMR predictors do not have sufficient accuracy to be useful for this task
- Peak-picking algorithms perform poorly in the crowded shielded region; multiple candidate 13C shifts from spectral overlap are resolved by using the closest HSQC resonance
- Do not install MADByTE using Anaconda Navigator — installation will fail; use the included .bat script or 'conda env create -f environment.yml'

## Evidence
