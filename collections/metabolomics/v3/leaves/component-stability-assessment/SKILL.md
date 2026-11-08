---
name: component-stability-assessment
description: Use when you need to choose the decomposition order (number of components)
  for ICA applied to transcriptomic/cancer omics data, and want the choice justified
  by across-run stability rather than by variance-explained or visual inspection.
when_to_use_negative:
- The component order is already fixed by prior knowledge or an existing BIODICA/s-ICA
  decomposition — stability re-computation is unnecessary (BIODICA notes it is not
  necessary to re-compute the decomposition each time)
- The analysis goal is downstream interpretation (e.g. transcriptome deconvolution
  of immune signals via DeconICA) rather than determining the number of components
- The input is not a matrix suitable for unsupervised source separation (e.g. already
  a per-component feature or annotation table)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BIODICA Navigator
  role: GUI/command-line environment that computes ICA components using stability
    analysis and derives the optimal number of components from their stabilities
  repo: https://sysbio-curie.github.io/biodica-environment/
- name: stabilized-ica
  role: Python package providing the default computational core BIODICA uses for stabilized
    ICA decomposition and stability estimation
  repo: https://github.com/ncaptier/stabilized-ica
- name: Jupyter notebook (ICA-in-Cancer-research-review-materials)
  role: Reference Python 3.7.3 notebook illustrating ICA application to cancer omics
    data, including determining the optimal number of components
  repo: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/ijms20184414
    title: Independent Component Analysis for Unraveling the Complexity of Cancer
      Omics Datasets
  source_content_hash: sha256:860fd7249b574b1b501efaddd0ed61e0a624e6e8425bdad6adc91dd32a243296
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BIODICA Navigator
  - Jupyter notebook (ICA-in-Cancer-research-review-materials)
  - Python
  - pip
  - stabilized-ica
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# component-stability-assessment

## Summary

Determines the optimal number of independent components (decomposition order) for ICA on transcriptomic data using the MSTD criterion: the order at which the distribution of independent-component stability indices is no longer dominated by highly unstable components. It is the reproducibility-based alternative to ad-hoc dimensionality choice for omics ICA.

## When to use

Use when you need to choose the decomposition order (number of components) for ICA applied to transcriptomic/cancer omics data, and want the choice justified by across-run stability rather than by variance-explained or visual inspection.

## When NOT to use

- The component order is already fixed by prior knowledge or an existing BIODICA/s-ICA decomposition — stability re-computation is unnecessary (BIODICA notes it is not necessary to re-compute the decomposition each time)
- The analysis goal is downstream interpretation (e.g. transcriptome deconvolution of immune signals via DeconICA) rather than determining the number of components
- The input is not a matrix suitable for unsupervised source separation (e.g. already a per-component feature or annotation table)

## Inputs

- Transcriptomic expression matrix (genes x samples) as loaded in the review's accompanying Jupyter notebook
- Range of candidate ICA component orders (low to high dimensionality)
- Number of ICA restarts per component order (multiple runs, e.g. FastICA)

## Outputs

- stability_indices.csv with one stability-index row per tested decomposition order
- Inferred MSTD value (optimal number of independent components)
- Per-order average stability-index curve showing the plateau

## How to apply

Load the transcriptomic expression matrix (e.g. from the sysbio-curie/ICA-in-Cancer-research-review-materials Jupyter notebook, run with Python 3.7.3) and run repeated FastICA decompositions over a range of component orders from low to high dimensionality, with multiple restarts per order. At each order, cluster the independent components across runs by correlating their sample-weight vectors to assess reproducibility. Derive a stability index per independent component from the correlation-based clustering and average it per decomposition order, writing the per-order values to stability_indices.csv. The MSTD is identified as the smallest component order at which the average stability index reaches a plateau, matching Section 2.5 of the review. The same stability-based logic is automated in BIODICA Navigator, which computes the optimal number of components from component stabilities.

## Related tools

- **BIODICA Navigator** (GUI/command-line environment that computes ICA components using stability analysis and derives the optimal number of components from their stabilities) — https://sysbio-curie.github.io/biodica-environment/
- **stabilized-ica** (Python package providing the default computational core BIODICA uses for stabilized ICA decomposition and stability estimation) — https://github.com/ncaptier/stabilized-ica
- **Jupyter notebook (ICA-in-Cancer-research-review-materials)** (Reference Python 3.7.3 notebook illustrating ICA application to cancer omics data, including determining the optimal number of components) — https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials

## Examples

```
java -cp BIODICA_GUI.jar BIODICAPipeLine
```

## Evaluation signals

- stability_indices.csv exists and contains exactly one stability-index row per tested decomposition order
- The reported MSTD coincides with the plateau of the average stability-index curve, as described in Section 2.5 of the review
- Stability indices are derived from correlation-based clustering of components' sample-weight vectors across repeated ICA runs, not from a single decomposition
- Stability is computed across multiple restarts at each component order, so index values are averaged over restarts per order
- The inferred order is reproducible when the restart procedure is repeated with the same range of orders

## Limitations

- The MSTD plateau must be visually/analytically identified, so the criterion is somewhat qualitative when the stability curve plateaus gradually
- Computational cost grows with the range of component orders and the number of restarts, since each order requires repeated FastICA runs plus cross-run clustering
- Stability results computed earlier can be reused (BIODICA explicitly supports re-analyzing prior decompositions), so recomputing decompositions unnecessarily is a workflow inefficiency

## Evidence

- [abstract] We particularly focus on the technical aspects of ICA application in omics studies such as using different protocols, determining the optimal number of components, assessing and improving: "We particularly focus on the technical aspects of ICA application in omics studies such as using different protocols, determining the optimal number of components, assessing and improving"
- [readme] BIODICA stability-based order selection: [evidence span withheld — anchor status: external_doc]
- [intro] Notebook environment for reproducing the stability workflow: "This notebook has been done with Python 3.7.3 (v3.7.3:ef4ec6ed12)."
- [abstract] Notebook as practical tool for ICA on cancer omics datasets: "Our review is accompanied by a Jupyter notebook which illustrates the discussed concepts and provides a practical tool for applying ICA to the analysis of cancer omics datasets."
