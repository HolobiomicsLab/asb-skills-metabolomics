---
name: metabolomics-workflow-router
description: Use when a user has a whole metabolomics analysis goal rather than a single atomic step and needs an end-to-end composite workflow selected and grounded against source papers.
license: CC-BY-4.0
metadata:
  kind: workflow-router
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/metabolomics/v3
  indexes:
  - workflows_index.json
  bound_by: perspicacite-semantic
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# ASB Metabolomics composite workflow router

This is the goal-level entry point of ASB Metabolomics. The leaf router (`metabolomics-router`)
finds one atomic skill; this router selects an end-to-end workflow with ordered
stages, candidate leaf skills, grounding and a declarative `workflow.yaml`.

## 1. Select

Run the commands as written, from any working directory; do not `cd`.
If a command still shows a `${...}` variable instead of an absolute path, the
host did not fill it in: replace the variable with the absolute directory of
this file, `<collection root>/workflows/_workflow_router`.

```bash
python3 "${CLAUDE_SKILL_DIR}/../../bin/search_skills.py" --query "<the user's goal>" --target workflows -k 3
```

Python 3.9 or later, standard library only, no network; where `python3` is missing, use `python` or `py -3`.
For a manual lookup, match the goal against the `description` of each row of
`${CLAUDE_SKILL_DIR}/../workflows_index.json`.

Available workflows (published as outlines):

| workflow | what it does |
|---|---|
| `compound-class-annotation` | MS2 in, a chemical-class-annotated table out: SIRIUS molecular formula, molecular fingerprint, and CANOPUS / NPClassifier compound-class prediction per feature. |
| `fbmn-annotation-propagation` | MS2 in, a network-propagated annotation table out: molecular networking, seed annotation, chemical-class assignment, and topology-driven propagation of annotations and classes across molecular families. |
| `gcms-deconvolution-and-identification` | End-to-end GC-MS annotation: deconvolve co-eluting EI spectra, match to GC-MS libraries with retention-index support, and compare groups. |
| `gnn-spectral-property-prediction` | SMILES + labelled property dataset in, a property-rescored candidate table out: RDKit/PyG molecular graph construction, GNN architecture design and supervised training, trained-model inference on candidate structures, and predicted-vs-observed candidate re-ranking. |
| `in-silico-biotransformation-prediction` | Parent SMILES + mzML in, a ranked biotransformation-product annotation table out: rule-based metabolite prediction (BioTransformer / EnviPath / Pickaxe), mass-based candidate filtering, MS/MS-based candidate screening and ranking. |
| `ion-mobility-4d-annotation` | End-to-end 4D ion-mobility annotation: extract CCS-resolved features, calibrate CCS, and annotate with collision-cross-section-aware matching. |
| `lipidomics-lcms-annotation` | End-to-end lipidomics annotation: raw LC-MS/MS in, a confidence-graded lipid table out, with lipid-class-aware identification, normalization, and group-wise statistics. |
| `masst-repository-scale-search` | End-to-end reverse metabolomics: take one molecule and find its public-dataset footprint with MASST, then add ecological context and co-occurrence interpretation. |
| `ms-imaging-spatial-metabolomics` | End-to-end MSI spatial metabolomics: from imzML to FDR-controlled ion-image annotations and anatomically-segmented, region-compared metabolite maps. |
| `ms2lda-substructure-discovery` | MS2 spectra in, substructure-annotated features out: bag-of-fragments corpus construction, MS2LDA topic modeling into Mass2Motifs, motif annotation against MotifDB/Spec2Vec, and propagation of motif labels onto a GNPS molecular network. |
| `nmr-metabolomics-profiling` | End-to-end NMR metabolomics: from raw spectra to identified, quantified metabolites and group-wise statistical comparison. |
| `paired-omics-bgc-metabolite-linking` | Assembled genomes + paired LC-MS/MS in, a ranked BGC-metabolite link table out: antiSMASH BGC detection, iPRESTO/Pfam domain tokenization, BiG-SCAPE gene cluster family clustering, GNPS molecular networking, and NPLinker-style genomic-metabolomic co-occurrence scoring. |
| `pathway-functional-analysis` | End-to-end functional analysis: turn a ranked m/z feature list into predicted pathway activity and enriched metabolite sets, even without confident structure annotations. |
| `sirius-denovo-structure-elucidation` | End-to-end de novo elucidation with SIRIUS 6: formula, structure and class prediction for novel/unannotated chemistry, with confidence-based filtering. No spectral library. |
| `spec2vec-ml-embedding-annotation` | MS2 in, an embedding-based annotation table out: peak/loss document conversion, Spec2Vec (Word2Vec) model training or loading, query-spectrum embedding, and nearest-neighbour library retrieval and annotation. |
| `stable-isotope-tracing-fluxomics` | Labelled LC-MS in, a labelling table out: isotopologue extraction, natural-abundance correction, and mass-isotopomer-distribution / enrichment analysis. |
| `statistics-and-biomarker-discovery` | End-to-end metabolomics statistics: from a raw feature table to normalized data, multivariate structure, differential features, pathway context, and ranked biomarkers. |
| `suspect-screening-exposomics` | HRMS in, a confidence-levelled suspect-hit table out: suspect-list matching, in-silico fragmentation, and identification-confidence assignment for exposomics. |
| `targeted-lcms-quantification` | Targeted transitions in, a QC'd quantification table out: peak integration, calibration-curve fitting, internal-standard normalization, and batch QC. |
| `untargeted-lcmsms-annotation` | End-to-end untargeted LC-MS/MS annotation: raw mzML in, an evidence-grounded master feature table out, combining molecular networking, library matching and SIRIUS. |

