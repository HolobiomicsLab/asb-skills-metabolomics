---
name: jupyter-notebook-end-to-end-execution
description: Use when the task is to reproduce or rerun a published ICA decomposition
  of cancer omics data via the review's companion notebook rather than reimplementing
  ICA from scratch.
when_to_use_negative:
- The goal is to perform stabilized ICA with a GUI or command-line pipeline over large
  omics compendia — use BIODICA Navigator instead of the companion notebook
- The goal is immune-related deconvolution of transcriptomes in R — use the DeconICA
  R package instead
- The input data or environment cannot support Python 3.7.3 and the pinned dependency
  versions of the requirements file
edam_operation: http://edamontology.org/operation_0004
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Jupyter Notebook
  role: Interactive notebook environment in which the example ICA on cancer omics
    data accompanying the IJMS review is executed cell-by-cell
  repo: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- name: Python 3.7.3
  role: Runtime interpreter (v3.7.3:ef4ec6ed12) in which the notebook was built and
    must be executed
- name: pip
  role: Installs all notebook dependencies from Jupyter_notebook_package_requirements.txt
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/ijms20184414
    title: Independent Component Analysis for Unraveling the Complexity of Cancer
      Omics Datasets
  source_content_hash: sha256:200f2940241594493fc1f7a7d5aac19a1392381099e281f6c7961f8a996d0776
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
  tools:
  - Jupyter Notebook
  - Jupyter notebook
  - Python
  - Python 3.7.3
  - pip
  edam_operation: http://edamontology.org/operation_0004
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# jupyter-notebook-end-to-end-execution

## Summary

Execute the interactive Jupyter notebook accompanying the IJMS review (DOI 10.3390/ijms20184414) end-to-end to reproduce an Independent Component Analysis (ICA) of a cancer omics dataset, decomposing the data matrix into metagene (sk) and metasample (ak) components.

## When to use

Use when the task is to reproduce or rerun a published ICA decomposition of cancer omics data via the review's companion notebook rather than reimplementing ICA from scratch.

## When NOT to use

- The goal is to perform stabilized ICA with a GUI or command-line pipeline over large omics compendia — use BIODICA Navigator instead of the companion notebook
- The goal is immune-related deconvolution of transcriptomes in R — use the DeconICA R package instead
- The input data or environment cannot support Python 3.7.3 and the pinned dependency versions of the requirements file

## Inputs

- Cancer omics dataset used by the notebook (omics data matrix to be decomposed)
- Jupyter notebook from github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- Jupyter_notebook_package_requirements.txt dependency list
- Python 3.7.3 environment

## Outputs

- Metagene (sk) source component matrix exported from the executed notebook
- Metasample (ak) mixing component matrix exported from the executed notebook
- Output plots generated during the ICA decomposition analysis
- Output tables generated during the ICA decomposition analysis

## How to apply

Set up the execution environment with Python 3.7.3 (v3.7.3:ef4ec6ed12), the exact version the authors used, since the notebook's dependency versions are pinned in Jupyter_notebook_package_requirements.txt. Install dependencies with pip using that requirements file, then launch the notebook obtained from github.com/sysbio-curie/ICA-in-Cancer-research-review-materials. Load the cancer omics dataset used by the notebook, and run the ICA decomposition cells in order, which decompose the omics data matrix into source (metagene, sk) and mixing (metasample, ak) components. Finally, export the sk and ak matrices and save the output plots and tables generated during the decomposition. The rationale is that the notebook is explicitly provided as a practical, reproducible tool so the ICA concepts of the review can be applied without reimplementation.

## Related tools

- **Jupyter Notebook** (Interactive notebook environment in which the example ICA on cancer omics data accompanying the IJMS review is executed cell-by-cell) — https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- **Python 3.7.3** (Runtime interpreter (v3.7.3:ef4ec6ed12) in which the notebook was built and must be executed)
- **pip** (Installs all notebook dependencies from Jupyter_notebook_package_requirements.txt)

## Examples

```
pip install Jupyter_notebook_package_requirements.txt
```

## Evaluation signals

- The notebook executes end-to-end without error in the Python 3.7.3 environment with the pinned dependencies
- The source (metagene, sk) and mixing (metasample, ak) matrices are produced as output artifacts
- The output plots and tables generated during the decomposition analysis are saved and populated
- The dependency installation via pip against Jupyter_notebook_package_requirements.txt completes successfully

## Limitations

- The notebook is tied to Python 3.7.3 (v3.7.3:ef4ec6ed12); other Python versions may break the pinned dependency set
- All dependencies must be installed from Jupyter_notebook_package_requirements.txt; deviating versions are not covered by the authors
- The notebook is a didactic example of ICA accompanying the review, not a production pipeline like BIODICA with stability analysis for choosing the optimal number of components

## Evidence

- [intro] Python 3.7.3 pin: "This notebook has been done with Python 3.7.3 (v3.7.3:ef4ec6ed12)."
- [intro] pip dependency installation: "All dependencies can be installed using: pip install Jupyter_notebook_package_requirements.txt"
- [abstract] notebook as practical ICA tool: "Our review is accompanied by a Jupyter notebook which illustrates the discussed concepts and provides a practical tool for applying ICA to the analysis of cancer omics datasets."
- [readme] requirements file location: "Dependencies and versions are available in the [Requirements](./Jupyter_notebook_package_requirements.txt) file."
