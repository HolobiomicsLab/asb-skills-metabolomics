---
name: sirius-denovo-structure-elucidation-workflow
description: 'Use when you have MS/MS for unknown features (a SIRIUS-flavour mgf /
  .ms) and want de novo annotation without a spectral match — molecular formula (SIRIUS+ZODIAC),
  structure (CSI:FingerID + COSMIC), compound class (CANOPUS), optionally against
  a custom database, filtered by confidence. Library-FREE by design.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 5
  member_skills:
  - molecular-formula-reranking-zodiac
  - molecular-formula-determination-isotope-fragmentation
  - molecular-formula-gibbs-sampling
  - isotope-pattern-based-formula-ranking
  - cheminformatics-structure-standardization
  - cross-dataset-molecule-deduplication
  - structure-database-size-validation
  - structural-descriptor-calculation
  - fingerprint-based-candidate-re-ranking
  - csi-fingerid-fingerprint-prediction
  - structure-candidate-ranking-comparison
  - confidence-score-fdr-estimation
  - compound-class-prediction-canopus
  - classyfire-npclassifier-ontology-mapping
  - classyfire-superclass-frequency-tabulation
  - classyfire-superclass-annotation-analysis
  - zodiac-posterior-score-threshold-analysis
  - annotation-accuracy-benchmarking
  - mass-spectrometry-annotation-file-parsing
  member_tools:
  - CSI:FingerID
  - QIIME 2
  - Qemistree
  - SIRIUS
  - ZODIAC
  - q2-qemistree
  - CDK
  - COCONUT compiler
  - MongoDB
  - RDKit
  - AWS CLI
  - MSNovelist
  - CANOPUS
  - MZmine
  - Python 3.10
  - microbeMASST
  - OpenMS
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

# SIRIUS De Novo Structure Elucidation

## Summary

End-to-end de novo elucidation with SIRIUS 6: formula, structure and class prediction for novel/unannotated chemistry, with confidence-based filtering. No spectral library.


## When to use

Use when you have MS/MS for unknown features (a SIRIUS-flavour mgf / .ms) and want de novo annotation without a spectral match — molecular formula (SIRIUS+ZODIAC), structure (CSI:FingerID + COSMIC), compound class (CANOPUS), optionally against a custom database, filtered by confidence. Library-FREE by design.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — formula

**Goal:** molecular formula determination (SIRIUS + ZODIAC)

**EDAM operation:** http://edamontology.org/operation_3860

**Inputs:** mgf/sirius · **Outputs:** tsv

**Candidate leaf skills:** `molecular-formula-reranking-zodiac` (primary), `molecular-formula-determination-isotope-fragmentation`, `molecular-formula-gibbs-sampling`, `isotope-pattern-based-formula-ranking`

**Tools (primary):** CSI:FingerID, QIIME 2, Qemistree, SIRIUS, ZODIAC, q2-qemistree

**Other candidate tools:** COSMIC, SIRIUS 4, OpenMS 2.4.0 (modified), sirius-client-openAPI

**Grounding:** 4 KB(s); DOIs: 10.1038/s41587-021-01045-9, 10.1038/s41589-020-00677-3, 10.1038/s41592-019-0344-8, 10.1038/s42256-020-00234-6

### Stage 2 — custom_db  [OPTIONAL]

**Goal:** (optional) build a custom structure database for the search space

**EDAM operation:** http://edamontology.org/operation_3434

**Inputs:** smiles · **Outputs:** tsv

**Candidate leaf skills:** `cheminformatics-structure-standardization` (primary), `cross-dataset-molecule-deduplication`, `structure-database-size-validation`, `structural-descriptor-calculation`

**Tools (primary):** CDK, COCONUT compiler, MongoDB, RDKit

**Other candidate tools:** RDKit (v2022.03.3), COSMIC, SIRIUS 4, SmiLib v2.0, Broad Institute Drug Repurposing Hub, Dictionary of Natural Products, DrugBank, DrugCentral, LOTUS, Prefect 2, PubChem, pip

**Grounding:** 4 KB(s); DOIs: 10.1038/s41467-023-39279-7, 10.1038/s41587-021-01045-9, 10.1038/s41592-025-02813-0, 10.1186/s13321-020-00478-9

### Stage 3 — structure

**Goal:** structure prediction (CSI:FingerID + COSMIC)

**EDAM operation:** http://edamontology.org/operation_3801

**Inputs:** tsv · **Outputs:** tsv

**Candidate leaf skills:** `fingerprint-based-candidate-re-ranking` (primary), `csi-fingerid-fingerprint-prediction`, `structure-candidate-ranking-comparison`, `confidence-score-fdr-estimation`

**Tools (primary):** AWS CLI, CSI:FingerID, MSNovelist, SIRIUS

**Other candidate tools:** HILIC-Q Exactive HF mass spectrometry, COSMIC, CSI:FingerID 1.2.0, SIRIUS 4

**Grounding:** 3 KB(s); DOIs: 10.1021/acs.analchem.8b04698, 10.1038/s41587-021-01045-9, 10.1038/s41592-022-01486-3

### Stage 4 — compound_class

**Goal:** compound class prediction (CANOPUS / NPClassifier)

**EDAM operation:** http://edamontology.org/operation_3803

**Inputs:** tsv · **Outputs:** tsv

**Candidate leaf skills:** `compound-class-prediction-canopus` (primary), `classyfire-npclassifier-ontology-mapping`, `classyfire-superclass-frequency-tabulation`, `classyfire-superclass-annotation-analysis`

**Tools (primary):** CANOPUS, MZmine, Python 3.10, SIRIUS, microbeMASST

**Other candidate tools:** ClassyFire, GNPS, NPClassifier, NPLinker, mibig_classifications, MolNetEnhancer, FIDDLE, Orbitrap, Q-TOF

**Grounding:** 3 KB(s); DOIs: 10.1038/s41467-025-66060-9, 10.1038/s41564-023-01575-9, 10.1186/s40168-022-01444-3

### Stage 5 — confidence_filter

**Goal:** filter annotations by ZODIAC / COSMIC confidence

**EDAM operation:** http://edamontology.org/operation_3695

**Inputs:** tsv, tsv · **Outputs:** tsv

**Candidate leaf skills:** `zodiac-posterior-score-threshold-analysis` (primary), `annotation-accuracy-benchmarking`, `mass-spectrometry-annotation-file-parsing`

**Tools (primary):** OpenMS, SIRIUS, ZODIAC

**Other candidate tools:** OpenMS 2.4.0 (modified)

**Grounding:** 1 KB(s); DOIs: 10.1038/s42256-020-00234-6

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
