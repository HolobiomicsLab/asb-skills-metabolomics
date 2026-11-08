---
name: skewness-based-sign-detection
description: Use when independent component analysis (e.g. sklearn FastICA or fastICA
  with icasso stabilisation) has been run on a samples x genes expression matrix and
  component signs must be fixed so each source's heaviest distribution tail is positive.
when_to_use_negative:
- The decomposition is not sign-ambiguous or an orientation convention has already
  been fixed by another method — flipping again would destroy comparability
- The source distributions are expected to be symmetric, so skewness carries no tail-direction
  information and the flip decision becomes arbitrary
- The task is deconvolution with a reference-based orientation (e.g. DeconICA-style
  immune deconvolution with known signatures) rather than unsupervised skewness-based
  orientation
tools:
- name: fastICA (R, via icasso stabilisation)
  role: Produces the ICA source and mixing matrices whose component signs are then
    oriented by the skewness rule; documented in the DeconICA vignette 'Running fastICA
    with icasso stabilisation'
  repo: https://urszulaczerwinska.github.io/DeconICA/
- name: Jupyter notebook (Python 3.7.3)
  role: Companion notebook implementing the ICA workflow on cancer omics data, including
    stabilization and orientation steps of the pipeline
  repo: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/ijms20184414
    title: Independent Component Analysis for Unraveling the Complexity of Cancer
      Omics Datasets
  source_content_hash: sha256:c1f9e2f9dc11741559d0f79c4da4cb00392b86f853bfd92c03aa1467874a31d1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter notebook (Python 3.7.3)
  - Python
  - fastICA (R, via icasso stabilisation)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# skewness-based-sign-detection

## Summary

Fixes the sign ambiguity inherent in ICA decompositions (e.g. fastICA) by checking the skewness of each independent component's source distribution and flipping the component (S -> -S, A -> -A) so that the heaviest tail is positive. This makes components from an icasso-stabilized ICA comparable and interpretable as oriented metagenes on transcriptomic data.

## When to use

Use when independent component analysis (e.g. sklearn FastICA or fastICA with icasso stabilisation) has been run on a samples x genes expression matrix and component signs must be fixed so each source's heaviest distribution tail is positive.

## When NOT to use

- The decomposition is not sign-ambiguous or an orientation convention has already been fixed by another method — flipping again would destroy comparability
- The source distributions are expected to be symmetric, so skewness carries no tail-direction information and the flip decision becomes arbitrary
- The task is deconvolution with a reference-based orientation (e.g. DeconICA-style immune deconvolution with known signatures) rather than unsupervised skewness-based orientation

## Inputs

- Source matrix S (components x samples) from fastICA / sklearn.decomposition.FastICA applied to a preprocessed expression matrix (samples x genes)
- Mixing matrix A from the same ICA decomposition, so sign flips can be applied consistently to both S and A
- Optionally, icasso cluster members (component loading vectors from multiple runs) whose signs must be harmonized

## Outputs

- Sign-oriented metagene matrix S (components x samples) written to metagene_matrix_oriented.csv
- Sign-corrected mixing matrix A with columns flipped consistently with S
- Per-component stability/quality index (mean intra-cluster absolute correlation) in stability_scores.csv

## How to apply

After collecting the source matrix S and mixing matrix A from each ICA run (or from the retained stable icasso clusters), compute the skewness (sk) of each component's source distribution across samples. Decide the sign per component: if the heaviest tail of the distribution is negative (negative skewness dominance), flip both the source vector and its corresponding mixing column (S -> -S, A -> -A); otherwise keep the original orientation. The convention to enforce is that the heaviest tail of sk is positive for every component. Apply this step after stability-based clustering (icasso-style agglomerative clustering on absolute-correlation distance) so that the final oriented metagene matrix (S, components x samples) is consistent across runs and can be written out alongside per-component stability scores. The DeconICA R package documents this pipeline as 'Running fastICA with icasso stabilisation', and the article's companion Jupyter notebook implements the same workflow in Python.

## Related tools

- **fastICA (R, via icasso stabilisation)** (Produces the ICA source and mixing matrices whose component signs are then oriented by the skewness rule; documented in the DeconICA vignette 'Running fastICA with icasso stabilisation') — https://urszulaczerwinska.github.io/DeconICA/
- **Jupyter notebook (Python 3.7.3)** (Companion notebook implementing the ICA workflow on cancer omics data, including stabilization and orientation steps of the pipeline) — https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials

## Examples

```
from scipy.stats import skew; sk = skew(S, axis=1); S_oriented = S * ((sk > 0)*2-1)[:, None]; A_oriented = A * ((sk > 0)*2-1)[None, :]
```

## Evaluation signals

- For every oriented component, the skewness of its source distribution has a positive-heaviest tail, i.e. re-computing skew after flipping confirms the convention
- The sign flip is applied consistently: flipping row i of S must correspond to flipping column i of A
- Oriented components from different ICA runs within the same icasso cluster align positively (high absolute correlation becomes high positive correlation after orientation)
- metagene_matrix_oriented.csv has shape (components x samples) matching S, and stability_scores.csv contains one stability index per retained cluster
- Downstream interpretation (e.g. gene loading vs. metagene sign relationships) remains consistent after orientation

## Limitations

- Sign orientation is a convention, not an inference: skewness only provides a deterministic tie-break when one tail is clearly heavier; near-symmetric components give unstable flips
- The flip must propagate to the mixing matrix; flipping only S or only A breaks the S ≈ A × (data) relationship
- For historical/compatibility reasons alternative ICA cores (e.g. the MATLAB implementation reachable through BIODICA's docker image auranic/biodica) may produce components needing the same orientation step but via different tooling

## Evidence

- [readme] DeconICA icasso stabilisation vignette: [evidence span withheld — anchor status: external_doc]
- [readme] BIODICA stability analysis: [evidence span withheld — anchor status: external_doc]
- [abstract] Companion notebook scope: "Our review is accompanied by a Jupyter notebook which illustrates the discussed concepts and provides a practical tool for applying ICA to the analysis of cancer omics datasets."
