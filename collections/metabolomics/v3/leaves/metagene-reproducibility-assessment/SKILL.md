---
name: metagene-reproducibility-assessment
description: Use when you must decide which matrix factorization method yields the
  most reproducible metagene/source signatures, by cross-comparing decompositions
  (ICA-vs-ICA, ICA-vs-PCA, ICA-vs-NMF, PCA-vs-NMF) of one shared data matrix.
when_to_use_negative:
- You already have a single fixed decomposition and only want to interpret or annotate
  its metagenes — no cross-method comparison is needed
- The task is immune signal deconvolution from transcriptome rather than benchmarking
  factorization reproducibility (DeconICA's dedicated use case)
- The data matrix cannot be decomposed with a comparable number of components across
  methods, making correlations non-comparable
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Jupyter notebook
  role: Companion notebook providing the example cancer omics dataset and practical
    tool for applying ICA used as the shared input for the comparison
  repo: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- name: stabilized-ica Python package
  role: Python implementation of stabilized ICA (default core of BIODICA) used to
    produce robust metagene signatures across multiple randomized initializations
  repo: https://github.com/ncaptier/stabilized-ica
- name: BIODICA Navigator
  role: GUI/command-line environment for computing independent components with stability
    analysis and deriving the optimal number of components from their stabilities
  repo: https://sysbio-curie.github.io/biodica-environment/
- name: pip
  role: Installs the notebook package requirements needed to run the ICA comparison
    workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/ijms20184414
    title: Independent Component Analysis for Unraveling the Complexity of Cancer
      Omics Datasets
  source_content_hash: sha256:453e1f636439425e005b015c3db0fa6fe93cc4cd45670482bd73276e1fcc879f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
  tools:
  - BIODICA Navigator
  - Jupyter notebook
  - Python
  - pip
  - stabilized-ica Python package
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# metagene-reproducibility-assessment

## Summary

Benchmark the reproducibility and generalizability of metagene definitions produced by competing matrix factorization methods (stabilized ICA, PCA, NMF) on the same cancer omics data matrix, using reciprocal best hit correlation analysis as in the article's Section 2.4 / Reference [38].

## When to use

Use when you must decide which matrix factorization method yields the most reproducible metagene/source signatures, by cross-comparing decompositions (ICA-vs-ICA, ICA-vs-PCA, ICA-vs-NMF, PCA-vs-NMF) of one shared data matrix.

## When NOT to use

- You already have a single fixed decomposition and only want to interpret or annotate its metagenes — no cross-method comparison is needed
- The task is immune signal deconvolution from transcriptome rather than benchmarking factorization reproducibility (DeconICA's dedicated use case)
- The data matrix cannot be decomposed with a comparable number of components across methods, making correlations non-comparable

## Inputs

- Cancer omics data matrix (pandas DataFrame, samples as rows, features as columns) from the companion notebook
- Number of components comparable across ICA, PCA, and NMF decompositions
- Correlation significance threshold from the reciprocal best hit framework (Reference [38])

## Outputs

- metagene_reproducibility_comparison.csv with one row per method (and method-pair) and columns for each reproducibility metric
- Per-method reproducibility and generalizability metrics (cross-decomposition correlation-hit counts)

## How to apply

Load the cancer omics data matrix from the companion Jupyter notebook (github.com/sysbio-curie/ICA-in-Cancer-research-review-materials) into a pandas DataFrame with samples as rows and features as columns. Run stabilized Independent Component Analysis over multiple randomized initializations to obtain robust metagene signatures — the stabilized-ica Python package is the ICA core used by BIODICA Navigator — and run PCA and NMF on the same matrix with a comparable number of components so the three decompositions are directly comparable. Compute pairwise correlations between the metagene (mixing/source) vectors of each decomposition pair, recording a hit when a correlation exceeds the significance threshold of the review's reciprocal best hit framework. Aggregate correlation-hit counts into per-method and per-pair reproducibility/generalizability metrics following Section 2.4, and write them to metagene_reproducibility_comparison.csv. The rationale, per the article's finding, is that stabilized ICA maximizing non-Gaussianity of metagenes outperforms PCA and NMF on these reproducibility metrics.

## Related tools

- **Jupyter notebook** (Companion notebook providing the example cancer omics dataset and practical tool for applying ICA used as the shared input for the comparison) — https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- **stabilized-ica Python package** (Python implementation of stabilized ICA (default core of BIODICA) used to produce robust metagene signatures across multiple randomized initializations) — https://github.com/ncaptier/stabilized-ica
- **BIODICA Navigator** (GUI/command-line environment for computing independent components with stability analysis and deriving the optimal number of components from their stabilities) — https://sysbio-curie.github.io/biodica-environment/
- **pip** (Installs the notebook package requirements needed to run the ICA comparison workflow)

## Examples

```
pip install Jupyter_notebook_package_requirements.txt
```

## Evaluation signals

- metagene_reproducibility_comparison.csv exists and contains a row for each of the three methods (stabilized ICA, PCA, NMF)
- Every method has a numeric cross-decomposition correlation-hit metric in the CSV
- The companion notebook reproduces the comparison qualitatively as reported in Section 2.4, where stabilized ICA scores highest on reproducibility metrics
- All three decompositions were run on the same data matrix with a comparable number of components
- ICA stabilization is confirmed by repeating the decomposition over multiple randomized initializations

## Limitations

- The comparison is qualitative relative to the paper's Section 2.4; the exact significance threshold is inherited from Reference [38] and must be matched to reproduce the ranking
- Fair comparison requires the same number of components across ICA, PCA, and NMF; mismatched component counts break the correlation-hit comparison
- Stabilized ICA is computationally heavier than PCA/NMF because it repeats the decomposition over multiple randomized initializations

## Evidence

- [abstract] companion notebook provides dataset and ICA tool: "Our review is accompanied by a Jupyter notebook which illustrates the discussed concepts and provides a practical tool for applying ICA to the analysis of cancer omics datasets."
- [abstract] review covers reproducibility and comparison with other factorizations: "We particularly focus on the technical aspects of ICA application in omics studies such as using different protocols, determining the optimal number of components, assessing and improving"
- [readme] stabilized ICA core required by BIODICA: [evidence span withheld — anchor status: external_doc]
- [readme] BIODICA stability analysis derives optimal component number: [evidence span withheld — anchor status: external_doc]
- [intro] notebook Python version and dependency install: "This notebook has been done with Python 3.7.3 (v3.7.3:ef4ec6ed12)."
