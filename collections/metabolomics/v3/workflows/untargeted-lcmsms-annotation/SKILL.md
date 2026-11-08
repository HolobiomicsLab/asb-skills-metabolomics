---
name: untargeted-lcmsms-annotation-workflow
description: 'Use when you have untargeted LC-MS/MS data (mzML) and want an annotated
  feature table — preprocessing, blank/QC filtering, feature-based molecular networking,
  spectral library matching, SIRIUS de novo annotation, optional taxonomy-aware re-weighting,
  and a fused master table. This is the canonical metabopipe-style annotation pipeline.

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
  - lcms-feature-quality-filtering
  - blank-subtraction-quality-filtering
  - blank-vs-qc-intensity-ratio-filtering
  - blank-sample-removal-qc
  - blank-filtering-of-ms-features
  - feature-based-molecular-networking--a1d4da5bf30d5f86
  - msms-spectral-similarity-molecular-networking
  - feature-based-molecular-networking--e34d752ede072f7b
  - gnps-molecular-networking-workflow
  - spectral-library-matching--c9ea083859d5db3b
  - mass-spectral-library-matching--d96cdbebfa644977
  - spectral-library-matching--c3bacc8572caec82
  - spectral-library-matching--599e7745b946f0de
  - batch-ms2-annotation-automation
  - msms-spectra-sirius-csi-fingerid-annotation
  - csi-fingerid-structure-annotation
  - insilico-formula-annotation-sirius
  - multi-tool-annotation-table-construction
  - metabolite-annotation-mapping
  - metabolomics-feature-table-processing
  - mass-spectral-library-matching-cleanup
  member_tools:
  - MS2LDA.Preprocessing.generate_corpus
  - MS2LDA.Preprocessing.load_and_clean
  - RMassBank
  - XCMS
  - Pandas
  - TidyMS
  - pyopenms
  - scikit-learn
  - ChemWalker
  - GNPS2
  - MSConvert
  - MZmine
  - MZmine 2.53
  - SIRIUS 5.8.5
  - GNPS
  - tissueMASST
  - CANOPUS
  - ENPKG
  - ENPKG (04_enpkg_sirius_canopus)
  - NPClassifier
  - SIRIUS
  - ISDB-LOTUS
  - MS2DECIDE
  - matchms
  coverage_gaps: []
  derived_from_workflows:
  - coll_ms2deepscore
  - coll_ramclust_cq
  - coll_xcms_cq
  - coll_inventa_cq
  - coll_nmr2struct
  - spec2vec_grounded
  - spec2vec_pkg_oalarge
  - coll_bioactivity_based_molecular_networking_cq
  - coll_concise_cq
  - coll_redu_cq
  - coll_deepmsprofiler_cq
  - coll_cardinal_cq
  - coll_dures_cq
  - coll_fbmn_stats_cq
  - coll_idsl_ipa_cq
  - coll_metabodirect
  - coll_multiomicsintegrator_cq
  - coll_peakqc_cq
  - coll_tardis
  - coll_vimms
  - coll_lipidin_cq
  - coll_molnetenhancer
  - coll_ms2rescore_immunopeptidome_rescoring_cq
  - coll_npclassscore_cq
  - coll_rapidmass_cq
  - coll_tardis_cq
  - coll_esp_cq
  - coll_graphormer_rt_cq
  - coll_lipidmatch_cq
  - coll_corems
  bound_by: perspicacite-semantic
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/metabolomics/v3
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# Untargeted LC-MS/MS Metabolite Annotation (FBMN + SIRIUS)

## Summary

End-to-end untargeted LC-MS/MS annotation: raw mzML in, an evidence-grounded master feature table out, combining molecular networking, library matching and SIRIUS.


## When to use

Use when you have untargeted LC-MS/MS data (mzML) and want an annotated feature table — preprocessing, blank/QC filtering, feature-based molecular networking, spectral library matching, SIRIUS de novo annotation, optional taxonomy-aware re-weighting, and a fused master table. This is the canonical metabopipe-style annotation pipeline.


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

### Stage 2 — qc_filter  [OPTIONAL]

**Goal:** (optional) remove blank / background / low-quality features before annotation

**EDAM operation:** http://edamontology.org/operation_3695

**Inputs:** feature-table · **Outputs:** feature-table

**Candidate leaf skills:** `lcms-feature-quality-filtering` (primary), `blank-subtraction-quality-filtering`, `blank-vs-qc-intensity-ratio-filtering`, `blank-sample-removal-qc`, `blank-filtering-of-ms-features`

**Tools (primary):** Pandas, TidyMS, pyopenms, scikit-learn

