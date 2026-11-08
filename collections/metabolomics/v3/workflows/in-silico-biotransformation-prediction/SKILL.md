---
name: in-silico-biotransformation-prediction-workflow
description: 'Use when you have a parent structure (drug, natural product, xenobiotic)
  and untargeted LC-MS/MS data and want to find its biotransformation products — predict
  plausible metabolites in-silico by rule-based expansion (BioTransformer mammalian/gut-microbial/
  environmental rules, EnviPath microbial degradation, or MINE/Pickaxe combinatorial
  reaction-network generation), filter candidates to those detectable in the experimental
  mass range, then screen and rank them against the observed LC-MS/MS features to
  annotate which predicted biotransformation products were actually seen.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 5
  member_skills:
  - lcms-feature-detection-and-quantification
  - lc-ms-peak-detection-and-feature-alignment
  - untargeted-metabolomics-feature-extraction
  - lc-ms-feature-detection--512a653b5701a1cd
  - cyp450-phasei-phaseii-biotransformation-rule-application
  - biotransformer-batch-metabolite-prediction
  - biotransformer-one-step-transformation-recapitulation
  - gut-microbial-biotransformation-prediction
  - metabolite-mass-filtering-by-exact-mass-tolerance
  - in-silico-biotransformation-prediction
  - candidate-mass-tolerance-filtering
  - metabolite-mass-accuracy-cross-referencing
  - candidate-molecule-fragmentation-scoring
  - in-silico-fragmentation-scoring--8f98c1a8185e1084
  - candidate-structure-retrieval-from-pubchem-chemspider
  - tandem-mass-spectra-candidate-ranking
  - biotransformation-reaction-rule-matching
  - feature-table-curation
  - compound-annotation-table-generation
  - differential-feature-cross-referencing
  member_tools:
  - Jupyter
  - OpenMS
  - OpenMS 3.0
  - Python
  - Snakemake
  - ThermoRawFileParser
  - UmetaFlow
  - pyOpenMS
  - BioTransformer
  - Java
  - RDKit
  - AddCarbon
  - DeepMet
  - PyTorch
  - MAGMa
  - PubChem
  - emetabolomics web interface
  - MetFrag
  - CMDN
  - LC-HRMS
  - biotransformation prediction
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

# In-Silico Biotransformation / Metabolite Prediction (parent structure -> matched biotransformation products)

## Summary

Parent SMILES + mzML in, a ranked biotransformation-product annotation table out: rule-based metabolite prediction (BioTransformer / EnviPath / Pickaxe), mass-based candidate filtering, MS/MS-based candidate screening and ranking.


## When to use

Use when you have a parent structure (drug, natural product, xenobiotic) and untargeted LC-MS/MS data and want to find its biotransformation products — predict plausible metabolites in-silico by rule-based expansion (BioTransformer mammalian/gut-microbial/ environmental rules, EnviPath microbial degradation, or MINE/Pickaxe combinatorial reaction-network generation), filter candidates to those detectable in the experimental mass range, then screen and rank them against the observed LC-MS/MS features to annotate which predicted biotransformation products were actually seen.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — preprocess

**Goal:** raw mzML -> feature table + MS2 export

**EDAM operation:** http://edamontology.org/operation_3215

**Inputs:** mzML · **Outputs:** feature-table, mgf/gnps-fbmn

**Candidate leaf skills:** `lcms-feature-detection-and-quantification` (primary), `lc-ms-peak-detection-and-feature-alignment`, `untargeted-metabolomics-feature-extraction`, `lc-ms-feature-detection--512a653b5701a1cd`

**Tools (primary):** Jupyter, OpenMS, OpenMS 3.0, Python, Snakemake, ThermoRawFileParser, UmetaFlow, pyOpenMS

**Other candidate tools:** MS2LDA.Preprocessing.generate_corpus, MS2LDA.Preprocessing.load_and_clean, RMassBank, XCMS, Azimuth DB, JMS, MAVEN, metDataModel, mummichog, MS-DIAL, MS-FINDER, MZmine, MetaboAnalystR 4.0, OptiLCMS, ProteoWizard, R, SIRIUS

