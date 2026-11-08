---
name: suspect-screening-exposomics-workflow
description: 'Use when you have untargeted HRMS data and want to screen for a defined
  suspect list of environmental / exposure-relevant compounds — detect features, match
  them to suspect-list entries by exact mass/RT/MS2, elucidate structures of hits
  by in-silico fragmentation, and assign identification confidence levels (Schymanski),
  producing a confidence-annotated suspect-hit table.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 5
  member_skills:
  - lcms-feature-detection-and-quantification
  - lc-msms-feature-detection
  - lc-ms-feature-detection--e300bd20a9eef616
  - lc-ms-feature-quantification-processing
  - accurate-mass-suspect-screening
  - retention-time-shift-and-noise-thresholding
  - msms-fragment-screening
  - mass-signal-cross-referencing-against-in-silico-predictions
  - candidate-structure-retrieval-from-pubchem-chemspider
  - spectra-candidate-ranking-evaluation
  - in-silico-fragmentation-spectrum-prediction
  - structure-candidate-ranking-comparison
  - schymanski-identification-level-assignment
  - ms-annotation-confidence-level-assignment
  - compound-annotation-confidence-estimation
  - annotation-confidence-level-mapping
  - csv-table-consolidation
  - gcms-hit-table-filtering
  - consensus-spectrum-candidate-ranking
  - compound-decontamination-filtering
  member_tools:
  - Jupyter
  - OpenMS
  - OpenMS 3.0
  - Python
  - Snakemake
  - ThermoRawFileParser
  - UmetaFlow
  - pyOpenMS
  - CompTox Dashboard
  - PFΔScreen
  - MetFrag
  - MetFrag2.2
  - MetFragCommandLine
  - MetFragLib
  - OpenBabel
  - metfRag (MetFragR)
  - xcms
  - R
  - patRoon 2.0
  - TurboPutative
  coverage_gaps: []
  derived_from_workflows:
  - coll_ms2deepscore
  - coll_metabodirect
  bound_by: perspicacite-semantic
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/metabolomics/v3
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# Suspect Screening for Exposomics (suspect lists -> confidence-levelled IDs)

## Summary

HRMS in, a confidence-levelled suspect-hit table out: suspect-list matching, in-silico fragmentation, and identification-confidence assignment for exposomics.


## When to use

Use when you have untargeted HRMS data and want to screen for a defined suspect list of environmental / exposure-relevant compounds — detect features, match them to suspect-list entries by exact mass/RT/MS2, elucidate structures of hits by in-silico fragmentation, and assign identification confidence levels (Schymanski), producing a confidence-annotated suspect-hit table.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — preprocess

**Goal:** raw HRMS -> feature table + MS2 export

**EDAM operation:** http://edamontology.org/operation_3215

**Inputs:** mzML · **Outputs:** feature-table, mgf/gnps-fbmn

**Candidate leaf skills:** `lcms-feature-detection-and-quantification` (primary), `lc-msms-feature-detection`, `lc-ms-feature-detection--e300bd20a9eef616`, `lc-ms-feature-quantification-processing`

**Tools (primary):** Jupyter, OpenMS, OpenMS 3.0, Python, Snakemake, ThermoRawFileParser, UmetaFlow, pyOpenMS

**Other candidate tools:** COSMIC, CSI:FingerID, SIRIUS 4, MZmine, ProteoWizard, matchms, GNPS, KNIME, MS-FINDER, Optimus, Sirius

**Grounding:** 4 KB(s); DOIs: 10.1021/acscentsci.3c00800, 10.1038/nprot.2017.122, 10.1038/s41587-021-01045-9, 10.1186/s13321-023-00724-w

### Stage 2 — suspect_match

**Goal:** features -> suspect-list matches by exact mass / RT / MS2

**EDAM operation:** http://edamontology.org/operation_3631

**Inputs:** feature-table, mgf/gnps-fbmn · **Outputs:** tsv

**Candidate leaf skills:** `accurate-mass-suspect-screening` (primary), `retention-time-shift-and-noise-thresholding`, `msms-fragment-screening`, `mass-signal-cross-referencing-against-in-silico-predictions`

**Tools (primary):** CompTox Dashboard, PFΔScreen, Python

**Other candidate tools:** MSnbase, MetFrag, R, Shinyscreen, mzR, FluoroMatch 2.0, FluoroMatch Flow 2.0, FluoroMatch Modular, LipidMatch, BioTransformer 3.0, MZmine 2.53, R 4.0.3

**Grounding:** 4 KB(s); DOIs: 10.1007/s00216-021-03392-7, 10.1007/s00216-023-05070-2, 10.1016/j.dmd.2025.100183, 10.1186/s13321-025-01044-x

### Stage 3 — in_silico_fragment

**Goal:** suspect hits -> in-silico fragmentation structure ranking (MetFrag / SIRIUS)

**EDAM operation:** http://edamontology.org/operation_3860

**Inputs:** tsv, mgf/gnps-fbmn · **Outputs:** tsv

**Candidate leaf skills:** `candidate-structure-retrieval-from-pubchem-chemspider` (primary), `spectra-candidate-ranking-evaluation`, `in-silico-fragmentation-spectrum-prediction`, `structure-candidate-ranking-comparison`

**Tools (primary):** MetFrag, MetFrag2.2, MetFragCommandLine, MetFragLib, OpenBabel, metfRag (MetFragR), xcms

**Other candidate tools:** SIRIUS4, TeFT, CFM-ID, CSI:FingerID, ChemDistiller, SIRIUS, HILIC-Q Exactive HF mass spectrometry

**Grounding:** 4 KB(s); DOIs: 10.1021/acs.analchem.8b04698, 10.1038/s42004-024-01189-0, 10.1093/bioinformatics/bty080, 10.1186/s13321-016-0115-9

### Stage 4 — confidence

**Goal:** assign identification confidence levels (Schymanski 1-5) to hits

**EDAM operation:** http://edamontology.org/operation_0224

**Inputs:** tsv · **Outputs:** tsv

**Candidate leaf skills:** `schymanski-identification-level-assignment` (primary), `ms-annotation-confidence-level-assignment`, `compound-annotation-confidence-estimation`, `annotation-confidence-level-mapping`

**Tools (primary):** R, patRoon 2.0

**Other candidate tools:** FingerID, Galaxy-M, MetabolomicsTools catalogue, patRoonData, MetabolomicsTools Wiki

**Grounding:** 2 KB(s); DOIs: 10.1007/s11306-017-1242-7, 10.21105/joss.04029

### Stage 5 — report

**Goal:** consolidate suspect hits + structures + confidence into an annotated table

**EDAM operation:** http://edamontology.org/operation_3434

**Inputs:** tsv, feature-table · **Outputs:** tsv

**Candidate leaf skills:** `csv-table-consolidation` (primary), `gcms-hit-table-filtering`, `consensus-spectrum-candidate-ranking`, `compound-decontamination-filtering`

**Tools (primary):** TurboPutative

**Other candidate tools:** Agilent Unknowns Analysis, PubChem, R, uafR, MVP, MZmine3, MassSpecGym, LOTUS, devtools

**Grounding:** 3 KB(s); DOIs: 10.1101/2025.11.12.688047, 10.1371/journal.pone.0306202, 10.3389/fmolb.2022.952149

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
