---
name: feature-based-molecular-networking-and-propagation-workflow
description: 'Use when you have untargeted LC-MS/MS MS2 data and want to spread a
  handful of confident annotations across whole molecular families — build a feature-based
  molecular network, seed it with spectral-library and SIRIUS/CANOPUS annotations,
  then propagate compound classes and analogue annotations across network components
  (MolNetEnhancer / network annotation propagation) so unannotated nodes inherit chemically-plausible
  identities.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 6
  member_skills:
  - lc-ms-peak-detection-and-feature-alignment
  - lcms-peak-detection-preprocessing
  - untargeted-metabolomics-preprocessing
  - lc-msms-feature-detection-and-alignment
  - feature-based-molecular-networking--7b025a18b065ae1c
  - modified-cosine-similarity-networking
  - feature-based-molecular-networking--a1d4da5bf30d5f86
  - msms-spectral-similarity-molecular-networking
  - modified-cosine-spectral-matching
  - spectral-library-matching--c3bacc8572caec82
  - spectral-library-matching--599e7745b946f0de
  - spectral-library-matching--c9ea083859d5db3b
  - msms-spectra-canopus-class-prediction
  - canopus-chemical-class-prediction
  - consensus-classification-propagation
  - in-silico-class-prediction-interpretation
  - molecular-network-node-annotation-mapping
  - chemical-class-propagation-onto-networks
  - substructure-annotation-of-molecular-families
  - molecular-network-annotation-propagation--3c622bace7b9313a
  - ms-feature-annotation-table-generation
  - molecular-network-construction-ms2-spectra
  - mass-spectrometry-feature-annotation-qc
  - mass-spectrometry-molecular-networking
  member_tools:
  - MS2LDA.Preprocessing.generate_corpus
  - MS2LDA.Preprocessing.load_and_clean
  - RMassBank
  - XCMS
  - GNPS
  - MS-DIAL
  - MZmine
  - GNPS (Global Natural Product Social Molecular Networking)
  - Manuscript_GNPS_Drug_Library
  - microbe_masst (domainMASSTs)
  - CANOPUS
  - ClassyFire
  - NPClassifier
  - NPLinker
  - SIRIUS
  - DEREPLICATOR
  - MolNetEnhancer
  - Network Annotation Propagation
  - molconvert
  - pyMolNetEnhancer
  - ConCISE
  - Python
  coverage_gaps: []
  derived_from_workflows:
  - coll_molnetenhancer
  - coll_npclassscore_cq
  - spec2vec_grounded
  - coll_ms2deepscore
  bound_by: perspicacite-semantic
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/metabolomics/v3
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# Feature-Based Molecular Networking with Annotation Propagation

## Summary

MS2 in, a network-propagated annotation table out: molecular networking, seed annotation, chemical-class assignment, and topology-driven propagation of annotations and classes across molecular families.


## When to use

Use when you have untargeted LC-MS/MS MS2 data and want to spread a handful of confident annotations across whole molecular families — build a feature-based molecular network, seed it with spectral-library and SIRIUS/CANOPUS annotations, then propagate compound classes and analogue annotations across network components (MolNetEnhancer / network annotation propagation) so unannotated nodes inherit chemically-plausible identities.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — preprocess

**Goal:** raw mzML -> aligned feature table + MS2 exports (GNPS-FBMN mgf + SIRIUS mgf)

**EDAM operation:** http://edamontology.org/operation_3215

**Inputs:** mzML · **Outputs:** feature-table, mgf/gnps-fbmn, mgf/sirius

**Candidate leaf skills:** `lc-ms-peak-detection-and-feature-alignment` (primary), `lcms-peak-detection-preprocessing`, `untargeted-metabolomics-preprocessing`, `lc-msms-feature-detection-and-alignment`

**Tools (primary):** MS2LDA.Preprocessing.generate_corpus, MS2LDA.Preprocessing.load_and_clean, RMassBank, XCMS

**Other candidate tools:** R, apLCMS, doParallel, foreach, mzR, splines, OpenMS, Python, Snakemake, ThermoRawFileParser, UmetaFlow, pyOpenMS, MZmine2, Optimus

**Grounding:** 4 KB(s); DOIs: 10.1021/acs.jnatprod.7b00737, 10.1038/s41598-020-70850-0, 10.1073/pnas.1608041113, 10.1186/s13321-023-00724-w

### Stage 2 — network

**Goal:** MS2 spectra -> molecular family graph (modified cosine; GNPS-style components)

**EDAM operation:** http://edamontology.org/operation_3214

