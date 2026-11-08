---
name: bioactive-prefraction-prioritization
description: Use when NMR (TOCSY/HSQC) data exist for a prefractionated extract library
  plus bioactivity screening results, and the goal is to locate network hotspots of
  shared spin system features correlated with activity for NMR-guided isolation.
when_to_use_negative:
- The library contains compound classes with exclusively shielded 1H signals (sterols,
  low MW isoprenoids, lipids, volatile terpenes) — these are precluded because TOCSY
  cross-peaks with both F1 and F2 < 2.5 ppm are excluded
- The input lacks bioactivity screening data, so 'hotspots' cannot be defined and
  the overlay prioritization has no activity signal to attribute
- Input is already a dereplicated feature table from a different platform without
  MADByTE spin system features — the network module expects MADByTE-generated TOCSY/HSQC
  feature data
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_0605
tools:
- name: MADByTE
  role: Generates spin system features from TOCSY/HSQC data and builds the chemical
    similarity association network; its 'dereplication' module can optionally compare
    network spin systems against a user-supplied library
  repo: https://github.com/liningtonlab/MADByTE
- name: Gephi
  role: Visualization of the exported graphML network using the Force Atlas 2 algorithm
    to reveal bioactivity hotspots
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jnatprod.0c01076
    title: Development of an NMR-Based Platform for the Direct Structural Annotation
      of Complex Natural Products Mixtures
  source_content_hash: sha256:8858a0e3c0ee4e5cc8771ed19172503cbdcb40c6582c7344178886195c4eb52e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/liningtonlab/MADByTE
  tools:
  - Gephi
  - MADByTE
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# bioactive-prefraction-prioritization

## Summary

Prioritize bioactive metabolites from a prefractionated microbial natural products library by overlaying BioMAP antimicrobial screening data onto MADByTE spin-system chemical similarity networks, then isolating from bioactivity hotspot clusters (e.g., the 2108D/2108E hotspot yielding collismycin A).

## When to use

Use when NMR (TOCSY/HSQC) data exist for a prefractionated extract library plus bioactivity screening results, and the goal is to locate network hotspots of shared spin system features correlated with activity for NMR-guided isolation.

## When NOT to use

- The library contains compound classes with exclusively shielded 1H signals (sterols, low MW isoprenoids, lipids, volatile terpenes) — these are precluded because TOCSY cross-peaks with both F1 and F2 < 2.5 ppm are excluded
- The input lacks bioactivity screening data, so 'hotspots' cannot be defined and the overlay prioritization has no activity signal to attribute
- Input is already a dereplicated feature table from a different platform without MADByTE spin system features — the network module expects MADByTE-generated TOCSY/HSQC feature data

## Inputs

- TOCSY and HSQC spectra (phase-sensitive HSQC, 50% NUS) for each prefractionated library sample (85 samples from the Linington lab prefractionated microbial natural products library)
- BioMAP antimicrobial screening results (15 clinically relevant bacterial pathogens) for library prefractions
- MADByTE spin system feature data / generated association network for the sample set
- Optionally, a user-supplied internal dereplication library of reference spectra

## Outputs

- Annotated chemical similarity network of the bioactive prefractions (e.g., 34 BioMAP-active nodes) in graphML format with bioactivity node attributes
- Gephi Force Atlas 2 visualization identifying bioactivity hotspot clusters (e.g., the 2108D/2108E hotspot from extract 2108, isolate RL12_176_HVF_A)
- Prioritized target prefractions for NMR-guided isolation of bioactive metabolites (e.g., collismycin A)

## How to apply

Acquire TOCSY and HSQC spectra for all library prefractions (in the study, 85 samples on 600 MHz Avance III TCI/QCI spectrometers in DMSO-d6) and run the MADByTE pipeline — data acquisition, peak picking (with solvent/water filtering at ±0.02 ppm in 1H, removal of the three illegitimate HSQC regions, exclusion of the <2.5 ppm TOCSY quadrant, and dropping TOCSY signals lacking HSQC cross-peaks), spin system feature creation, and inter-sample comparison. Run the MADByTE 'generate_network' module to build the full chemical similarity association network, where edges use the higher of two pairwise similarity scores and non-reciprocal edges are removed. Restrict the network to the bioactive prefractions flagged by the screen (34 BioMAP-active prefractions against 15 clinically relevant bacterial pathogens) and overlay screening results as node attributes, encoding narrow-to-broad spectrum activity with color coding. Export the annotated network in graphML format and process it in Gephi with the Force Atlas 2 layout algorithm to reproduce hotspot visualizations (Figure 6A); identify hotspot clusters containing the most active prefractions for downstream NMR-guided isolation. Optionally run the MADByTE 'dereplication' module against a user-supplied internal library to annotate known scaffolds within the hotspot before isolation.

## Related tools

- **MADByTE** (Generates spin system features from TOCSY/HSQC data and builds the chemical similarity association network; its 'dereplication' module can optionally compare network spin systems against a user-supplied library) — https://github.com/liningtonlab/MADByTE
- **Gephi** (Visualization of the exported graphML network using the Force Atlas 2 algorithm to reveal bioactivity hotspots)

## Examples

```
conda activate madbyte; python madbyte_gui.py  # then run generate_network on spin system features, export graphML, open in Gephi (Force Atlas 2)
```

## Evaluation signals

- Exported graphML parses as a valid graph and contains exactly the 34 BioMAP-active prefraction nodes with bioactivity attributes attached
- The 2108D/2108E hotspot (from extract 2108, isolate RL12_176_HVF_A) appears as a connected cluster matching Figure 6A of the paper
- MADByTE filtering invariants hold: no signals remain in solvent/water regions (±0.02 ppm in 1H), the three illegitimate HSQC regions, or the <2.5 ppm TOCSY quadrant
- Network edges are reciprocal only, using the higher of the two pairwise similarity scores (A vs B or B vs A)
- Gephi layout reproduces narrow-to-broad spectrum color coding consistent with the paper's BioMAP encoding

## Limitations

- MADByTE will not identify every spin system in a given mixture
- Peak-picking algorithms perform poorly in the crowded shielded region (<2.5 ppm), which is why that quadrant is excluded
- Spectral overlap can yield multiple candidate 13C chemical shifts; the closest HSQC resonance value is used, which may be ambiguous
- Dereplication is limited by the lack of large-scale NMR repositories; current 'lightweight' NMR predictors do not have sufficient accuracy for this task

## Evidence
