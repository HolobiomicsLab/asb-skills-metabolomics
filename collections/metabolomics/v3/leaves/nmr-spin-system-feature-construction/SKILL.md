---
name: nmr-spin-system-feature-construction
description: Use when you have 2D NMR (TOCSY + HSQC) peak data from complex natural
  product mixtures and need discrete, comparable spin system features to drive sample-to-sample
  similarity networks.
when_to_use_negative:
- Compounds with exclusively shielded 1H signals (e.g. sterols, low-MW isoprenoids,
  lipids, volatile terpenes) — the <2.5 ppm TOCSY quadrant is excluded and 1H < 2.5
  ppm cross-peaks are removed
- You already have a processed feature/similarity table and only need network generation
  — go straight to correlation_matrix_generation/generate_network instead
- Absolute-value (non-phase-sensitive) HSQC data is the only option and SNR trade-offs
  of phase-sensitive acquisition were not planned for
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3361
tools:
- name: MADByTE
  role: 'Runs spin_system_construction: solvent selection, TOCSY/HSQC filtering, peak
    picking, and directed-graph spin system feature construction with 13C annotation'
  repo: https://github.com/liningtonlab/MADByTE
- name: NMRglue
  role: Visualization panels for direct visualization of 1D spectra used to inspect
    peak picking and spin system construction
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1021/acs.jnatprod.0c01076
    title: Development of an NMR-Based Platform for the Direct Structural Annotation
      of Complex Natural Products Mixtures
  source_content_hash: sha256:d9407df8cebfcf773b935b02eceb374415db3b864ac9b7eeb49e7164b287d745
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/liningtonlab/MADByTE
  tools:
  - Gephi
  - MADByTE
  - NMRglue
  - spin_system_construction
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# nmr-spin-system-feature-construction

## Summary

Constructs spin system features from paired TOCSY and HSQC spectra by generating a directed graph from each TOCSY peak table (nodes = 1H signals) and annotating nodes with 13C shifts from HSQC cross-peaks, producing the feature objects MADByTE uses for chemical similarity networking.

## When to use

Use when you have 2D NMR (TOCSY + HSQC) peak data from complex natural product mixtures and need discrete, comparable spin system features to drive sample-to-sample similarity networks.

## When NOT to use

- Compounds with exclusively shielded 1H signals (e.g. sterols, low-MW isoprenoids, lipids, volatile terpenes) — the <2.5 ppm TOCSY quadrant is excluded and 1H < 2.5 ppm cross-peaks are removed
- You already have a processed feature/similarity table and only need network generation — go straight to correlation_matrix_generation/generate_network instead
- Absolute-value (non-phase-sensitive) HSQC data is the only option and SNR trade-offs of phase-sensitive acquisition were not planned for

## Inputs

- TOCSY spectra (processed peak lists) for each sample, acquired in DMSO-d6 on 600 MHz Bruker Avance III TCI/QCI instruments
- HSQC spectra (phase-sensitive, matching the TOCSY acquisitions) for each sample
- NMR solvent identity (selected via the MADByTE GUI drop-down menu)

## Outputs

- Per-sample spin system features as directed graphs with nodes representing 1H signals annotated with 13C shifts from HSQC
- Filtered TOCSY/HSQC peak tables ready for correlation_matrix_generation

## How to apply

Launch the MADByTE GUI ('python madbyte_gui.py', conda env 'madbyte') and run spin_system_construction on each spectrum, selecting the NMR solvent from the drop-down menu. Filter the HSQC peak lists to remove solvent/water bands at ±0.02 ppm in 1H and the three illegitimate HSQC regions (1H > 7.0 with 13C < 50 ppm; 1H < 7.0 with 13C > 170 ppm; 1H < 2.4 with 13C > 100 ppm). Exclude the shielded TOCSY quadrant (<2.5 ppm in both F1 and F2, since peak picking performs poorly in that crowded region and 0.4% of Natural Products Atlas molecules have exclusively shielded protons), symmetrize TOCSY data by correcting f1 chemical shifts against the corresponding f2 values, and drop TOCSY signals lacking an HSQC cross-peak in the 1H dimension. Then build the directed graph from the TOCSY peak table where nodes represent 1H signals, and annotate each node with 13C shifts from HSQC — where spectral overlap yields multiple candidate 13C values, use the closest HSQC resonance. The resulting spin system feature objects are the inputs to MADByTE's correlation_matrix_generation and generate_network steps.

## Related tools

- **MADByTE** (Runs spin_system_construction: solvent selection, TOCSY/HSQC filtering, peak picking, and directed-graph spin system feature construction with 13C annotation) — https://github.com/liningtonlab/MADByTE
- **NMRglue** (Visualization panels for direct visualization of 1D spectra used to inspect peak picking and spin system construction)

## Examples

```
conda activate madbyte; python madbyte_gui.py  # then run spin_system_construction with the NMR solvent selected from the GUI drop-down menu
```

## Evaluation signals

- Every retained TOCSY signal has a corresponding HSQC cross-peak in the 1H dimension
- No retained HSQC peaks fall in the solvent/water ±0.02 ppm band or the three illegitimate regions (1H > 7.0 / 13C < 50; 1H < 7.0 / 13C > 170; 1H < 2.4 / 13C > 100)
- No TOCSY cross-peaks remain where both F1 and F2 chemical shifts are < 2.5 ppm
- TOCSY f1 shifts are symmetrized against f2 (f1 == f2 for corrected peaks)
- On the 17-compound reference set (Zenodo 10.5281/zenodo.4317721), constructed features reproduce known shared spin systems, e.g. the macrolide lactone junction feature with four resonances at 0.76-0.80, 1.37-1.38, 1.77-1.81, and 4.75-5.13 ppm

## Limitations

- MADByTE will not identify every spin system in a given mixture
- TOCSY cross-peaks with both F1 and F2 < 2.5 ppm are excluded, precluding analysis of compound classes with exclusively shielded protons (sterols, low MW isoprenoids, lipids, volatile terpenes) unless the user supplies reference spectra
- Peak-picking algorithms perform poorly in the crowded shielded region
- Phase-sensitive HSQC has marginally lower SNR than absolute-value HSQC due to increased experiment length
- Where spectral overlap yields multiple candidate 13C chemical shifts, only the closest HSQC resonance value is assigned

## Evidence
