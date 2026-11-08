---
name: cancer-omics-matrix-factorization
description: Use when a cancer omics matrix must be factorized into interpretable
  metagene/metasample components, when comparing ICA vs PCA vs NMF reproducibility,
  or when deconvolving immune signals from bulk transcriptomes.
when_to_use_negative:
- Input is already a reduced or factorized representation rather than an omics data
  matrix requiring source separation
- The goal is supervised prediction or classification of tumors — ICA here is unsupervised
  decomposition for signal separation, not a class predictor
- A single, non-stabilized ICA run is acceptable only for exploratory inspection;
  the review's reproducibility claims rest on stabilization, so single-run output
  should not be treated as the robust result
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3474
tools:
- name: Jupyter notebook
  role: Interactive environment executing the worked ICA example on the cancer omics
    dataset accompanying the IJMS review
  repo: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- name: stabilized-ica
  role: Python package providing the stabilized ICA computation used by BIODICA for
    reproducible component extraction
  repo: https://github.com/ncaptier/stabilized-ica
- name: BIODICA Navigator
  role: GUI/command-line environment for computing independent components with stability
    analysis and interpreting results (e.g. toppgene via -dotoppgene)
  repo: https://github.com/LabBandSB/BIODICA
- name: DeconICA
  role: R package for deconvolving immune-related signals in transcriptomes via unsupervised
    source separation with icasso stabilization
  repo: https://github.com/UrszulaCzerwinska/DeconICA
- name: Python 3.7.3
  role: Pinned runtime (v3.7.3:ef4ec6ed12) for the companion notebook
- name: pip
  role: Installs the notebook's dependencies from the requirements file
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.3390/ijms20184414
    title: Independent Component Analysis for Unraveling the Complexity of Cancer
      Omics Datasets
  source_content_hash: sha256:c4b14a9f75a1dba0652447e99357a9537b26f73453b75b9bd412dea40586e4ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
  tools:
  - BIODICA Navigator
  - DeconICA
  - Jupyter notebook
  - Python
  - Python 3.7.3
  - pip
  - stabilized-ica
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cancer-omics-matrix-factorization

## Summary

Decomposes cancer omics data matrices (transcriptome, methylome, proteome, single-cell) into independent components yielding metagene (source, sk) and metasample (mixing, ak) matrices, using the IJMS review's companion Jupyter notebook and stabilized ICA tooling, with PCA/NMF as comparison baselines.

## When to use

Use when a cancer omics matrix must be factorized into interpretable metagene/metasample components, when comparing ICA vs PCA vs NMF reproducibility, or when deconvolving immune signals from bulk transcriptomes.

## When NOT to use

- Input is already a reduced or factorized representation rather than an omics data matrix requiring source separation
- The goal is supervised prediction or classification of tumors — ICA here is unsupervised decomposition for signal separation, not a class predictor
- A single, non-stabilized ICA run is acceptable only for exploratory inspection; the review's reproducibility claims rest on stabilization, so single-run output should not be treated as the robust result

## Inputs

- Cancer omics data matrix (transcriptome, methylome, proteome, or single-cell) with samples as rows and features as columns
- Jupyter_notebook_package_requirements.txt dependency list
- Precomputed metagene/metasample decomposition files (e.g. -sfile path_to_metagene_file for BIODICA toppgene analysis)

## Outputs

- Metagene (source, sk) matrix and metasample (mixing, ak) matrix
- Notebook output plots and tables from the decomposition
- metagene_reproducibility_comparison.csv with one row per method (stabilized ICA, PCA, NMF) and cross-decomposition correlation-hit metrics

## How to apply

