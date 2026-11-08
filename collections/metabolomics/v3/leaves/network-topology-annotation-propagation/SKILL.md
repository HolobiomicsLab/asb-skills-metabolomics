---
name: network-topology-annotation-propagation
description: Use when an FBMN molecular network has spectral pairing (GNPS/GNPS2 library)
  matches for only a subset of nodes and the remaining MS/MS spectra still need chemical
  annotations.
when_to_use_negative:
- When the node already has a direct spectral library match in the GNPS2 library —
  propagation adds nothing beyond the first-level annotation
- When the input is raw vendor data or unprocessed MS1/MS2 centroids rather than an
  MZmine-preprocessed feature table and .mgf MS/MS spectra
- When the analysis goal is discovering entirely novel spectra without structural
  analogs in the network — propagation requires connected annotated neighbors
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: GNPS2 (FBMN workflow)
  role: Builds the Feature Based Molecular Network and provides the graphical interface
    hosting ChemWalker for topology-based annotation propagation
- name: ChemWalker
  role: Propagates first-level spectral pairing annotations along the molecular network
    topology to annotate nodes without direct library matches (adduct [M+H]+, 15 ppm
    tolerance)
- name: SIRIUS
  role: 'Third annotation layer: in silico molecular formula and candidate assignment
    for spectra left unannotated after propagation, version 5.8.5 with Q-TOF parameters
    and adducts [M+H]+/[M+K]+/[M+Na]+'
- name: MZmine
  role: Upstream preprocessing that exports the feature table and .mgf MS/MS spectra
    consumed by the FBMN/propagation pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:713fe8a91e8cabe09bfa522f3c87cce6ceaa9d9d5be31835cd2720b5d7e99517
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
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# network-topology-annotation-propagation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate spectral-library-unmatched nodes in a Feature Based Molecular Network by propagating known annotations along the molecular network topology, then filling remaining gaps with SIRIUS in silico annotation — the second and third annotation layers in this RegFilter LC-MS/MS workflow for actinomycete extracts.

## When to use

Use when an FBMN molecular network has spectral pairing (GNPS/GNPS2 library) matches for only a subset of nodes and the remaining MS/MS spectra still need chemical annotations.

## When NOT to use

- When the node already has a direct spectral library match in the GNPS2 library — propagation adds nothing beyond the first-level annotation
- When the input is raw vendor data or unprocessed MS1/MS2 centroids rather than an MZmine-preprocessed feature table and .mgf MS/MS spectra
- When the analysis goal is discovering entirely novel spectra without structural analogs in the network — propagation requires connected annotated neighbors

## Inputs

- MZmine-exported feature quantification table (.csv) from the GNPS-FBMN export step (rows filtered to only with MS2)
- MZmine-exported MS/MS spectra in .mgf format
- FBMN molecular network with spectral pairing (library) annotations on GNPS2

## Outputs

- Network-propagated annotations per node (ChemWalker output via GNPS2 interface)
- Per-spectrum in silico formula/candidate annotations ranked by SIRIUS 5.8.5
- Compiled annotation table per strain (e.g. recovered list-only ion annotations: BRA006 = 43, BRA010 = 46, BRA177 = 47)

## How to apply

Import the MZmine-exported feature table and MS/MS .mgf spectra into the GNPS2 Feature Based Molecular Networking (FBMN) workflow to build the molecular network and obtain first-level spectral pairing annotations. For nodes without library matches, use ChemWalker through its graphical interface on GNPS2 to propagate annotations along the molecular network topology; set the criteria adduct to [M+H]+ and a mass tolerance of 15 ppm. For nodes still unannotated after ChemWalker, run SIRIUS version 5.8.5 with Q-TOF instrument parameters, searching formulas from all databases with adducts [M+H]+, [M+K]+, and [M+Na]+, and assign the best-ranked candidate to each spectrum. Rationale: propagation exploits structural relatedness of co-eluting features connected in the network so an annotation of one node extends to close analogs that lack their own MS2 library match.

## Related tools

- **GNPS2 (FBMN workflow)** (Builds the Feature Based Molecular Network and provides the graphical interface hosting ChemWalker for topology-based annotation propagation)
- **ChemWalker** (Propagates first-level spectral pairing annotations along the molecular network topology to annotate nodes without direct library matches (adduct [M+H]+, 15 ppm tolerance))
- **SIRIUS** (Third annotation layer: in silico molecular formula and candidate assignment for spectra left unannotated after propagation, version 5.8.5 with Q-TOF parameters and adducts [M+H]+/[M+K]+/[M+Na]+)
- **MZmine** (Upstream preprocessing that exports the feature table and .mgf MS/MS spectra consumed by the FBMN/propagation pipeline)

## Evaluation signals

- Every node in the molecular network has an annotation from at least one of the three layers (library pairing, ChemWalker propagation, SIRIUS)
- Propagated annotations are consistent with the propagation criteria: adduct [M+H]+ and mass tolerance within 15 ppm
- SIRIUS candidates are assigned the best-ranked candidate for each spectrum, with Q-TOF instrument setting and adducts [M+H]+/[M+K]+/[M+Na]+ explicitly recorded
- Per-strain annotation counts are reproducible and comparable to reported values (BRA006: 43, BRA010: 46, BRA177: 47 recovered annotations)
- The FBMN import used the MZmine 'Export/Submit to GNPS-FBMN' CSV filtered to rows only with MS2

## Limitations

- ChemWalker propagation inherits uncertainty from the seed annotation — a wrong library match propagates along the topology; only an [M+H]+ adduct with 15 ppm tolerance is used as propagation criteria
- SIRIUS annotations are in silico formulas/candidates, not confirmed structures; the paper cross-references KEGG to attach biological relevance
- Propagated coverage depends on network topology: isolated nodes with no annotated neighbors cannot gain annotations from ChemWalker

## Evidence

- [methods] For nodes without spectral library matches, propagate annotations along the molecular network topology using ChemWalker via its GNPS2 graphical interface, with criteria adduct [M+H]+ and tolerance 15 ppm.: [evidence span withheld — anchor status: not_found]
- [methods] run SIRIUS version 5.8.5 with the recommended Q-TOF instrument parameters, using formulas from all databases and adducts [M+H]+, [M+K]+, and [M+Na]+.: [evidence span withheld — anchor status: not_found]
- [results] 43 annotations were recovered for BRA006, 46 for BRA010, and 47 for BRA177.: [evidence span withheld — anchor status: not_found]
