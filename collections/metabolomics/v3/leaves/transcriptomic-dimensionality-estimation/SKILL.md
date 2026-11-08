---
name: transcriptomic-dimensionality-estimation
description: Use when running repeated ICA (e.g. FastICA) decompositions on a transcriptomic
  expression matrix and you need to choose the decomposition order from stability
  behavior rather than arbitrarily.
when_to_use_negative:
- The decomposition order has already been fixed by downstream constraints and no
  stability-based dimensionality choice is needed
- The data are not transcriptomic/omics matrix-factorization inputs (e.g. already-reduced
  feature tables or non-matrix data), where ICA order selection via MSTD does not
  apply
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3316
tools:
- name: Jupyter notebook
  role: Interactive environment for running repeated ICA decompositions and stability-index
    computation accompanying the review
  repo: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- name: Python 3.7.3
  role: Runtime (with FastICA) used to execute the notebook's repeated decompositions
    and correlation-based clustering of components
  repo: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- name: BIODICA Navigator
  role: GUI/command-line environment computing independent components for omics data
    using stability analysis and deriving the optimal number of components from their
    stabilities
  repo: https://sysbio-curie.github.io/biodica-environment/
- name: pip
  role: Installs notebook dependencies from Jupyter_notebook_package_requirements.txt
  repo: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/ijms20184414
    title: Independent Component Analysis for Unraveling the Complexity of Cancer
      Omics Datasets
  source_content_hash: sha256:2f2188a5a6c4581c33e83970e248002866b5e03ad5888d7aed55bf35d79a1d4c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
  tools:
  - BIODICA Navigator
  - Jupyter notebook
  - Python
  - Python 3.7.3
  - pip
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# transcriptomic-dimensionality-estimation

## Summary

Determines the optimal number of independent components (decomposition order) for ICA applied to transcriptomic data using the MSTD criterion, where the order is chosen as the point at which the distribution of independent-component stability indices is no longer dominated by highly unstable components.

## When to use

Use when running repeated ICA (e.g. FastICA) decompositions on a transcriptomic expression matrix and you need to choose the decomposition order from stability behavior rather than arbitrarily.

## When NOT to use

- The decomposition order has already been fixed by downstream constraints and no stability-based dimensionality choice is needed
- The data are not transcriptomic/omics matrix-factorization inputs (e.g. already-reduced feature tables or non-matrix data), where ICA order selection via MSTD does not apply

## Inputs

- Transcriptomic expression matrix (genes x samples) loadable in the accompanying Jupyter notebook
- Range of candidate component orders (low to high dimensionality)
- Number of ICA restarts per order (e.g. FastICA)

## Outputs

- stability_indices.csv with one stability-index row per tested decomposition order
- Inferred MSTD value (optimal number of independent components)

## How to apply

Load the transcriptomic expression matrix in a Jupyter notebook (Python 3.7.3 environment; dependencies installed with pip from the review's Jupyter_notebook_package_requirements.txt). Run repeated FastICA decompositions over a range of component orders from low to high dimensionality, with multiple restarts per order. At each order, cluster the independent components across runs by correlating their sample-weight vectors to assess reproducibility, derive a stability index per component (e.g. from the correlation-based clustering), and average it per decomposition order. The MSTD is the smallest order at which the average stability index reaches a plateau; write the per-order stability indices to stability_indices.csv and record the inferred MSTD. Alternatively, BIODICA computes the optimal number of components directly from component stabilities via GUI or command line.

## Related tools

- **Jupyter notebook** (Interactive environment for running repeated ICA decompositions and stability-index computation accompanying the review) — https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- **Python 3.7.3** (Runtime (with FastICA) used to execute the notebook's repeated decompositions and correlation-based clustering of components) — https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- **BIODICA Navigator** (GUI/command-line environment computing independent components for omics data using stability analysis and deriving the optimal number of components from their stabilities) — https://sysbio-curie.github.io/biodica-environment/
- **pip** (Installs notebook dependencies from Jupyter_notebook_package_requirements.txt) — https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials

## Examples

```
pip install Jupyter_notebook_package_requirements.txt; jupyter notebook ICA_in_cancer_research.ipynb  # run repeated FastICA over orders, write stability_indices.csv
```

## Evaluation signals

- stability_indices.csv exists and contains exactly one stability-index row per tested decomposition order
- The reported MSTD corresponds to the plateau of the stability-index curve as described in Section 2.5 of the review
- Per-order average stability index is monotonically non-decreasing until the plateau is reached
- Each order was evaluated with multiple FastICA restarts, and components were clustered across runs by correlation of sample-weight vectors
- If using BIODICA, the reported component number matches the stabilities computed by the tool's stability analysis

## Limitations

- BIODICA's local folder name must not contain spaces or hyphens and must not be 'BIODICA-master', otherwise certain functions won't work
- BIODICA's Python ICA core requires the 'python' command to be available from the command line and the stabilized-ica package installed; the MATLAB core requires Java 1.8+ or the Docker image auranic/biodica
- The Python ICA implementation depends on the stabilized-ica package (https://github.com/ncaptier/stabilized-ica) being accessible

## Evidence

- [full_text] MSTD plateau criterion: [evidence span withheld — anchor status: not_found]
- [readme] BIODICA README stability claim: [evidence span withheld — anchor status: external_doc]
- [intro] Notebook environment: "This notebook has been done with Python 3.7.3 (v3.7.3:ef4ec6ed12)."
- [abstract] Review focus on optimal number of components: "We particularly focus on the technical aspects of ICA application in omics studies such as using different protocols, determining the optimal number of components"
