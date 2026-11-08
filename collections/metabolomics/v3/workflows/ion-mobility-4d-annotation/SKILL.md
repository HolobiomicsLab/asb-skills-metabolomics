---
name: ion-mobility-4d-annotation-workflow
description: 'Use when you have ion-mobility LC-IMS-MS/MS data (e.g. timsTOF / PASEF)
  and want CCS-aware annotations — 4D feature extraction with collision cross section,
  CCS calibration and filtering, CCS-aware library matching, and (optional) networking.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 4
  member_skills:
  - ion-mobility-mass-spectrometry-data-processing
  - lc-ims-msms-feature-detection
  - ion-mobility-ms-data-processing
  - ion-mobility-separation-profile-matching
  - collision-cross-section-calibration
  - ion-mobility-ccs-calibration
  - collision-cross-section-calculation
  - reduced-collision-cross-section-correction
  - ccs-based-compound-identification
  - msms-longitudinal-fragment-profiling-scoring
  - feature-based-molecular-networking--7b025a18b065ae1c
  - feature-based-molecular-networking--f72759146010fbc6
  - modified-cosine-similarity-networking
  - feature-based-molecular-networking--a1d4da5bf30d5f86
  member_tools:
  - DEIMoS
  - MS-DIAL
  - Python
  - SVR
  - GNN-RT
  - SigmaCCS
  - GNPS
  - MZmine
  - XCMS
  coverage_gaps: []
  derived_from_workflows: []
  bound_by: perspicacite-semantic
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/metabolomics/v3
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# Ion Mobility (4D LC-IMS-MS/MS) Annotation

## Summary

End-to-end 4D ion-mobility annotation: extract CCS-resolved features, calibrate CCS, and annotate with collision-cross-section-aware matching.


## When to use

Use when you have ion-mobility LC-IMS-MS/MS data (e.g. timsTOF / PASEF) and want CCS-aware annotations — 4D feature extraction with collision cross section, CCS calibration and filtering, CCS-aware library matching, and (optional) networking.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — preprocess_4d

**Goal:** 4D LC-IMS-MS/MS feature extraction (with CCS)

**EDAM operation:** http://edamontology.org/operation_3215

**Inputs:** mzML · **Outputs:** feature-table, mgf

**Candidate leaf skills:** `ion-mobility-mass-spectrometry-data-processing` (primary), `lc-ims-msms-feature-detection`, `ion-mobility-ms-data-processing`, `ion-mobility-separation-profile-matching`

**Tools (primary):** DEIMoS, MS-DIAL, Python, SVR

**Other candidate tools:** scipy, Agilent ESI-L Tune Mix, MOCCal, NIST SRM 1950, Waters SYNAPT XS, poly-dl-alanine

**Grounding:** 2 KB(s); DOIs: 10.1021/acs.analchem.1c05017, 10.1021/acs.analchem.3c04290

### Stage 2 — ccs_calibration

**Goal:** collision cross section calibration + filtering

**EDAM operation:** http://edamontology.org/operation_3695

**Inputs:** feature-table · **Outputs:** feature-table

**Candidate leaf skills:** `collision-cross-section-calibration` (primary), `ion-mobility-ccs-calibration`, `collision-cross-section-calculation`, `reduced-collision-cross-section-correction`

**Tools (primary):** DEIMoS, Python

**Other candidate tools:** AutoCCS, conda, pip, Python 3.7

**Grounding:** 2 KB(s); DOIs: 10.1021/acs.analchem.1c05017, 10.1093/bioinformatics/btab429

### Stage 3 — ccs_library_match

**Goal:** CCS-aware spectral / library annotation

**EDAM operation:** http://edamontology.org/operation_3631

**Inputs:** mgf, feature-table · **Outputs:** tsv

**Candidate leaf skills:** `ccs-based-compound-identification` (primary), `msms-longitudinal-fragment-profiling-scoring`

**Tools (primary):** GNN-RT, SigmaCCS

**Other candidate tools:** 001-007.ipynb, Jaccard similarity algorithm (optimized), Jupyter Notebook, Python, Python 3.8.13

**Grounding:** 2 KB(s); DOIs: 10.1021/acs.analchem.5c01414, 10.1038/s42004-023-00939-w

### Stage 4 — networking  [OPTIONAL]

**Goal:** (optional) molecular networking of IM-resolved features

**EDAM operation:** http://edamontology.org/operation_3432

**Inputs:** mgf, feature-table, tsv · **Outputs:** graphml

**Candidate leaf skills:** `feature-based-molecular-networking--7b025a18b065ae1c` (primary), `feature-based-molecular-networking--f72759146010fbc6`, `modified-cosine-similarity-networking`, `feature-based-molecular-networking--a1d4da5bf30d5f86`

**Tools (primary):** GNPS, MS-DIAL, MZmine, XCMS

**Other candidate tools:** CANOPUS, ClassyFire, ConCISE, Python, SIRIUS, GNPS2, ms1-id, ChemWalker, MSConvert, MZmine 2.53, SIRIUS 5.8.5

**Grounding:** 4 KB(s); DOIs: 10.1038/s41467-021-23953-9, 10.1101/2024.08.07.607073, 10.1101/2024.10.14.618269, 10.3390/metabo12121275

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