Set up the environment with Python 3.7.3 (v3.7.3:ef4ec6ed12) and install dependencies from Jupyter_notebook_package_requirements.txt via pip, then run the companion notebook from github.com/sysbio-curie/ICA-in-Cancer-research-review-materials, loading the omics matrix with samples as rows and features as columns and decomposing it into source (metagene, sk) and mixing (metasample, ak) components. For robustness, use stabilized ICA — as in the stabilized-ica Python package used by BIODICA — repeating the decomposition over multiple randomized initializations, since BIODICA computes 'the optimal number of the components from their stabilities'. To benchmark, run PCA and NMF on the same matrix with a comparable number of components, then count pairwise correlation hits between component vectors across decompositions (ICA-vs-ICA, ICA-vs-PCA, ICA-vs-NMF, PCA-vs-NMF) following the reciprocal best hit framework of Reference [38] (Section 2.4), which found stabilized ICA superior in reproducibility. For immune-focused deconvolution, the DeconICA R package offers fastICA with icasso stabilization (vignette 'Running fastICA with icasso stabilisation'). BIODICA Navigator can be run as a GUI (java -jar BIODICA_GUI.jar) or command line and reused on precomputed decompositions without recomputing ICA.

## Related tools

- **Jupyter notebook** (Interactive environment executing the worked ICA example on the cancer omics dataset accompanying the IJMS review) — https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- **stabilized-ica** (Python package providing the stabilized ICA computation used by BIODICA for reproducible component extraction) — https://github.com/ncaptier/stabilized-ica
- **BIODICA Navigator** (GUI/command-line environment for computing independent components with stability analysis and interpreting results (e.g. toppgene via -dotoppgene)) — https://github.com/LabBandSB/BIODICA
- **DeconICA** (R package for deconvolving immune-related signals in transcriptomes via unsupervised source separation with icasso stabilization) — https://github.com/UrszulaCzerwinska/DeconICA
- **Python 3.7.3** (Pinned runtime (v3.7.3:ef4ec6ed12) for the companion notebook)
- **pip** (Installs the notebook's dependencies from the requirements file)

## Examples

```
pip install Jupyter_notebook_package_requirements.txt
```

## Evaluation signals

- The notebook executes end-to-end without error under Python 3.7.3 and produces both the sk and ak matrices as output artifacts
- Output plots and tables are generated and saved from the executed decomposition cells
- Across repeated randomized stabilizing ICA runs, correlation hits between metagene vectors exceed the significance threshold used in the Reference [38] framework
- metagene_reproducibility_comparison.csv exists, contains a row for each of stabilized ICA, PCA, and NMF, and every method has a numeric cross-decomposition correlation-hit metric
- Qualitative comparison matches Section 2.4: stabilized ICA outperforms PCA and NMF on reproducibility/generalizability metrics

## Limitations

- The companion notebook is pinned to Python 3.7.3; other interpreter versions are not validated
- BIODICA requires Java 1.8 or later, the local folder name must not contain spaces or hyphens, and the MATLAB core is 'currently ... of interest only for historical and compatibility reasons'
- Unstabilized ICA components vary across initializations; stability analysis over multiple runs is needed for the optimal number of components and reproducible metagenes
- DeconICA installation requires devtools with build_vignettes and dependencies enabled

## Evidence

- [abstract] Our review is accompanied by a Jupyter notebook which illustrates the discussed concepts and provides a practical tool for applying ICA to the analysis of cancer omics datasets.: "Our review is accompanied by a Jupyter notebook which illustrates the discussed concepts and provides a practical tool for applying ICA to the analysis of cancer omics datasets."
- [abstract] Such works highlight the use of ICA in dimensionality reduction, deconvolution, data pre-processing, meta-analysis, and others applied to different data types: "Such works highlight the use of ICA in dimensionality reduction, deconvolution, data pre-processing, meta-analysis, and others applied to different data types"
- [intro] This notebook has been done with Python 3.7.3 (v3.7.3:ef4ec6ed12).: "This notebook has been done with Python 3.7.3 (v3.7.3:ef4ec6ed12)."
- [readme] using stability analysis and computing the optimal number of the components from their stabilities,: [evidence span withheld — anchor status: external_doc]
- [readme] AN R PACKAGE FOR IDENTIFYING  IMMUNE-RELATED SIGNALS  IN TRANSCRIPTOME THROUGH DECONVOLUTION OR UNSUPERVISED  SOURCE SEPARATION METHODS: [evidence span withheld — anchor status: external_doc]
