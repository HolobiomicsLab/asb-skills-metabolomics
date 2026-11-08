---
name: sirius-in-silico-formula-annotation
description: Use when molecular network nodes (e.g., from GNPS2 FBMN on MZmine .mgf/.csv
  inputs) have no spectral library match and no ChemWalker propagation, and a molecular-formula-level
  annotation is still needed.
when_to_use_negative:
- The spectrum already has a GNPS2 spectral library match or a ChemWalker-propagated
  annotation — SIRIUS is only the fallback for still-unannotated nodes
- The precursor is from an instrument class other than Q-TOF, since the workflow specifies
  SIRIUS's recommended Q-TOF instrument parameters
- Only spectral-level (library) annotation is required and no in silico formula layer
  is desired
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3332
tools:
- name: SIRIUS 5.8.5
  role: Computes the in silico formula/candidate annotation for nodes that failed
    library matching and ChemWalker propagation, using Q-TOF parameters and adducts
    [M+H]+, [M+K]+, [M+Na]+
- name: GNPS2 (FBMN workflow)
  role: 'First annotation layer: molecular networking and spectral pairing that determines
    which nodes remain for the SIRIUS layer'
- name: ChemWalker
  role: 'Second annotation layer: propagates annotations along molecular network topology
    (adduct [M+H]+, 15 ppm) via its GNPS2 interface; only nodes it leaves unannotated
    go to SIRIUS'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:bbd597ae102f038b537f5fe5d1f09c89e8534aa025b8dd32eb9950422f8f1ab5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/computational-chemical-biology/regression_filter
  tools:
  - ChemWalker
  - GNPS
  - GNPS2
  - GNPS2 (FBMN workflow)
  - MZmine
  - SIRIUS
  - SIRIUS 5.8.5
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# SIRIUS in silico formula annotation of unannotated MS/MS spectra

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A final annotation layer in which molecular formulas and compound candidates are computed in silico with SIRIUS 5.8.5 for LC-MS/MS spectra that remained unannotated after GNPS2 FBMN spectral pairing and ChemWalker propagation, applied here to list-only precursor ions from three actinomycete strains.

## When to use

Use when molecular network nodes (e.g., from GNPS2 FBMN on MZmine .mgf/.csv inputs) have no spectral library match and no ChemWalker propagation, and a molecular-formula-level annotation is still needed.

## When NOT to use

- The spectrum already has a GNPS2 spectral library match or a ChemWalker-propagated annotation — SIRIUS is only the fallback for still-unannotated nodes
- The precursor is from an instrument class other than Q-TOF, since the workflow specifies SIRIUS's recommended Q-TOF instrument parameters
- Only spectral-level (library) annotation is required and no in silico formula layer is desired

## Inputs

- Unannotated list-only fragmentation spectra (.mgf) exported from MZmine (precursors selected in the RegFilter scheduled precursor list but missed by DDA)
- MZmine-exported feature quantification table (.csv)
- Precursor m/z values and adduct candidates ([M+H]+, [M+K]+, [M+Na]+) for each unannotated node

## Outputs

- Best-ranked SIRIUS molecular formula/compound candidate per unannotated spectrum
- Per-strain annotation table compiling recovered annotations (BRA006: 43, BRA010: 46, BRA177: 47 list-only ion annotations)

## How to apply

First run the GNPS2 Feature-Based Molecular Networking workflow on the MZmine-exported feature table (.csv) and list-only fragmentation spectra (.mgf) to obtain spectral pairing annotations, then propagate annotations via ChemWalker through its GNPS2 interface using adduct [M+H]+ and 15 ppm tolerance. Nodes still lacking an annotation are exported and analyzed with SIRIUS version 5.8.5 configured with the recommended Q-TOF instrument parameters, requesting formulas from all databases and the adducts [M+H]+, [M+K]+, and [M+Na]+. The best-ranked SIRIUS candidate is assigned as the annotation for each spectrum, and per-strain results are compiled into a single table. This three-level scheme (library match → ChemWalker propagation → SIRIUS) recovered 43 annotations for BRA006, 46 for BRA010, and 47 for BRA177 list-only ions.

## Related tools

- **SIRIUS 5.8.5** (Computes the in silico formula/candidate annotation for nodes that failed library matching and ChemWalker propagation, using Q-TOF parameters and adducts [M+H]+, [M+K]+, [M+Na]+)
- **GNPS2 (FBMN workflow)** (First annotation layer: molecular networking and spectral pairing that determines which nodes remain for the SIRIUS layer)
- **ChemWalker** (Second annotation layer: propagates annotations along molecular network topology (adduct [M+H]+, 15 ppm) via its GNPS2 interface; only nodes it leaves unannotated go to SIRIUS)

## Evaluation signals

- Recovered list-only ion annotation counts per strain match the reported values: BRA006: 43, BRA010: 46, BRA177: 47
- SIRIUS is run only on nodes lacking both a GNPS2 spectral pairing match and a ChemWalker-propagated annotation
- Every SIRIUS run uses the documented settings: version 5.8.5, Q-TOF instrument parameters, formulas from all databases, adducts [M+H]+, [M+K]+, [M+Na]+
- The assigned annotation per spectrum is the best-ranked SIRIUS candidate, and all per-strain results are consolidated into a single table

## Limitations

- SIRIUS provides in silico (formula/candidate-level) annotations, not spectral-library-confirmed identifications; confidence depends on the preceding network and ChemWalker layers
- Coverage is bounded by which list-only spectra exist: only ions whose precursor was selected in the RegFilter SPL but missed by DDA are annotated, and total features detected in full scan mode exceed those detected in DDA mode
- DDA's stochastic nature means ions of similar intensity may or may not be fragmented, so some SPL features have no MS/MS spectrum to annotate

## Evidence

- [results] Recovered annotation counts per strain: [evidence span withheld — anchor status: not_found]
