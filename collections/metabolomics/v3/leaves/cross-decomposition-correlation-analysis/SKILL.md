---
name: cross-decomposition-correlation-analysis
description: Use when you need to assess which matrix factorization method produces
  the most reproducible metagene signatures from the same omics matrix, e.g. by correlating
  ICA, PCA, and NMF components against a significance threshold.
when_to_use_negative:
- When the goal is to use a single decomposition for downstream deconvolution or pre-processing
  (e.g. immune deconvolution via DeconICA) rather than comparing reproducibility across
  methods
- When only one factorization method is of interest and no cross-decomposition comparison
  is required
- When the input is a pre-computed component/metagene table with no access to the
  original sample-by-feature data matrix needed to refit all three decompositions
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Jupyter notebook
  role: Companion notebook from the review used to run the ICA, PCA, and NMF decompositions
    on the cancer omics matrix and the correlation-based reproducibility comparison
  repo: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- name: BIODICA
  role: GUI/command-line environment for computing independent components for omics
    data using stability analysis and computing the optimal number of components from
    their stabilities
  repo: https://github.com/LabBandSB/BIODICA
- name: stabilized-ica
  role: Python package providing the stabilized ICA implementation used as the default
    computational core of BIODICA
  repo: https://github.com/ncaptier/stabilized-ica
- name: DeconICA
  role: R package applying ICA-based deconvolution of transcriptomic data; its icasso-stabilised
    fastICA vignette is relevant to stabilizing ICA runs before cross-method comparison
  repo: https://github.com/UrszulaCzerwinska/DeconICA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/ijms20184414
    title: Independent Component Analysis for Unraveling the Complexity of Cancer
      Omics Datasets
  source_content_hash: sha256:c6b4be66db770834c54aa737eee7201d319a1d55a083927811397d27de1fa8d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
  tools:
  - BIODICA
  - DeconICA
  - Jupyter notebook
  - Python
  - pip
  - stabilized-ica
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cross-decomposition-correlation-analysis

## Summary

Compares metagene definitions obtained from different matrix factorization methods (stabilized ICA, PCA, NMF) on the same cancer omics data matrix using pairwise correlations and a reciprocal-best-hit style reproducibility framework, to determine which decomposition yields the most reproducible and generalizable metagenes.

## When to use

Use when you need to assess which matrix factorization method produces the most reproducible metagene signatures from the same omics matrix, e.g. by correlating ICA, PCA, and NMF components against a significance threshold.

## When NOT to use

- When the goal is to use a single decomposition for downstream deconvolution or pre-processing (e.g. immune deconvolution via DeconICA) rather than comparing reproducibility across methods
- When only one factorization method is of interest and no cross-decomposition comparison is required
- When the input is a pre-computed component/metagene table with no access to the original sample-by-feature data matrix needed to refit all three decompositions

## Inputs

- Cancer omics data matrix (e.g. transcriptomic/methylation data) as a pandas DataFrame with samples as rows and features as columns
- Number of components (chosen to be comparable across ICA, PCA, and NMF decompositions)
- Significance threshold for correlation hits from the reciprocal best hit framework (Reference [38])

## Outputs

- metagene_reproducibility_comparison.csv with one row per method (and method-pair) and columns for each reproducibility metric
- Pairwise correlation matrices/hit counts between metagenes of ICA, PCA, and NMF decompositions
- Per-method reproducibility and generalizability metrics

## How to apply

Load the cancer omics data matrix (samples as rows, features as columns) into a pandas DataFrame from the companion notebook repository (github.com/sysbio-curie/ICA-in-Cancer-research-review-materials). Run stabilized ICA over multiple randomized initializations to obtain robust metagene signatures, and run PCA and NMF on the same matrix with a comparable number of components so the three decompositions are directly comparable. Compute pairwise correlations between the mixing/source vectors of each decomposition pair (ICA-vs-ICA across stabilizing runs, ICA-vs-PCA, ICA-vs-NMF, PCA-vs-NMF) and record a hit when a correlation exceeds the significance threshold of the reciprocal best hit framework of Reference [38]. Aggregate correlation-hit counts into per-method reproducibility and generalizability metrics following Section 2.4, and write them to metagene_reproducibility_comparison.csv with one row per method/method-pair. The rationale is that stabilized ICA maximizing non-Gaussianity of metagenes was reported as superior to PCA and NMF on these reproducibility metrics.

## Related tools

- **Jupyter notebook** (Companion notebook from the review used to run the ICA, PCA, and NMF decompositions on the cancer omics matrix and the correlation-based reproducibility comparison) — https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- **BIODICA** (GUI/command-line environment for computing independent components for omics data using stability analysis and computing the optimal number of components from their stabilities) — https://github.com/LabBandSB/BIODICA
- **stabilized-ica** (Python package providing the stabilized ICA implementation used as the default computational core of BIODICA) — https://github.com/ncaptier/stabilized-ica
- **DeconICA** (R package applying ICA-based deconvolution of transcriptomic data; its icasso-stabilised fastICA vignette is relevant to stabilizing ICA runs before cross-method comparison) — https://github.com/UrszulaCzerwinska/DeconICA

## Examples

```
pip install Jupyter_notebook_package_requirements.txt
```

## Evaluation signals

- metagene_reproducibility_comparison.csv exists and contains a row for each of the three methods (stabilized ICA, PCA, NMF)
- Every method row has a numeric cross-decomposition correlation-hit metric (no missing/NaN values in the reproducibility columns)
- PCA and NMF were run with a number of components comparable to the ICA decomposition so results are directly comparable
- The comparison reproduces the review's Section 2.4 result qualitatively: stabilized ICA maximizing non-Gaussianity of metagenes outperforms PCA and NMF on reproducibility metrics

## Limitations

- ICA results are initialization-dependent; the workflow requires repeating the decomposition over multiple randomized initializations (stabilization) before correlations are meaningful
- BIODICA's MATLAB ICA core is retained mainly for historical and compatibility reasons; the Python stabilized-ica core is the default
- The companion notebook was built with Python 3.7.3 and a specific package requirements file, so dependency versions must be pinned for reproduction

## Evidence

- [abstract] Our review is accompanied by a Jupyter notebook which illustrates the discussed concepts and provides a practical tool for applying ICA to the analysis of cancer omics datasets.: "Our review is accompanied by a Jupyter notebook which illustrates the discussed concepts and provides a practical tool for applying ICA to the analysis of cancer omics datasets."
- [abstract] We particularly focus on the technical aspects of ICA application in omics studies such as using different protocols, determining the optimal number of components, assessing and improving: "We particularly focus on the technical aspects of ICA application in omics studies such as using different protocols, determining the optimal number of components, assessing and improving"
- [readme] Jupyter Notebook containing example of Independent Component Analysis (ICA) accompanying the review in Internal Journal of Molecular Sciences (IJMS): "Jupyter Notebook containing example of Independent Component Analysis (ICA) accompanying the review in Internal Journal of Molecular Sciences (IJMS)"
- [readme] using stability analysis and computing the optimal number of the components from their stabilities: [evidence span withheld — anchor status: external_doc]
- [intro] This notebook has been done with Python 3.7.3 (v3.7.3:ef4ec6ed12).: "This notebook has been done with Python 3.7.3 (v3.7.3:ef4ec6ed12)."