**Inputs:** mgf/gnps-fbmn · **Outputs:** graphml, tsv

**Candidate leaf skills:** `feature-based-molecular-networking--7b025a18b065ae1c` (primary), `modified-cosine-similarity-networking`, `feature-based-molecular-networking--a1d4da5bf30d5f86`, `msms-spectral-similarity-molecular-networking`

**Tools (primary):** GNPS, MS-DIAL, MZmine, XCMS

**Other candidate tools:** GNPS2, ms1-id, ChemWalker, MSConvert, MZmine 2.53, SIRIUS 5.8.5, Cytoscape, MSConvert (ProteoWizard)

**Grounding:** 4 KB(s); DOIs: 10.1038/s41467-021-23953-9, 10.1038/s41596-020-0317-5, 10.1101/2024.08.07.607073, 10.1101/2024.10.14.618269

### Stage 3 — seed_annotate

**Goal:** MS2 spectra -> seed spectral-library annotations to propagate from

**EDAM operation:** http://edamontology.org/operation_3631

**Inputs:** mgf/gnps-fbmn · **Outputs:** tsv

**Candidate leaf skills:** `modified-cosine-spectral-matching` (primary), `spectral-library-matching--c3bacc8572caec82`, `spectral-library-matching--599e7745b946f0de`, `spectral-library-matching--c9ea083859d5db3b`

**Tools (primary):** GNPS (Global Natural Product Social Molecular Networking), Manuscript_GNPS_Drug_Library, microbe_masst (domainMASSTs)

**Other candidate tools:** ChemWalker, GNPS, GNPS library, GNPS2, GNPS2 (FBMN workflow), MZmine, MZmine 2.53, RegFilter, MS2DECIDE, Spec2Vec, gensim (Word2Vec), matchms, spec2vec_gnps_data_analysis, tissueMASST

**Grounding:** 4 KB(s); DOIs: 10.1038/s41467-025-65993-5, 10.1101/2024.08.07.607073, 10.1101/2025.04.28.651123, 10.1371/journal.pcbi.1008724

### Stage 4 — class_annotate

**Goal:** MS2 spectra -> molecular formula + chemical class (SIRIUS / CANOPUS / NPClassifier)

**EDAM operation:** http://edamontology.org/operation_3860

**Inputs:** mgf/sirius · **Outputs:** tsv

**Candidate leaf skills:** `msms-spectra-canopus-class-prediction` (primary), `canopus-chemical-class-prediction`, `consensus-classification-propagation`, `in-silico-class-prediction-interpretation`

**Tools (primary):** CANOPUS, ClassyFire, GNPS, NPClassifier, NPLinker, SIRIUS

**Other candidate tools:** ENPKG, ConCISE, Python

**Grounding:** 3 KB(s); DOIs: 10.1021/acscentsci.3c00800, 10.1186/s40168-022-01444-3, 10.3390/metabo12121275

### Stage 5 — propagate

**Goal:** spread seed annotations + chemical classes across molecular families

**EDAM operation:** http://edamontology.org/operation_3434

**Inputs:** graphml, tsv, tsv · **Outputs:** tsv

**Candidate leaf skills:** `molecular-network-node-annotation-mapping` (primary), `chemical-class-propagation-onto-networks`, `substructure-annotation-of-molecular-families`, `molecular-network-annotation-propagation--3c622bace7b9313a`

**Tools (primary):** ClassyFire, DEREPLICATOR, GNPS, MolNetEnhancer, Network Annotation Propagation, molconvert, pyMolNetEnhancer

**Other candidate tools:** Cytoscape, MS2LDA, RMolNetEnhancer

**Grounding:** 2 KB(s); DOIs: 10.1038/s41587-020-0700-3, 10.3390/metabo9070144

### Stage 6 — consolidate

**Goal:** consolidate network family + seed + class + propagated annotations into one table

**EDAM operation:** http://edamontology.org/operation_3434

**Inputs:** feature-table, graphml, tsv · **Outputs:** tsv

**Candidate leaf skills:** `ms-feature-annotation-table-generation` (primary), `molecular-network-construction-ms2-spectra`, `mass-spectrometry-feature-annotation-qc`, `mass-spectrometry-molecular-networking`

**Tools (primary):** CANOPUS, ClassyFire, ConCISE, GNPS, NPClassifier, Python

**Other candidate tools:** MassTools, Metaboseek, igraph, xcms, SIRIUS

**Grounding:** 2 KB(s); DOIs: 10.1038/s41467-022-28391-9, 10.3390/metabo12121275

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