If no workflow fits the goal, fall back to the leaf router (`metabolomics-router`) and
assemble steps from atomic skills.

## 2. Run

Read the chosen `workflows/<slug>/SKILL.md` (the `read:` path of the hit) and
follow its **Stages** in order. Each stage carries a goal, candidate leaf skills
(primary first), the tools to install or invoke, and its typed inputs and
outputs. The machine-readable `workflows/<slug>/workflow.yaml` is the DAG
(`after`, `inputs_from`). Automatic grading of that DAG (`asb solve-workflow`)
is **not part of this release**: no released ASB version loads these files, so
run the stages yourself. Honor the I/O contract: each stage consumes the prior
stage's declared outputs. Optional stages are marked.

For a stage's leaf skills, read each `${CLAUDE_SKILL_DIR}/../../leaves/<leaf-slug>/SKILL.md` for
the procedure, or use the leaf router to pick among the candidates for the
exact data.

## 3. Ground

Before trusting a parameter or default, ground the stage's leaves against the
papers they were distilled from. Each stage's `grounding.kb_slugs` and `dois` in
`workflow.yaml` name the source papers and their `asb-paper-<doi>` knowledge
bases.

> These workflows are published as **outlines**: the stage structure is
> validated, the execution is not. Bindings were chosen by semantic retrieval
> plus deterministic selection. `derived_from_workflows` is a provenance record;
> no ablation experiment consuming it is released.

## Report and contribute

`metabolomics/v3` improves from use. Offer a report yourself, once per
problem, when a procedure failed or no longer matches the tool, the search
returned nothing usable and the work was done unaided, the user names a
method paper whose DOI the search does not find, the leaves had to be
composed by hand, or a command of this collection failed. Name the route:

1. this skill is wrong or could be better;
2. a skill is missing: request it, or propose yours;
3. a paper is not referenced: propose it;
4. something technical failed: report it.

After a no, do not raise it again. Draft the report, show the user its exact
title, labels and body, and post only on an explicit yes. Never paste paths,
credentials or sample names.

```bash
python3 "${CLAUDE_SKILL_DIR}/../../bin/report_issue.py" skill --skill <slug> --kind defect --symptom "<what happened>"
```

Routes: `skill`, `missing`, `paper`, `fault` (`--help` lists the fields). It
redacts and prints the report and a prefilled address. With `gh` and the
user's yes, run `gh issue create --repo HolobiomicsLab/asb-skill-collections`; otherwise give the user
the address.

Reports go to the public registry; a maintainer carries them into the
domain repository.
