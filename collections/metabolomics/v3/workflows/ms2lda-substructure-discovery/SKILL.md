---
name: ms2lda-substructure-discovery-workflow
description: 'Use when you want to discover shared substructures (Mass2Motifs) across
  a corpus of MS2 spectra via LDA topic modeling — convert MS/MS spectra into a bag-of-fragments
  document corpus, run MS2LDA (Latent Dirichlet Allocation) to infer recurring fragmentation
  motifs, annotate motifs against MotifDB / Spec2Vec embeddings, and map motifs back
  onto molecular-network features for substructure-level annotation of an untargeted
  LC-MS/MS dataset.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 5
  member_skills:
  - msms-noise-filtering
  - mgf-spectra-noise-filtering-and-preprocessing
  - mass-spectrometry-spectrum-preprocessing--99e25f8cc3a2c48f
  - mass-spectra-preprocessing-and-binning
  - latent-dirichlet-allocation-topic-modeling
  - lda-topic-model-visualization
  - mass-spectrometry-fragmentation-motif-visualization
  - msms-fragment-bag-of-words-corpus-construction
  - mass2motif-spectral-matching-annotation
  - motif-annotation-agreement-comparison
  - combinatorial-fragmentation-substructure-annotation
  - molecular-network-mass2motif-mapping
  - molecular-network-motif-annotation
  - molecular-network-annotation-propagation--3c622bace7b9313a
  - mass-spectral-molecular-network-motif-mapping
  - ms2lda-motif-summary-parsing
  - fragment-pseudo-spectrum-representation-of-motifs
  - mass2motif-membership-annotation
  member_tools:
  - MS2LDA.Preprocessing.generate_corpus
  - MS2LDA.Preprocessing.load_and_clean
  - RMassBank
  - XCMS
  - MS2LDA
  - MS2LDAvis
  - MS2Analyzer
  - Cytoscape
  - GNPS
  - MolNetEnhancer
  - Network Annotation Propagation
  - RMolNetEnhancer
  - molconvert
  - networkx
  - pyMolNetEnhancer
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

# MS2LDA Substructure Discovery (MS2 corpus -> Mass2Motifs -> network-mapped substructures)

## Summary

MS2 spectra in, substructure-annotated features out: bag-of-fragments corpus construction, MS2LDA topic modeling into Mass2Motifs, motif annotation against MotifDB/Spec2Vec, and propagation of motif labels onto a GNPS molecular network.


## When to use

Use when you want to discover shared substructures (Mass2Motifs) across a corpus of MS2 spectra via LDA topic modeling — convert MS/MS spectra into a bag-of-fragments document corpus, run MS2LDA (Latent Dirichlet Allocation) to infer recurring fragmentation motifs, annotate motifs against MotifDB / Spec2Vec embeddings, and map motifs back onto molecular-network features for substructure-level annotation of an untargeted LC-MS/MS dataset.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — preprocess

**Goal:** raw MS2 spectra (mgf/mzML/msp) -> cleaned, intensity-normalized bag-of-fragments corpus

**EDAM operation:** http://edamontology.org/operation_3632

**Inputs:** mgf, mzML · **Outputs:** corpus-json, mgf/gnps-fbmn

**Candidate leaf skills:** `msms-noise-filtering` (primary), `mgf-spectra-noise-filtering-and-preprocessing`, `mass-spectrometry-spectrum-preprocessing--99e25f8cc3a2c48f`, `mass-spectra-preprocessing-and-binning`

**Tools (primary):** MS2LDA.Preprocessing.generate_corpus, MS2LDA.Preprocessing.load_and_clean, RMassBank, XCMS

**Other candidate tools:** RDKit, TransExION, h5py, pyteomics, pyOpenMS, pymzML, seaborn, spectrum_utils, MS2DeepScore, matchms

**Grounding:** 4 KB(s); DOIs: 10.1021/acs.analchem.9b04884, 10.1073/pnas.1608041113, 10.1186/s13321-021-00558-4, 10.1186/s13321-024-00858-5

### Stage 2 — lda_modeling

**Goal:** bag-of-fragments corpus -> Mass2Motifs (LDA topic model over spectral documents)

**EDAM operation:** http://edamontology.org/operation_3644

**Inputs:** corpus-json · **Outputs:** motifset-json

**Candidate leaf skills:** `latent-dirichlet-allocation-topic-modeling` (primary), `lda-topic-model-visualization`, `mass-spectrometry-fragmentation-motif-visualization`, `msms-fragment-bag-of-words-corpus-construction`

**Tools (primary):** MS2LDA, MS2LDAvis

**Other candidate tools:** LDAvis, MS2LDAViz, MS2LDA (latent Dirichlet allocation)

**Grounding:** 1 KB(s); DOIs: 10.1073/pnas.1608041113

### Stage 3 — motif_annotation

**Goal:** Mass2Motifs -> putative substructure annotations (MotifDB + Spec2Vec embedding match)

**EDAM operation:** http://edamontology.org/operation_3629

**Inputs:** motifset-json · **Outputs:** annotated-motifset-json

**Candidate leaf skills:** `mass2motif-spectral-matching-annotation` (primary), `motif-annotation-agreement-comparison`, `combinatorial-fragmentation-substructure-annotation`

**Tools (primary):** MS2Analyzer, MS2LDA

**Other candidate tools:** MAGMa, ModiFinder

**Grounding:** 2 KB(s); DOIs: 10.1021/jasms.4c00061, 10.1073/pnas.1608041113

### Stage 4 — network_mapping

**Goal:** annotated Mass2Motifs + GNPS molecular network -> motif-enriched network nodes

**EDAM operation:** http://edamontology.org/operation_3778

**Inputs:** annotated-motifset-json, mgf/gnps-fbmn · **Outputs:** graphml

**Candidate leaf skills:** `molecular-network-mass2motif-mapping` (primary), `molecular-network-motif-annotation`, `molecular-network-annotation-propagation--3c622bace7b9313a`, `mass-spectral-molecular-network-motif-mapping`

**Tools (primary):** Cytoscape, GNPS, MS2LDA, MolNetEnhancer, Network Annotation Propagation, RMolNetEnhancer, molconvert, networkx, pyMolNetEnhancer

**Other candidate tools:** DEREPLICATOR

**Grounding:** 2 KB(s); DOIs: 10.1038/s41587-020-0700-3, 10.3390/metabo9070144

### Stage 5 — report

**Goal:** consolidate motifs + annotations + motif-enriched network into a substructure discovery report

**EDAM operation:** http://edamontology.org/operation_3434

**Inputs:** annotated-motifset-json, graphml · **Outputs:** tsv

**Candidate leaf skills:** `ms2lda-motif-summary-parsing` (primary), `fragment-pseudo-spectrum-representation-of-motifs`, `mass2motif-membership-annotation`

**Tools (primary):** Cytoscape, GNPS, MS2LDA, MolNetEnhancer, RMolNetEnhancer, pyMolNetEnhancer

**Other candidate tools:** MS2Analyzer, MS2LDAvis, Molecular Networking

**Grounding:** 2 KB(s); DOIs: 10.1073/pnas.1608041113, 10.3390/metabo9070144

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
