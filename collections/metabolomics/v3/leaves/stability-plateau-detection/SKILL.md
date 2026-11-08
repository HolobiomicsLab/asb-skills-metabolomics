---
name: stability-plateau-detection
description: Use when applying ICA (e.g. FastICA) to a transcriptomic expression matrix
  and you must decide the decomposition order (number of components) from component
  reproducibility across restarts rather than an arbitrary choice.
when_to_use_negative:
- The decomposition order is already fixed by prior knowledge or a previous MSTD analysis
  — recomputing the stability plateau is unnecessary.
- The data are not omics expression data suitable for ICA source separation (e.g.
  already a feature table or metadata).
- Only a single ICA run is planned — stability indices require multiple restarts per
  component order to quantify reproducibility.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: Jupyter notebook
  role: Interactive environment in which repeated ICA decompositions and stability-index
    computation are performed
  repo: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- name: Python 3.7.3
  role: Python runtime (via FastICA) used to run decompositions over the component-order
    range and compute clustering-based stability indices
  repo: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- name: pip
  role: Installs notebook dependencies from Jupyter_notebook_package_requirements.txt
  repo: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- name: BIODICA Navigator
  role: GUI/command-line environment that computes ICA for omics data using stability
    analysis and derives the optimal number of components from component stabilities
  repo: https://github.com/LabBandSB/BIODICA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/ijms20184414
    title: Independent Component Analysis for Unraveling the Complexity of Cancer
      Omics Datasets
  source_content_hash: sha256:857f8a99746bd73b10b90fcdaecb8d64f9beb6fac1f68639ca60fde5fe554d30
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
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# stability-plateau-detection

## Summary

Determines the optimal number of independent components (MSTD — most stable transcriptomic dimension) for ICA on transcriptomic data by running repeated decompositions over a range of component orders, computing per-component stability indices, and selecting the smallest order at which average stability reaches a plateau.

## When to use

Use when applying ICA (e.g. FastICA) to a transcriptomic expression matrix and you must decide the decomposition order (number of components) from component reproducibility across restarts rather than an arbitrary choice.

## When NOT to use

- The decomposition order is already fixed by prior knowledge or a previous MSTD analysis — recomputing the stability plateau is unnecessary.
- The data are not omics expression data suitable for ICA source separation (e.g. already a feature table or metadata).
- Only a single ICA run is planned — stability indices require multiple restarts per component order to quantify reproducibility.

## Inputs

- transcriptomic expression matrix (genes x samples) from the review's accompanying Jupyter notebook repository
- range of candidate decomposition orders (low to high)
- number of ICA restarts per order

## Outputs

- stability_indices.csv containing one stability-index row per tested decomposition order
- inferred MSTD value (optimal number of independent components)

## How to apply

Load the transcriptomic expression matrix (e.g. from the Jupyter notebook accompanying the review at github.com/sysbio-curie/ICA-in-Cancer-research-review-materials, run with Python 3.7.3). Run repeated FastICA decompositions over a range of component orders, from low to high dimensionality, with multiple restarts per order. At each order, cluster the independent components across runs by correlating their sample-weight vectors to assess reproducibility, derive a stability index per component (e.g. from the correlation-based clustering), and average it per decomposition order. The MSTD is the smallest order at which the average stability index plateaus — the point where the distribution is no longer dominated by highly unstable components. Write the per-order stability indices to stability_indices.csv and record the inferred MSTD value.

## Related tools

- **Jupyter notebook** (Interactive environment in which repeated ICA decompositions and stability-index computation are performed) — https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- **Python 3.7.3** (Python runtime (via FastICA) used to run decompositions over the component-order range and compute clustering-based stability indices) — https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- **pip** (Installs notebook dependencies from Jupyter_notebook_package_requirements.txt) — https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- **BIODICA Navigator** (GUI/command-line environment that computes ICA for omics data using stability analysis and derives the optimal number of components from component stabilities) — https://github.com/LabBandSB/BIODICA

## Examples

```
pip install Jupyter_notebook_package_requirements.txt; jupyter notebook ICA_review.ipynb
```

## Evaluation signals

- stability_indices.csv exists and contains exactly one stability-index row per tested decomposition order
- The reported MSTD corresponds to the plateau of the stability-index curve as described in Section 2.5 of the review
- Average stability index per order increases with order and then flattens at the reported MSTD
- Per-order distributions of component stability indices are no longer dominated by highly unstable components at and above the MSTD

## Limitations

- Requires multiple ICA restarts per component order, so computation cost grows with both the order range and the number of restarts
- The MSTD is defined as a plateau onset, which can be sensitive to how stability indices are averaged and how the plateau is judged
- BIODICA-based alternative requires Java 1.8+ and the stabilized-ica Python package; the local folder name must not contain spaces or hyphens or certain functions fail

## Evidence

- [full_text] The MSTD criterion defines the transcriptomic decomposition order as the point where the distribution of independent-component stability indices is no longer dominated by highly unstable components.: [evidence span withheld — anchor status: not_found]
- [readme] This notebook has been done with Python 3.7.3 (v3.7.3:ef4ec6ed12).: "This notebook has been done with Python 3.7.3 (v3.7.3:ef4ec6ed12)."
- [readme] user-friendly environment for computation of independent components for omics data, using stability analysis and computing the optimal number of the components from their stabilities: "user-friendly environment for computation of independent components for omics data, 
using stability analysis and computing the optimal number of the components from their stabilities"
- [abstract] We particularly focus on the technical aspects of ICA application in omics studies such as using different protocols, determining the optimal number of components: "We particularly focus on the technical aspects of ICA application in omics studies such as using different protocols, determining the optimal number of components, assessing and improving"
