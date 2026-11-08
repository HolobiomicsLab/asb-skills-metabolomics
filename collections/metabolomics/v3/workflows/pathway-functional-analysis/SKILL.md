---
name: pathway-functional-analysis-workflow
description: 'Use when you have an LC-MS metabolomics feature list (m/z, optionally
  p-values/fold changes) and want biological interpretation without prior identification
  — feature preparation, mummichog functional analysis from m/z, pathway/enrichment
  analysis, and pathway-level interpretation.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 4
  member_skills:
  - lcms-feature-table-filtering--14a2cde736d2dabd
  - metabolite-log2fc-ranking
  - metabolomics-feature-table-generation--f513762b0eaf782d
  - lc-msms-feature-quantification-table-transposition
  - mummichog-network-activity-prediction
  - mummichog-pathway-enrichment-analysis
  - mummichog-pathway-activity-prediction
  - pathway-enrichment-analysis-fishers-exact-test
  - pathway-enrichment-analysis
  - metabolite-pathway-enrichment-analysis--b4a4fd3929413241
  - pathway-over-representation-analysis--2a80d81751b143d3
  - metabolic-pathway-enrichment-visualization
  - kegg-pathway-enrichment-interpretation
  - metabolomics-pathway-enrichment-execution
  - pathway-impact-plot-visualization
  member_tools:
  - Azimuth DB
  - JMS
  - mummichog
  - cloud-mummichog
  - metDataModel
  - ConsensusPathDB
  - Human Metabolome Database (HMDB)
  - KEGG pathways database
  - dbnorm
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

# Pathway & Functional Analysis (m/z to Biology)

## Summary

End-to-end functional analysis: turn a ranked m/z feature list into predicted pathway activity and enriched metabolite sets, even without confident structure annotations.


## When to use

Use when you have an LC-MS metabolomics feature list (m/z, optionally p-values/fold changes) and want biological interpretation without prior identification — feature preparation, mummichog functional analysis from m/z, pathway/enrichment analysis, and pathway-level interpretation.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — feature_prep

**Goal:** prepare a ranked m/z feature list for functional analysis

**EDAM operation:** http://edamontology.org/operation_3435

**Inputs:** feature-table · **Outputs:** tsv

**Candidate leaf skills:** `lcms-feature-table-filtering--14a2cde736d2dabd` (primary), `metabolite-log2fc-ranking`, `metabolomics-feature-table-generation--f513762b0eaf782d`, `lc-msms-feature-quantification-table-transposition`

**Tools (primary):** Azimuth DB, JMS, mummichog

**Other candidate tools:** EnrichMet, KEGG, R, fgsea, MetaboAnalyst 6.0, MetaboAnalystR, MetaboAnalystR 4.0, OptiLCMS, asari, Anaconda, IRkernel, Jupyter Notebook, MZmine2, Optimus

**Grounding:** 4 KB(s); DOIs: 10.1021/acs.jnatprod.7b00737, 10.1093/nar/gkae253, 10.1101/2025.08.28.672951, 10.1371/journal.pcbi.1003123

### Stage 2 — mummichog

**Goal:** functional analysis directly from m/z (mummichog)

**EDAM operation:** http://edamontology.org/operation_3928

**Inputs:** tsv · **Outputs:** tsv

**Candidate leaf skills:** `mummichog-network-activity-prediction` (primary), `mummichog-pathway-enrichment-analysis`, `mummichog-pathway-activity-prediction`

**Tools (primary):** Azimuth DB, cloud-mummichog, metDataModel, mummichog

**Other candidate tools:** MetaboAnalystR 4.0, OptiLCMS, R, MetaboAnalyst 6.0, MetaboAnalystR

**Grounding:** 3 KB(s); DOIs: 10.1038/s41467-024-48009-6, 10.1093/nar/gkae253, 10.1371/journal.pcbi.1003123

### Stage 3 — pathway_enrichment

**Goal:** pathway + metabolite-set enrichment

**EDAM operation:** http://edamontology.org/operation_3928

**Inputs:** tsv, tsv · **Outputs:** tsv

**Candidate leaf skills:** `pathway-enrichment-analysis-fishers-exact-test` (primary), `pathway-enrichment-analysis`, `metabolite-pathway-enrichment-analysis--b4a4fd3929413241`, `pathway-over-representation-analysis--2a80d81751b143d3`

**Tools (primary):** cloud-mummichog, mummichog

**Other candidate tools:** Azimuth DB, metDataModel, Hmisc, IMPaLA, IP4M, MPEA, MetPA, MetaboAnalystR, R, R-scripts-for-iMAP, iMAP, lilikoi, ropls, metGWAS 1.0

**Grounding:** 3 KB(s); DOIs: 10.1093/bioinformatics/btad523, 10.1371/journal.pcbi.1003123, 10.3389/fchem.2021.659656

### Stage 4 — interpretation

**Goal:** interpret + visualize enriched pathways

**EDAM operation:** http://edamontology.org/operation_3659

**Inputs:** tsv, tsv · **Outputs:** tsv, html

**Candidate leaf skills:** `metabolic-pathway-enrichment-visualization` (primary), `kegg-pathway-enrichment-interpretation`, `metabolomics-pathway-enrichment-execution`, `pathway-impact-plot-visualization`

**Tools (primary):** ConsensusPathDB, Human Metabolome Database (HMDB), KEGG pathways database, dbnorm

**Other candidate tools:** EnrichMet, KEGG, KEGGREST, MetENP, R, STITCH, fgsea, FELLA, LION, MetaboAnalystR, Reactome

**Grounding:** 2 KB(s); DOIs: 10.1038/s41598-021-84824-3, 10.1101/2025.08.28.672951

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
