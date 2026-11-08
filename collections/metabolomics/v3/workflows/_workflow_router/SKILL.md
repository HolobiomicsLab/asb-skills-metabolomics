---
name: metabolomics-workflow-router
description: Use when a user has a whole metabolomics analysis goal rather than a single atomic step and needs an end-to-end composite workflow selected and grounded against source papers.
license: CC-BY-4.0
metadata:
  kind: workflow-router
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/metabolomics/v3
  indexes: [workflows_index.json]
  bound_by: perspicacite-semantic
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# ASB Metabolomics — composite workflow router

This is the **goal-level** entry point for the ASB Metabolomics collection. The leaf router (`metabolomics-router`) finds one atomic skill; this router selects an **end-to-end composite workflow** with ordered stages, vetted leaf skills, grounding, and a declarative `workflow.yaml`.

Use it in three steps: **select → run → ground**.

## 1. Select

Run the collection's retrieval command:

```bash
python bin/search_skills.py --query "<the user's goal>" --collection . --target workflows --k 3
```

For manual lookup, load `workflows_index.json` and match the user's goal against each workflow's `description`.

Available workflows (published as outlines):

| name | intent | stage count |
|---|---|---:|
| `compound-class-annotation-workflow` | MS2 in, a chemical-class-annotated table out: SIRIUS molecular formula, molecular fingerprint, and CANOPUS / NPClassifier compound-class prediction per feature. | 5 |
| `feature-based-molecular-networking-and-propagation-workflow` | MS2 in, a network-propagated annotation table out: molecular networking, seed annotation, chemical-class assignment, and topology-driven propagation of annotations and classes across molecular families. | 6 |
| `gcms-deconvolution-identification-workflow` | End-to-end GC-MS annotation: deconvolve co-eluting EI spectra, match to GC-MS libraries with retention-index support, and compare groups. | 5 |
| `gnn-spectral-property-prediction-workflow` | SMILES + labelled property dataset in, a property-rescored candidate table out: RDKit/PyG molecular graph construction, GNN architecture design and supervised training, trained-model inference on candidate structures, and predicted-vs-observed candidate re-ranking. | 5 |
| `in-silico-biotransformation-prediction-workflow` | Parent SMILES + mzML in, a ranked biotransformation-product annotation table out: rule-based metabolite prediction (BioTransformer / EnviPath / Pickaxe), mass-based candidate filtering, MS/MS-based candidate screening and ranking. | 5 |
| `ion-mobility-4d-annotation-workflow` | End-to-end 4D ion-mobility annotation: extract CCS-resolved features, calibrate CCS, and annotate with collision-cross-section-aware matching. | 4 |
| `lipidomics-lcms-annotation-workflow` | End-to-end lipidomics annotation: raw LC-MS/MS in, a confidence-graded lipid table out, with lipid-class-aware identification, normalization, and group-wise statistics. | 6 |
| `masst-repository-scale-search-workflow` | End-to-end reverse metabolomics: take one molecule and find its public-dataset footprint with MASST, then add ecological context and co-occurrence interpretation. | 4 |
| `ms-imaging-spatial-metabolomics-workflow` | End-to-end MSI spatial metabolomics: from imzML to FDR-controlled ion-image annotations and anatomically-segmented, region-compared metabolite maps. | 4 |
| `ms2lda-substructure-discovery-workflow` | MS2 spectra in, substructure-annotated features out: bag-of-fragments corpus construction, MS2LDA topic modeling into Mass2Motifs, motif annotation against MotifDB/Spec2Vec, and propagation of motif labels onto a GNPS molecular network. | 5 |
| `nmr-metabolomics-profiling-workflow` | End-to-end NMR metabolomics: from raw spectra to identified, quantified metabolites and group-wise statistical comparison. | 4 |
| `paired-omics-bgc-metabolite-linking-workflow` | Assembled genomes + paired LC-MS/MS in, a ranked BGC-metabolite link table out: antiSMASH BGC detection, iPRESTO/Pfam domain tokenization, BiG-SCAPE gene cluster family clustering, GNPS molecular networking, and NPLinker-style genomic-metabolomic co-occurrence scoring. | 6 |
| `pathway-functional-analysis-workflow` | End-to-end functional analysis: turn a ranked m/z feature list into predicted pathway activity and enriched metabolite sets, even without confident structure annotations. | 4 |
| `sirius-denovo-structure-elucidation-workflow` | End-to-end de novo elucidation with SIRIUS 6: formula, structure and class prediction for novel/unannotated chemistry, with confidence-based filtering. No spectral library. | 5 |
| `spec2vec-ml-embedding-annotation-workflow` | MS2 in, an embedding-based annotation table out: peak/loss document conversion, Spec2Vec (Word2Vec) model training or loading, query-spectrum embedding, and nearest-neighbour library retrieval and annotation. | 4 |
| `stable-isotope-tracing-fluxomics-workflow` | Labelled LC-MS in, a labelling table out: isotopologue extraction, natural-abundance correction, and mass-isotopomer-distribution / enrichment analysis. | 5 |
| `statistics-biomarker-discovery-workflow` | End-to-end metabolomics statistics: from a raw feature table to normalized data, multivariate structure, differential features, pathway context, and ranked biomarkers. | 5 |
| `suspect-screening-exposomics-workflow` | HRMS in, a confidence-levelled suspect-hit table out: suspect-list matching, in-silico fragmentation, and identification-confidence assignment for exposomics. | 5 |
| `targeted-lcms-quantification-workflow` | Targeted transitions in, a QC'd quantification table out: peak integration, calibration-curve fitting, internal-standard normalization, and batch QC. | 5 |
| `untargeted-lcmsms-annotation-workflow` | End-to-end untargeted LC-MS/MS annotation: raw mzML in, an evidence-grounded master feature table out, combining molecular networking, library matching and SIRIUS. | 6 |

For a single-step request, use the leaf router (`metabolomics-router`) and select an atomic skill.

## 2. Run

Read the chosen `workflows/<slug>/SKILL.md` and follow its **Stages** in order. Each stage carries a goal, candidate leaf skills (primary first), the tools to install or invoke, and its typed inputs and outputs. The machine-readable `workflows/<slug>/workflow.yaml` is the DAG (`after`, `inputs_from`). Automatic grading of that DAG (`asb solve-workflow`) is **not part of this release**: no released ASB version loads these files, so run the stages yourself. Honor the I/O contract: each stage consumes the prior stage's declared outputs. Optional stages are marked.

For a stage's leaf skills, read each `leaves/<leaf-slug>/SKILL.md` for the procedure, or use the leaf router to pick among the candidates for the exact data.

## 3. Ground

Before trusting a parameter or default, ground the stage's leaves against the papers they were distilled from. Each stage's `grounding` record in `workflow.yaml` names the source-paper knowledge bases.

> These workflows are published as **outlines**: the stage structure is validated, the execution is not. Bindings were chosen by semantic retrieval plus deterministic selection. `derived_from_workflows` is a provenance record; no experiment consuming it is released.
