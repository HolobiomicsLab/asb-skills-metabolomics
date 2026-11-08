---
name: metagene-metasample-matrix-extraction
description: Use when a cancer omics dataset (e.g. transcriptome, methylome, proteome)
  must be decomposed into metagene and metasample components via ICA and the resulting
  matrices exported as artifacts.
when_to_use_negative:
- The goal is a GUI-driven or stability-optimized ICA run over large omics data —
  use BIODICA Navigator instead, which computes components with stability analysis
  and the optimal number of components
- The aim is immune-related signal deconvolution of transcriptome data specifically
  — the DeconICA R package is the dedicated tool for that task
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Jupyter notebook
  role: Interactive environment in which the example ICA decomposition is run to produce
    the sk and ak matrices
  repo: github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- name: Python 3.7.3
  role: Runtime used to execute the notebook's ICA decomposition code
- name: pip
  role: Installs the notebook dependencies from Jupyter_notebook_package_requirements.txt
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/ijms20184414
    title: Independent Component Analysis for Unraveling the Complexity of Cancer
      Omics Datasets
  source_content_hash: sha256:d3de85a0e464e733b079e029ffdf8d51545b3de33a946212ba1aeb12650e2bf0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter notebook
  - Python
  - Python 3.7.3
  - pip
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# metagene-metasample-matrix-extraction

## Summary

Extracts the source (metagene, sk) and mixing (metasample, ak) matrices from an Independent Component Analysis decomposition of a cancer omics data matrix, using the review's accompanying Jupyter notebook implemented in Python 3.7.3.

## When to use

Use when a cancer omics dataset (e.g. transcriptome, methylome, proteome) must be decomposed into metagene and metasample components via ICA and the resulting matrices exported as artifacts.

## When NOT to use

- The goal is a GUI-driven or stability-optimized ICA run over large omics data — use BIODICA Navigator instead, which computes components with stability analysis and the optimal number of components
- The aim is immune-related signal deconvolution of transcriptome data specifically — the DeconICA R package is the dedicated tool for that task

## Inputs

- Cancer omics dataset used by the notebook (e.g. transcriptome/methylome/proteome data matrix)
- Jupyter_notebook_package_requirements.txt dependency list

## Outputs

- Metagene (sk) matrix exported from the executed notebook
- Metasample (ak) matrix exported from the executed notebook
- Output plots and tables generated during the decomposition analysis

## How to apply

Set up the execution environment with Python 3.7.3 and install dependencies from Jupyter_notebook_package_requirements.txt with pip. Launch the Jupyter Notebook from github.com/sysbio-curie/ICA-in-Cancer-research-review-materials, which contains the example of Independent Component Analysis (ICA) accompanying the IJMS review. Load the cancer omics data matrix into the notebook environment and run the ICA decomposition cells end-to-end so the data matrix is factored into its source (metagene, sk) and mixing (metasample, ak) components. Export the sk and ak matrices and save the plots and tables generated during the decomposition analysis. The rationale is that the review positions the notebook as a practical tool for applying ICA to cancer omics data, and the decomposition is the concrete numerical output users need to reproduce.

## Related tools

- **Jupyter notebook** (Interactive environment in which the example ICA decomposition is run to produce the sk and ak matrices) — github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- **Python 3.7.3** (Runtime used to execute the notebook's ICA decomposition code)
- **pip** (Installs the notebook dependencies from Jupyter_notebook_package_requirements.txt)

## Examples

```
pip install Jupyter_notebook_package_requirements.txt && jupyter notebook
```

## Evaluation signals

- Notebook executes end-to-end without error under Python 3.7.3 with all pip dependencies installed
- The sk (metagene) and ak (metasample) matrices are produced as output artifacts of the executed notebook
- Accompanying plots and tables generated during the decomposition are saved as artifacts
- The decomposition reproduces the example of ICA accompanying the IJMS review as published in the repository

## Limitations

- The notebook is pinned to Python 3.7.3 and the dependency versions in Jupyter_notebook_package_requirements.txt; other versions are not guaranteed
- The notebook is a single illustrative example accompanying the review; it does not provide the stability analysis or optimal-number-of-components estimation offered by BIODICA
- The decomposition does not itself assess component stability or reproducibility — technical aspects like protocols and optimal component number are reviewed but handled elsewhere

## Evidence

- [abstract] abstract: "Our review is accompanied by a Jupyter notebook which illustrates the discussed concepts and provides a practical tool for applying ICA to the analysis of cancer omics datasets."
- [intro] intro: "This notebook has been done with Python 3.7.3 (v3.7.3:ef4ec6ed12)."
- [intro] intro-2: "All dependencies can be installed using:
pip install Jupyter_notebook_package_requirements.txt"
- [readme] readme: "This review is accompanied by interactive Jupyter notebook located at https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials."
