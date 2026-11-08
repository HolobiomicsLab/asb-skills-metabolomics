---
name: nmr-metabolomics-profiling-workflow
description: 'Use when you have NMR metabolomics data (1D/2D spectra or FIDs) and
  want a quantified, identified metabolite profile — spectral preprocessing (phase/baseline/referencing,
  binning), metabolite identification by chemical shift, quantification, and group
  statistics.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 4
  member_skills:
  - fid-to-spectrum-preprocessing-pipeline
  - nmr-preprocessing-pipeline-interfacing
  - metabolomics-nmr-data-processing
  - spectral-preprocessing-for-multivariate-analysis
  - spectral-library-matching--0c95a117d483839e
  - metabolite-database-interrogation
  - nmr-reference-spectrum-retrieval-from-hmdb
  - nmr-metabolite-identification
  - nmr-spectral-deconvolution
  - metabolite-peak-amplitude-quantification
  - metabolite-concentration-estimation-nmr
  - nmr-mixture-quantification
  - metabolomics-univariate-group-comparison
  - two-sample-t-test-biomarker-testing
  - multiple-testing-correction
  - benjamini-hochberg-multiple-testing-correction--167cbe6451059b92
  member_tools:
  - Protomix
  - Python
  - numpy
  - pandas
  - pyicoshift
  - BMRB (Biological Magnetic Resonance Bank)
  - Bruker Avance 800 MHz NMR spectrometer with cryoprobe
  - Chenomx
  - HMDB (Human Metabolome Database)
  - TopSpin
  - MATLAB
  - NMRPipe
  - NMRbox
  - SAND
  - POMA
  - POMAShiny
  - R
  - limma
  - tidyverse
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

# NMR Metabolomics Profiling

## Summary

End-to-end NMR metabolomics: from raw spectra to identified, quantified metabolites and group-wise statistical comparison.


## When to use

Use when you have NMR metabolomics data (1D/2D spectra or FIDs) and want a quantified, identified metabolite profile — spectral preprocessing (phase/baseline/referencing, binning), metabolite identification by chemical shift, quantification, and group statistics.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — preprocess_nmr

**Goal:** NMR spectral preprocessing (phase, baseline, referencing, binning)

**EDAM operation:** http://edamontology.org/operation_3215

**Inputs:** nmr-spectrum · **Outputs:** feature-table, nmr-spectrum

**Candidate leaf skills:** `fid-to-spectrum-preprocessing-pipeline` (primary), `nmr-preprocessing-pipeline-interfacing`, `metabolomics-nmr-data-processing`, `spectral-preprocessing-for-multivariate-analysis`

**Tools (primary):** Protomix, Python, numpy, pandas, pyicoshift

**Other candidate tools:** MetaboLabPy 0.9.66, PySide6, scikit-learn, scipy, MetaboLabPy

**Grounding:** 2 KB(s); DOIs: 10.1093/bioadv/vbae192, 10.3390/metabo15010048

### Stage 2 — identification

**Goal:** identify metabolites by chemical shift matching

**EDAM operation:** http://edamontology.org/operation_3803

**Inputs:** feature-table · **Outputs:** tsv

**Candidate leaf skills:** `spectral-library-matching--0c95a117d483839e` (primary), `metabolite-database-interrogation`, `nmr-reference-spectrum-retrieval-from-hmdb`, `nmr-metabolite-identification`

**Tools (primary):** BMRB (Biological Magnetic Resonance Bank), Bruker Avance 800 MHz NMR spectrometer with cryoprobe, Chenomx, HMDB (Human Metabolome Database), Python, TopSpin

**Other candidate tools:** Human Metabolome Database (HMDB), PyQt5 5.15.11, Python >=3.9, ROIAL-NMR, pandas, pandas 2.2.3, CluPA, HMDB, R, icoshift, speaq 2.0, speaq R package

**Grounding:** 3 KB(s); DOIs: 10.1002/nbm.70131, 10.1371/journal.pcbi.1006018, 10.3389/fnins.2024.1520982

### Stage 3 — quantification

**Goal:** quantify metabolites from NMR signals

**EDAM operation:** http://edamontology.org/operation_3799

**Inputs:** nmr-spectrum, tsv · **Outputs:** tsv

**Candidate leaf skills:** `nmr-spectral-deconvolution` (primary), `metabolite-peak-amplitude-quantification`, `metabolite-concentration-estimation-nmr`, `nmr-mixture-quantification`

**Tools (primary):** MATLAB, NMRPipe, NMRbox, SAND

**Other candidate tools:** EDTA plasma analysis (NMRQNet tutorial), NMRQNet, MANIQ, Magnetstein, MasserFront

**Grounding:** 3 KB(s); DOIs: 10.1021/acs.analchem.3c03078, 10.1021/acs.analchem.3c03594, 10.1101/2023.03.01.530642

### Stage 4 — statistics

**Goal:** differential analysis of NMR profiles (univariate; multivariate where a leaf exists)

**EDAM operation:** http://edamontology.org/operation_3659

**Inputs:** tsv · **Outputs:** tsv

**Candidate leaf skills:** `metabolomics-univariate-group-comparison` (primary), `two-sample-t-test-biomarker-testing`, `multiple-testing-correction`, `benjamini-hochberg-multiple-testing-correction--167cbe6451059b92`

**Tools (primary):** POMA, POMAShiny, R, limma, tidyverse

**Other candidate tools:** NMRQNet, rDolphin, dplyr, massdataset, massstat, purrr, tidyMass, MetaboAnalyst, speaq 2.0, speaq R package

**Grounding:** 4 KB(s); DOIs: 10.1038/s41467-022-32155-w, 10.1101/2023.03.01.530642, 10.1371/journal.pcbi.1006018, 10.1371/journal.pcbi.1009148

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
