---
name: fastica-decomposition
description: Use when the input is a samples x genes expression matrix (e.g., transcriptomic
  data from tumoral samples) and the goal is to extract independent, interpretable
  metagene components that are stable across random ICA initializations.
when_to_use_negative:
- The decomposition does not need to be reproducible across initializations (e.g.,
  a single exploratory decomposition is sufficient and re-computation cost is prohibitive).
- The input is not an expression/matrix dataset suited to blind source separation
  (e.g., raw reads or feature tables requiring preprocessing first).
tools:
- name: sklearn.decomposition.FastICA
  role: Python core computing the fastICA decomposition (mixing matrix A and source
    matrix S) per random initialization
- name: BIODICA Navigator
  role: GUI/command-line environment that computes independent components for omics
    data with stability analysis and derives the optimal number of components from
    their stabilities
  repo: https://github.com/LabBandSB/BIODICA
- name: DeconICA
  role: R package offering a vignette for running fastICA with icasso stabilisation
    to identify immune-related signals in transcriptomes
  repo: https://github.com/UrszulaCzerwinska/DeconICA
- name: stabilized-ica
  role: Python package used by BIODICA Navigator as its default stabilized ICA computational
    core
  repo: https://github.com/ncaptier/stabilized-ica
- name: Jupyter notebook
  role: Interactive notebook accompanying the review that illustrates applying ICA
    to cancer omics datasets (Python 3.7.3)
  repo: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/ijms20184414
    title: Independent Component Analysis for Unraveling the Complexity of Cancer
      Omics Datasets
  source_content_hash: sha256:515c29f1c832aa19029dcf982dccfdfba6eb8a841337f0f30fe0d707d5b577d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LabBandSB/BIODICA
  tools:
  - BIODICA Navigator
  - DeconICA
  - Jupyter notebook
  - Python
  - sklearn.decomposition.FastICA
  - stabilized-ica
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# fastica-decomposition

## Summary

Decomposes an omics (e.g., transcriptomic) expression matrix into statistically independent components using fastICA, stabilized across multiple random initializations via icasso-style clustering so that the retained components are reproducible rather than initialization-dependent.

## When to use

Use when the input is a samples x genes expression matrix (e.g., transcriptomic data from tumoral samples) and the goal is to extract independent, interpretable metagene components that are stable across random ICA initializations.

## When NOT to use

- The decomposition does not need to be reproducible across initializations (e.g., a single exploratory decomposition is sufficient and re-computation cost is prohibitive).
- The input is not an expression/matrix dataset suited to blind source separation (e.g., raw reads or feature tables requiring preprocessing first).

## Inputs

- Preprocessed expression matrix (samples x genes) as a pandas DataFrame / CSV
- Number of components to extract and number of random initializations/seeds

## Outputs

- stability_scores.csv — stability/quality index per retained component
- metagene_matrix_oriented.csv — sign-oriented metagene matrix S (components x samples)

## How to apply

Load the preprocessed expression matrix (samples x genes) into a pandas DataFrame and run sklearn.decomposition.FastICA many times with different random initializations/seeds, collecting the mixing matrix A and source matrix S from each run. Cluster all components across runs with icasso-style agglomerative clustering on a distance derived from absolute correlation between component loading vectors, and compute a stability index (e.g., mean intra-cluster absolute correlation) per cluster, retaining only stable components. Fix the sign orientation per component by computing the skewness (sk) of each source distribution and flipping the sign (S -> -S, A -> -A) whenever the heaviest tail is negative, so the heaviest tail of sk is positive. Write stability scores per component to stability_scores.csv and the sign-oriented metagene matrix (S, components x samples) to metagene_matrix_oriented.csv. As an alternative to manual fastICA, BIODICA computes independent components for omics data using stability analysis to determine the optimal number of components, and DeconICA provides an R vignette for running fastICA with icasso stabilisation.

## Related tools

- **sklearn.decomposition.FastICA** (Python core computing the fastICA decomposition (mixing matrix A and source matrix S) per random initialization)
- **BIODICA Navigator** (GUI/command-line environment that computes independent components for omics data with stability analysis and derives the optimal number of components from their stabilities) — https://github.com/LabBandSB/BIODICA
- **DeconICA** (R package offering a vignette for running fastICA with icasso stabilisation to identify immune-related signals in transcriptomes) — https://github.com/UrszulaCzerwinska/DeconICA
- **stabilized-ica** (Python package used by BIODICA Navigator as its default stabilized ICA computational core) — https://github.com/ncaptier/stabilized-ica
- **Jupyter notebook** (Interactive notebook accompanying the review that illustrates applying ICA to cancer omics datasets (Python 3.7.3)) — https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials

## Examples

```
java -cp BIODICA_GUI.jar BIODICAPipeLine
```

## Evaluation signals

- Stability index (mean intra-cluster absolute correlation) computed per component cluster; only components passing the stability threshold are retained
- Sign orientation invariant: after correction, the heaviest tail of the skewness (sk) of every oriented source distribution is positive
- Output schema check: stability_scores.csv has one stability score per retained component; metagene_matrix_oriented.csv is components x samples
- Cross-run agreement: components from different fastICA random seeds cluster together under absolute-correlation-based agglomerative clustering
- BIODICA runs report the optimal number of components derived from component stabilities

## Limitations

- fastICA results are initialization-dependent; single un-stabilized runs are not reproducible, hence multiple runs with different seeds are required.
- BIODICA installation is sensitive: the local folder name must not contain spaces or hyphens, 'BIODICA-master' must not be used, and Java 1.8+ is required.
- The MATLAB ICA core in BIODICA is maintained only for historical/compatibility reasons; the Python (stabilized-ica) core is the default.
- Re-computing ICA is unnecessary if decompositions were computed earlier (with BIODICA or not); downstream analyses can be applied to prior results.

## Evidence

- [full_text] Stabilization of ICA components is performed by re-computing the ICA decomposition multiple times with different random initializations: [evidence span withheld — anchor status: not_found]
- [readme] BIODICA stability analysis: [evidence span withheld — anchor status: external_doc]
- [readme] DeconICA icasso vignette: [evidence span withheld — anchor status: external_doc]
- [abstract] Jupyter notebook purpose: "Our review is accompanied by a Jupyter notebook which illustrates the discussed concepts and provides a practical tool for applying ICA to the analysis of cancer omics datasets."
- [readme] stabilized-ica dependency: [evidence span withheld — anchor status: external_doc]
