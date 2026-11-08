---
name: compound-class-annotation-workflow
description: 'Use when you want chemical-class-level annotations for untargeted LC-MS/MS
  features rather than exact structures — determine molecular formulas with SIRIUS,
  compute CSI:FingerID fingerprints, and predict compound classes with CANOPUS and
  NPClassifier (superclass / class / pathway), producing a class-annotated feature
  table for chemical-inventory and enrichment analysis.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 5
  member_skills:
  - lc-msms-feature-detection
  - mass-spectrometry-feature-detection
  - lc-ms-feature-quantification-processing
  - lc-ms-feature-detection--e300bd20a9eef616
  - molecular-formula-reranking-zodiac
  - molecular-formula-determination-isotope-fragmentation
  - isotope-pattern-based-formula-ranking
  - molecular-formula-gibbs-sampling
  - csi-fingerprint-prediction
  - substructure-fingerprint-prediction
  - molecular-fingerprint-prediction
  - csi-fingerid-fingerprint-prediction
  - msms-spectra-canopus-class-prediction
  - classyfire-npclassifier-ontology-mapping
  - in-silico-class-prediction-interpretation
  - molecular-fingerprint-encoding--7b81bc09ce5106ca
  - mass-spectrometry-feature-table-processing
  - mass-spectrometry-feature-annotation-qc
  - insilico-ms2-annotation-sirius-canopus
  - batch-ms2-annotation-automation
  member_tools:
  - COSMIC
  - CSI:FingerID
  - SIRIUS 4
  - QIIME 2
  - Qemistree
  - SIRIUS
  - ZODIAC
  - q2-qemistree
  - GNPS
  - MZmine2
  - CANOPUS
  - ClassyFire
  - NPClassifier
  - NPLinker
  - ConCISE
  - Python
  coverage_gaps: []
  derived_from_workflows:
  - coll_npclassscore_cq
  - coll_molnetenhancer
  bound_by: perspicacite-semantic
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/metabolomics/v3
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# Compound-Class Annotation (SIRIUS formula -> CANOPUS / NPClassifier class)

## Summary

MS2 in, a chemical-class-annotated table out: SIRIUS molecular formula, molecular fingerprint, and CANOPUS / NPClassifier compound-class prediction per feature.


## When to use

Use when you want chemical-class-level annotations for untargeted LC-MS/MS features rather than exact structures — determine molecular formulas with SIRIUS, compute CSI:FingerID fingerprints, and predict compound classes with CANOPUS and NPClassifier (superclass / class / pathway), producing a class-annotated feature table for chemical-inventory and enrichment analysis.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — preprocess

**Goal:** raw mzML -> feature table + SIRIUS-flavour MS2 export

**EDAM operation:** http://edamontology.org/operation_3215

**Inputs:** mzML · **Outputs:** feature-table, mgf/sirius

**Candidate leaf skills:** `lc-msms-feature-detection` (primary), `mass-spectrometry-feature-detection`, `lc-ms-feature-quantification-processing`, `lc-ms-feature-detection--e300bd20a9eef616`

**Tools (primary):** COSMIC, CSI:FingerID, SIRIUS 4

**Other candidate tools:** ClassyFire, GNPS, MZmine2, QIIME 2, Qemistree, SIRIUS, KNIME, MS-FINDER, OpenMS, Optimus, MZmine, ProteoWizard, matchms

**Grounding:** 4 KB(s); DOIs: 10.1021/acscentsci.3c00800, 10.1038/nprot.2017.122, 10.1038/s41587-021-01045-9, 10.1038/s41589-020-00677-3

### Stage 2 — formula

**Goal:** MS2 spectra -> molecular formula (SIRIUS + ZODIAC re-ranking)

**EDAM operation:** http://edamontology.org/operation_3860

**Inputs:** mgf/sirius · **Outputs:** tsv

**Candidate leaf skills:** `molecular-formula-reranking-zodiac` (primary), `molecular-formula-determination-isotope-fragmentation`, `isotope-pattern-based-formula-ranking`, `molecular-formula-gibbs-sampling`

**Tools (primary):** CSI:FingerID, QIIME 2, Qemistree, SIRIUS, ZODIAC, q2-qemistree

**Other candidate tools:** COSMIC, SIRIUS 4, sirius-client-openAPI, OpenMS 2.4.0 (modified)

**Grounding:** 4 KB(s); DOIs: 10.1038/s41587-021-01045-9, 10.1038/s41589-020-00677-3, 10.1038/s41592-019-0344-8, 10.1038/s42256-020-00234-6

### Stage 3 — fingerprint

**Goal:** formula + MS2 -> molecular fingerprint (CSI:FingerID)

**EDAM operation:** http://edamontology.org/operation_3801

**Inputs:** tsv · **Outputs:** tsv

**Candidate leaf skills:** `csi-fingerprint-prediction` (primary), `substructure-fingerprint-prediction`, `molecular-fingerprint-prediction`, `csi-fingerid-fingerprint-prediction`

**Tools (primary):** CSI:FingerID, GNPS, MZmine2, QIIME 2, Qemistree, SIRIUS, q2-qemistree

**Other candidate tools:** ZODIAC, ClassyFire, AWS CLI, MSNovelist

**Grounding:** 2 KB(s); DOIs: 10.1038/s41589-020-00677-3, 10.1038/s41592-022-01486-3

### Stage 4 — classify

**Goal:** fingerprint -> compound class (CANOPUS / NPClassifier: superclass/class/pathway)

**EDAM operation:** http://edamontology.org/operation_0224

**Inputs:** tsv · **Outputs:** tsv

**Candidate leaf skills:** `msms-spectra-canopus-class-prediction` (primary), `classyfire-npclassifier-ontology-mapping`, `in-silico-class-prediction-interpretation`, `molecular-fingerprint-encoding--7b81bc09ce5106ca`

**Tools (primary):** CANOPUS, ClassyFire, GNPS, NPClassifier, NPLinker, SIRIUS

**Other candidate tools:** mibig_classifications, ConCISE, Python, TensorFlow, TensorFlow 2.3.0, docker, docker-compose

**Grounding:** 3 KB(s); DOIs: 10.1021/acs.jnatprod.1c00399, 10.1186/s40168-022-01444-3, 10.3390/metabo12121275

### Stage 5 — consolidate

**Goal:** consolidate formula + fingerprint + class into a class-annotated feature table

**EDAM operation:** http://edamontology.org/operation_3434

**Inputs:** feature-table, tsv · **Outputs:** tsv

**Candidate leaf skills:** `mass-spectrometry-feature-table-processing` (primary), `mass-spectrometry-feature-annotation-qc`, `insilico-ms2-annotation-sirius-canopus`, `batch-ms2-annotation-automation`

**Tools (primary):** ConCISE, GNPS, Python

**Other candidate tools:** CANOPUS, ClassyFire, SIRIUS, ENPKG, ENPKG (04_enpkg_sirius_canopus), MZmine, RDF, matchms, NPClassifier

**Grounding:** 2 KB(s); DOIs: 10.1021/acscentsci.3c00800, 10.3390/metabo12121275

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