**Other candidate tools:** MSnbase, Metaboseek, mzR, xcms, GetFeatistics, MS-DIAL, R, patRoon, Jupyter, OpenMS, OpenMS 3.0, Python, Snakemake, UmetaFlow, FluoroMatch Flow 2.0, FluoroMatch Modular, LipidMatch, LipidMatch 4.2

**Grounding:** 5 KB(s); DOIs: 10.1007/s00216-021-03392-7, 10.1038/s41467-022-28391-9, 10.1186/s13321-023-00724-w, 10.1515/jib-2025-0047 …

### Stage 3 — network

**Goal:** MS2 spectra -> molecular family graph (modified cosine; GNPS-style components)

**EDAM operation:** http://edamontology.org/operation_3214

**Inputs:** mgf/gnps-fbmn · **Outputs:** graphml, tsv

**Candidate leaf skills:** `feature-based-molecular-networking--a1d4da5bf30d5f86` (primary), `msms-spectral-similarity-molecular-networking`, `feature-based-molecular-networking--e34d752ede072f7b`, `gnps-molecular-networking-workflow`

**Tools (primary):** ChemWalker, GNPS2, MSConvert, MZmine, MZmine 2.53, SIRIUS 5.8.5

**Other candidate tools:** Cytoscape, GNPS, MSConvert (ProteoWizard), GNPS (Global Natural Products Social Molecular Networking), Mascot Generic Format (MGF), NP3 MS Workflow, Tremolo, MZmine2, OpenMS TOPPView

**Grounding:** 3 KB(s); DOIs: 10.1021/acs.analchem.3c05829, 10.1038/s41596-020-0317-5, 10.1101/2024.08.07.607073

### Stage 4 — library_match

**Goal:** MS2 spectra -> spectral library annotations (cosine match to reference libraries)

**EDAM operation:** http://edamontology.org/operation_3631

**Inputs:** mgf/gnps-fbmn · **Outputs:** tsv

**Candidate leaf skills:** `spectral-library-matching--c9ea083859d5db3b` (primary), `mass-spectral-library-matching--d96cdbebfa644977`, `spectral-library-matching--c3bacc8572caec82`, `spectral-library-matching--599e7745b946f0de`

**Tools (primary):** GNPS, tissueMASST

**Other candidate tools:** MS2LDA, MS2LDAvis, Mass Bank, MassIVE, ChemWalker, GNPS library, GNPS2, GNPS2 (FBMN workflow), MZmine, MZmine 2.53, RegFilter, MS2DECIDE, Spec2Vec, gensim (Word2Vec), matchms, spec2vec_gnps_data_analysis

**Grounding:** 4 KB(s); DOIs: 10.1073/pnas.1608041113, 10.1101/2024.08.07.607073, 10.1101/2025.04.28.651123, 10.1371/journal.pcbi.1008724

### Stage 5 — sirius

**Goal:** MS2 spectra -> formula + structure + class (SIRIUS / CSI:FingerID / CANOPUS)

**EDAM operation:** http://edamontology.org/operation_3860

**Inputs:** mgf/sirius · **Outputs:** tsv

**Candidate leaf skills:** `batch-ms2-annotation-automation` (primary), `msms-spectra-sirius-csi-fingerid-annotation`, `csi-fingerid-structure-annotation`, `insilico-formula-annotation-sirius`

**Tools (primary):** CANOPUS, ENPKG, ENPKG (04_enpkg_sirius_canopus), NPClassifier, SIRIUS

**Other candidate tools:** CSI:FingerID, HILIC-Q Exactive HF mass spectrometry, OpenMS, UmetaFlow, pyOpenMS, Fast Search API, MZmine, Python 3.10, microbeMASST

**Grounding:** 3 KB(s); DOIs: 10.1021/acs.analchem.8b04698, 10.1021/acscentsci.3c00800, 10.1038/s41564-023-01575-9

### Stage 6 — fusion

**Goal:** consolidate networking + library + SIRIUS into one master table, with optional taxonomy re-weighting

**EDAM operation:** http://edamontology.org/operation_3434

**Inputs:** feature-table, graphml, tsv, metadata · **Outputs:** tsv

**Candidate leaf skills:** `multi-tool-annotation-table-construction` (primary), `metabolite-annotation-mapping`, `metabolomics-feature-table-processing`, `mass-spectral-library-matching-cleanup`

**Tools (primary):** GNPS, ISDB-LOTUS, MS2DECIDE, Sirius, matchms

**Other candidate tools:** CSI:FingerID, ClassyFire, Empress, QIIME 2, Qemistree, q2-qemistree, MZmine2, Python, Qemistree (q2-qemistree), R, Inventa, MZmine3, gnps_postprocessing, timaR

**Grounding:** 3 KB(s); DOIs: 10.1002/cmtd.202400088, 10.1038/s41589-020-00677-3, 10.3389/fmolb.2022.1028334

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