**Grounding:** 4 KB(s); DOIs: 10.1038/s41467-024-48009-6, 10.1073/pnas.1608041113, 10.1186/s13321-023-00724-w, 10.1371/journal.pcbi.1003123

### Stage 2 — biotransformation_prediction

**Goal:** parent structure -> predicted metabolite/transformation-product structures (rule-based expansion)

**EDAM operation:** http://edamontology.org/operation_3802

**Inputs:** smiles · **Outputs:** tsv

**Candidate leaf skills:** `cyp450-phasei-phaseii-biotransformation-rule-application` (primary), `biotransformer-batch-metabolite-prediction`, `biotransformer-one-step-transformation-recapitulation`, `gut-microbial-biotransformation-prediction`

**Tools (primary):** BioTransformer, Java, RDKit

**Other candidate tools:** DeepMet, DeepMet (CLM), PyTorch, Python, Snakemake

**Grounding:** 2 KB(s); DOIs: 10.1101/2024.11.13.623458, 10.1289/ehp7722

### Stage 3 — candidate_filtering

**Goal:** predicted candidates -> mass-plausible candidates (filter against the experimental peak list)

**EDAM operation:** http://edamontology.org/operation_3801

**Inputs:** tsv, feature-table · **Outputs:** tsv

**Candidate leaf skills:** `metabolite-mass-filtering-by-exact-mass-tolerance` (primary), `in-silico-biotransformation-prediction`, `candidate-mass-tolerance-filtering`, `metabolite-mass-accuracy-cross-referencing`

**Tools (primary):** AddCarbon, DeepMet, PyTorch, Python, RDKit, Snakemake

**Other candidate tools:** MSnbase, MetApp, MetIDfyR, R, Rdisop, 001-007.ipynb, Jaccard similarity algorithm, Python 3.8.13, BioTransformer 3.0, MZmine 2.53, R 4.0.3

**Grounding:** 4 KB(s); DOIs: 10.1016/j.dmd.2025.100183, 10.1021/acs.analchem.0c02281, 10.1021/acs.analchem.5c01414, 10.1101/2024.11.13.623458

### Stage 4 — ms_matching

**Goal:** mass-plausible candidates -> MS/MS-matched and ranked biotransformation products

**EDAM operation:** http://edamontology.org/operation_3802

**Inputs:** tsv, mgf/gnps-fbmn · **Outputs:** tsv

**Candidate leaf skills:** `candidate-molecule-fragmentation-scoring` (primary), `in-silico-fragmentation-scoring--8f98c1a8185e1084`, `candidate-structure-retrieval-from-pubchem-chemspider`, `tandem-mass-spectra-candidate-ranking`

**Tools (primary):** MAGMa, PubChem, emetabolomics web interface, BioTransformer, MetFrag

**Other candidate tools:** MetFrag2.2, MetFragCommandLine, MetFragLib, OpenBabel, metfRag (MetFragR), xcms, CFM-ID

**Grounding:** 2 KB(s); DOIs: 10.1186/s13321-016-0115-9, 10.5702/massspectrometry.s0033

### Stage 5 — report

**Goal:** consolidate parent structure, predicted candidates, and MS/MS-matched hits into a biotransformation-product annotation table

**EDAM operation:** http://edamontology.org/operation_3434

**Inputs:** tsv, feature-table · **Outputs:** tsv

**Candidate leaf skills:** `biotransformation-reaction-rule-matching` (primary), `feature-table-curation`, `compound-annotation-table-generation`, `differential-feature-cross-referencing`

**Tools (primary):** CMDN, LC-HRMS, biotransformation prediction

**Other candidate tools:** BioTransformer 3.0, MZmine 2.53, R 4.0.3, MetaboAnalyst 6.0, MetaboAnalystR, MetaboAnalystR 4.0, R, MSDial

**Grounding:** 3 KB(s); DOIs: 10.1016/j.dmd.2025.100183, 10.1021/acs.est.5c08558, 10.1093/nar/gkae253

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
