---
name: ica-stability-assessment
description: Use when decomposing transcriptomic or multi-omics data with ICA and
  component stability across random initializations is needed, or when assessing and
  improving ICA reproducibility and selecting the optimal number of components.
when_to_use_negative:
- When the ICA decomposition is already stabilized and only downstream analyses (e.g.
  functional interpretation) are needed — BIODICA explicitly notes 'It is not necessary
  to re-compute the ICA decomposition each time'
- When the data type is not suitable for ICA decomposition of omics signals (e.g.
  already a feature/summary table rather than a raw expression matrix)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BIODICA Navigator
  role: GUI/command-line environment for computing independent components of omics
    data using stability analysis and selecting the optimal number of components from
    their stabilities
  repo: https://github.com/LabBandSB/BIODICA
- name: DeconICA
  role: R package providing a vignette for running fastICA with icasso stabilisation
    to identify immune-related signals in transcriptome data
  repo: https://github.com/UrszulaCzerwinska/DeconICA
- name: stabilized-ica
  role: Python computational core used by BIODICA Navigator for stabilized ICA computation
  repo: https://github.com/ncaptier/stabilized-ica
- name: ICA-in-Cancer-research-review-materials
  role: Jupyter notebook illustrating ICA concepts and stabilization on cancer omics
    datasets (Python 3.7.3)
  repo: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/ijms20184414
    title: Independent Component Analysis for Unraveling the Complexity of Cancer
      Omics Datasets
  source_content_hash: sha256:452ac3a118f6b4422ecd63dc5946990e611dd92d70eb1717b35a5c28a222bb62
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LabBandSB/BIODICA
  tools:
  - BIODICA Navigator
  - DeconICA
  - ICA-in-Cancer-research-review-materials
  - Python
  - stabilized-ica
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# ica-stability-assessment

## Summary

Stabilizes Independent Component Analysis of transcriptomic/omics data by repeating fastICA with different random initializations, clustering the resulting components icasso-style, and retaining stable components with quality indices. This addresses the reproducibility and reliability issues of a single ICA run on cancer omics datasets.

## When to use

Use when decomposing transcriptomic or multi-omics data with ICA and component stability across random initializations is needed, or when assessing and improving ICA reproducibility and selecting the optimal number of components.

## When NOT to use

- When the ICA decomposition is already stabilized and only downstream analyses (e.g. functional interpretation) are needed — BIODICA explicitly notes 'It is not necessary to re-compute the ICA decomposition each time'
- When the data type is not suitable for ICA decomposition of omics signals (e.g. already a feature/summary table rather than a raw expression matrix)

## Inputs

- Preprocessed expression matrix (samples x genes) as a pandas DataFrame
- Number of ICA runs / random initializations (seeds)
- stabilized-ica Python package accessible from the command line

## Outputs

- stability_scores.csv (stability/quality index per component)
- metagene_matrix_oriented.csv (sign-oriented metagene matrix S, components x samples)
- Retained stable independent components with mixing matrices

## How to apply

Load the preprocessed expression matrix (samples x genes) and run sklearn.decomposition.FastICA many times with different random seeds, collecting the mixing matrix A and source matrix S from each run. Cluster all components across runs with icasso-style agglomerative clustering using a distance derived from absolute correlation between component loading vectors. Compute a stability/quality index per cluster (e.g. mean intra-cluster absolute correlation) and retain only stable components, whose stabilities also inform the optimal number of components. Finally fix sign orientation per component by computing the skewness of each source distribution and flipping S -> -S, A -> -A whenever the heaviest tail is negative. The BIODICA Navigator provides a GUI/command-line implementation of this workflow, and DeconICA offers an R equivalent ('Running fastICA with icasso stabilisation').

## Related tools

- **BIODICA Navigator** (GUI/command-line environment for computing independent components of omics data using stability analysis and selecting the optimal number of components from their stabilities) — https://github.com/LabBandSB/BIODICA
- **DeconICA** (R package providing a vignette for running fastICA with icasso stabilisation to identify immune-related signals in transcriptome data) — https://github.com/UrszulaCzerwinska/DeconICA
- **stabilized-ica** (Python computational core used by BIODICA Navigator for stabilized ICA computation) — https://github.com/ncaptier/stabilized-ica
- **ICA-in-Cancer-research-review-materials** (Jupyter notebook illustrating ICA concepts and stabilization on cancer omics datasets (Python 3.7.3)) — https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials

## Examples

```
from sklearn.decomposition import FastICA; ica = FastICA(n_components=50, random_state=seed); S = ica.fit_transform(X)
```

## Evaluation signals

- Each retained component has a computed stability/quality index (e.g. mean intra-cluster absolute correlation) recorded in stability_scores.csv
- Unstable components across random fastICA runs are excluded by the icasso-style agglomerative clustering on absolute-correlation distance
- After sign correction, the skewness (sk) of each oriented source distribution has its heaviest tail positive
- metagene_matrix_oriented.csv has shape components x samples consistent with the number of retained stable clusters
- In BIODICA, the number of components chosen from stability values should be reported alongside component stabilities

## Limitations

- BIODICA's MATLAB ICA core is 'currently, it is of interest only for historical and compatibility reasons'; the Python core requires the stabilized-ica package and Java 1.8+ for the GUI
- Folder names containing spaces or hyphens break certain BIODICA functions, and the suggested 'BIODICA-master' name must not be used
- Components must be recomputed if the decomposition itself changes; BIODICA does not require re-running decomposition only for downstream analyses, implying high cost of full re-stabilization

## Evidence

- [readme] BIODICA stability: [evidence span withheld — anchor status: external_doc]
- [readme] DeconICA icasso: [evidence span withheld — anchor status: external_doc]
- [abstract] review scope: "determining the optimal number of components, assessing and improving"
