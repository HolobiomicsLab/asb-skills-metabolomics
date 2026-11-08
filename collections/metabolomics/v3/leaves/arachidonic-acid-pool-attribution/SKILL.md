---
name: arachidonic-acid-pool-attribution
description: Use when targeted lipidomics abundance data across multiple platelet
  agonist stimulations (0.01/1 U/mL thrombin, 1/5 µg/mL CRP vs unstimulated) must
  be attributed to a lipid class as the dominant AA reservoir, testing PI depletion
  against PC/PE stability.
when_to_use_negative:
- The experiment is an untargeted lipidome-wide screening effort rather than a targeted
  panel of functionally related lipids (the paper explicitly targets distinct molecule
  sets 'in contrast to screening approaches')
- No unstimulated control condition is measured, so per-species log2 fold-change baselines
  cannot be computed
- The question requires resolving stereoisomers or unvalidated fragment identifications,
  which LipidCreator does not address
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3353
tools:
- name: LipidCreator
  role: Designs the targeted transition lists and fragment libraries for the PRM/MRM
    assays that generate the AA-pool lipid abundance data; source releases at https://github.com/lifs-tools/lipidcreator/releases
  repo: github.com/lifs-tools/lipidcreator
- name: Skyline
  role: Hosts LipidCreator as a plugin and performs DIA/PRM method preparation and
    data analysis for the targeted lipidomics measurements
  repo: https://skyline.ms/project/home/software/Skyline/begin.view
- name: MSStats
  role: Statistical comparison of quantified lipid abundances across stimulation conditions
    supporting the fold-change and effect-size estimates
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:f74e9d27f664c8ab67eab5d4d25eaf107eec02d659825ddc9c88db72b096cee4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidCreator
  - MSStats
  - Skyline
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3353
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# arachidonic-acid-pool-attribution

## Summary

Reconstructs the platelet lipid-signaling correlation network and per-headgroup fold-change statistics to attribute the arachidonic acid (AA) pool to its dominant reservoir lipid class (PI), distinguishing PI-cluster depletion from PC/PE stability across thrombin and CRP stimulations.

## When to use

Use when targeted lipidomics abundance data across multiple platelet agonist stimulations (0.01/1 U/mL thrombin, 1/5 µg/mL CRP vs unstimulated) must be attributed to a lipid class as the dominant AA reservoir, testing PI depletion against PC/PE stability.

## When NOT to use

- The experiment is an untargeted lipidome-wide screening effort rather than a targeted panel of functionally related lipids (the paper explicitly targets distinct molecule sets 'in contrast to screening approaches')
- No unstimulated control condition is measured, so per-species log2 fold-change baselines cannot be computed
- The question requires resolving stereoisomers or unvalidated fragment identifications, which LipidCreator does not address

## Inputs

- Targeted lipid abundance matrix (rows: PI, PC, PE, PS, PG and AA-containing lipid species; columns: unstimulated control plus 0.01 U/mL thrombin, 1 U/mL thrombin, 1 µg/mL CRP, 5 µg/mL CRP)
- Lipid-to-headgroup mapping for PI, PC, PE, PS, PG classes
- Identification of AA-containing PI species (e.g. PI 38:4) with fatty-acyl composition

## Outputs

- Per-species log2 fold-change table for each stimulation condition relative to the unstimulated control
- Mean absolute log2 fold-change per lipid class (PI, PC, PE, PS, PG) per condition
- Spearman correlation network with edges filtered at |rho| > 0.7 and FDR-adjusted p < 0.05, showing lipid-class clusters
- AA-reservoir attribution ranking of lipid classes and condition-wise effect sizes ranking CRP vs thrombin

## How to apply

Load the lipid abundance matrix in pandas with species from PI, PC, PE, PS, PG and AA-containing lipids as rows and the four stimulation conditions plus the unstimulated control as columns. Compute per-species log2 fold-changes for each condition relative to the unstimulated control so up- and down-regulation are symmetric. Group species by headgroup and compute mean absolute log2 fold-change per class per condition to test PI reduction vs PC/PE stability. Build the correlation network by computing Spearman rank correlations (with p-values) across all lipid species pairs, keeping only edges with |rho| > 0.7 and FDR-adjusted p < 0.05. Within PI, compare the relative abundance of AA-containing species (e.g. PI 38:4 and related) against other PI species to confirm PI as the dominant AA reservoir, and rank CRP vs thrombin regulatory strength by condition-wise effect sizes. Accept the analysis only if PI's mean |log2FC| substantially exceeds PC/PE, PI carries the largest AA-containing fraction, CRP-induced |log2FC| exceeds thrombin at matched stimulus intensity, and the network reproduces PI-species clustering.

## Related tools

- **LipidCreator** (Designs the targeted transition lists and fragment libraries for the PRM/MRM assays that generate the AA-pool lipid abundance data; source releases at https://github.com/lifs-tools/lipidcreator/releases) — github.com/lifs-tools/lipidcreator
- **Skyline** (Hosts LipidCreator as a plugin and performs DIA/PRM method preparation and data analysis for the targeted lipidomics measurements) — https://skyline.ms/project/home/software/Skyline/begin.view
- **MSStats** (Statistical comparison of quantified lipid abundances across stimulation conditions supporting the fold-change and effect-size estimates)

## Examples

```
import pandas as pd, numpy as np; df = pd.read_csv('platelet_lipidome.csv', index_col=0); lfc = np.log2(df[['t0.01U','t1U','crp1','crp5']].div(df['ctrl'], axis=0))
```

## Evaluation signals

- Re-derived mean |log2FC| for PI species is substantially larger than for PC/PE in every stimulation condition (directionally consistent with 'dramatically reduced' vs 'unaffected')
- PI carries the largest AA-containing fraction, with PI 38:4 and related species dominating within the PI class
- CRP-induced mean |log2FC| exceeds thrombin-induced |log2FC| at matched stimulus intensities
- The reconstructed Spearman network reproduces PI-species clustering among the flagged edges (|rho| > 0.7, FDR-adjusted p < 0.05)
- Every reported correlation edge carries an FDR-adjusted p-value below the 0.05 acceptance threshold

## Limitations

- LipidCreator does not provide a dedicated scoring algorithm for false discovery rates and intentionally does not address coeluting/isobaric lipids or peak area integration
- Retention time, elution order, and isotope-labeled standards must be considered to avoid lipid mis-annotations
- Due to lack of isotope-coded standards, no large-scale methods exist to cope with stereoisomers
- LipidCreator collision-energy models are currently exclusive to lipid mediators; other classes require more structurally diverse lipid standards

## Evidence

- [abstract] Targeted assays underpin the AA-pool attribution: "We validate LipidCreator assays computationally and analytically and prove that it is capable to generate large targeted experiments to analyze blood and to dissect lipid-signaling pathways such as"
- [methods] Instrument modes used for the measurements: "The LC was coupled to a Q-Exactive HF (QEx HF) mass spectrometer (Thermo Scientific, Bremen, Germany) or a QTRAP 6500"
- [discussion] Tool integration for assay development: "LipidCreator is well integrated into Skyline for small molecules, which makes it a vendor-independent software for fast assay development in targeted lipidomics."
- [discussion] FDR handling limitation of the assay tool: "LipidCreator does not provide a dedicated scoring algorithm to determine false discovery rates but supports a knowledge-based selection of transition lists."
- [readme] README description of LipidCreator's role: "It can be used to create user-defined target lists and fragment libraries for PRM and MRM experiments in Skyline."
