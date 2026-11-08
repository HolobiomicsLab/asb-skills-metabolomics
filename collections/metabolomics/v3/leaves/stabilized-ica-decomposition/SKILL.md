---
name: stabilized-ica-decomposition
description: Use when a samples-by-features cancer omics matrix (transcriptome, methylome,
  proteome, or single-cell) needs to be decomposed into independent, reproducible
  metagene/source components, or when ICA must be compared against PCA/NMF.
when_to_use_negative:
- The analysis goal is supervised feature selection or predictive modeling rather
  than unsupervised blind source separation of omics data
- The data matrix is not omics-shaped (samples × features) or the goal is deconvolution
  of immune cell fractions only, for which DeconICA's targeted deconvolution workflow
  is more appropriate
- A decomposition must be exactly reproduced rather than statistically reproducible
  — ICA component order and sign vary across randomized initializations
tools:
- name: Jupyter notebook
  role: Companion notebook illustrating ICA application to cancer omics datasets,
    including the metagene comparison workflow
  repo: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- name: BIODICA Navigator
  role: GUI/command-line environment computing independent components for omics data
    with stability analysis and stability-based optimal component number
  repo: https://github.com/LabBandSB/BIODICA
- name: stabilized-ica Python package
  role: Python ICA implementation used by BIODICA as the default computational core
    for stabilized decompositions
  repo: https://github.com/ncaptier/stabilized-ica
- name: Python 3.7.3
  role: Runtime environment for the companion ICA notebook
- name: pip
  role: Installs the notebook dependencies via the requirements file
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/ijms20184414
    title: Independent Component Analysis for Unraveling the Complexity of Cancer
      Omics Datasets
  source_content_hash: sha256:216eee0a2e957a934c8cec59dd31edb1f9e0142fd14d8a0f1d0b3019ac12b125
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
  - stabilized-ica Python package
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# stabilized-ica-decomposition

## Summary

Run stabilized Independent Component Analysis (ICA) over multiple randomized initializations on a cancer omics data matrix to extract robust, non-Gaussian metagene signatures, as reviewed and demonstrated in the IJMScancer-omics ICA review and its companion Jupyter notebook.

## When to use

Use when a samples-by-features cancer omics matrix (transcriptome, methylome, proteome, or single-cell) needs to be decomposed into independent, reproducible metagene/source components, or when ICA must be compared against PCA/NMF.

## When NOT to use

- The analysis goal is supervised feature selection or predictive modeling rather than unsupervised blind source separation of omics data
- The data matrix is not omics-shaped (samples × features) or the goal is deconvolution of immune cell fractions only, for which DeconICA's targeted deconvolution workflow is more appropriate
- A decomposition must be exactly reproduced rather than statistically reproducible — ICA component order and sign vary across randomized initializations

## Inputs

- Cancer omics data matrix (samples as rows, features as columns) loadable as a pandas DataFrame
- Number of components or a stability-based criterion for choosing the optimal number of components
- Multiple random seeds/initializations for stabilization

## Outputs

- Metagene (source) vectors and mixing matrix from the stabilized ICA decomposition
- Component stability scores used to select the optimal number of components
- Pairwise cross-decomposition correlation-hit counts and per-method reproducibility metrics (e.g., written to metagene_reproducibility_comparison.csv)

## How to apply

Load the omics matrix into a pandas DataFrame with samples as rows and features as columns (the companion notebook at github.com/sysbio-curie/ICA-in-Cancer-research-review-materials, Python 3.7.3, uses pip-installable dependencies). Run stabilized ICA over multiple randomized initializations so that stability analysis across runs yields robust metagene signatures; the stabilized-ica Python package is the computational core used by BIODICA Navigator, which also computes the optimal number of components from their stabilities. When benchmarking, run PCA and NMF with a comparable number of components on the same matrix, then compare metagene vectors via pairwise correlations using a reciprocal best hit framework (Reference [38] / Section 2.4), scoring a hit when a correlation exceeds the significance threshold. The review reports that stabilized ICA maximizing non-Gaussianity of metagenes was superior to PCA and NMF on reproducibility metrics, so aggregate correlation-hit counts into per-method reproducibility metrics to validate your own decomposition.

## Related tools

- **Jupyter notebook** (Companion notebook illustrating ICA application to cancer omics datasets, including the metagene comparison workflow) — https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- **BIODICA Navigator** (GUI/command-line environment computing independent components for omics data with stability analysis and stability-based optimal component number) — https://github.com/LabBandSB/BIODICA
- **stabilized-ica Python package** (Python ICA implementation used by BIODICA as the default computational core for stabilized decompositions) — https://github.com/ncaptier/stabilized-ica
- **Python 3.7.3** (Runtime environment for the companion ICA notebook)
- **pip** (Installs the notebook dependencies via the requirements file)

## Examples

```
java -cp BIODICA_GUI.jar BIODICAPipeLine -config config -sfile [path_to_metagene_file] -dotoppgene
```

## Evaluation signals

- metagene_reproducibility_comparison.csv exists and contains one row for each of the three methods (stabilized ICA, PCA, NMF)
- Every method row carries a numeric cross-decomposition correlation-hit metric (e.g., ICA-vs-ICA, ICA-vs-PCA, ICA-vs-NMF, PCA-vs-NMF hit counts)
- Stabilized ICA qualitatively reproduces the review's Section 2.4 result: higher reproducibility/generalizability metrics than PCA and NMF
- Component stabilities are computed over multiple randomized initializations and used to justify the chosen number of components
- The data matrix orientation is verified (samples as rows, features as columns) before decomposition

## Limitations

- Benchmarks in the review are qualitative reproductions of Section 2.4 (Reference [38]) rather than exact numeric replications
- The companion notebook environment is pinned to Python 3.7.3; the BIODICA folder name must not contain spaces or hyphens or certain functions fail
- BIODICA's MATLAB ICA core is kept only for historical/compatibility reasons; the default is the Python stabilized-ica core, and Java 1.8+ is required for BIODICA Navigator
- ICA results depend on randomized initializations; stabilization across runs is required for reproducible metagene definitions

## Evidence

- [other] The stabilized version of ICA maximizing non-Gaussianity of metagenes was found superior to other matrix factorization methods (PCA, NMF): [evidence span withheld — anchor status: not_found]
- [readme] BIODICA stability analysis: [evidence span withheld — anchor status: external_doc]
- [readme] BIODICA Python core: [evidence span withheld — anchor status: external_doc]
- [abstract] Review technical focus: "We particularly focus on the technical aspects of ICA application in omics studies such as using different protocols, determining the optimal number of components, assessing and improving"
- [intro] Notebook environment: "This notebook has been done with Python 3.7.3 (v3.7.3:ef4ec6ed12)."
