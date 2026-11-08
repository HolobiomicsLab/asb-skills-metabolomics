---
name: modified-cosine-spectral-similarity-scoring
description: Use when LC-MS/MS untargeted metabolomics data include MS/MS peak lists
  and the goal is to group features that are structurally similar before feature group-wise
  statistical testing in msFeaST.
when_to_use_negative:
- The dataset lacks MS/MS fragment spectra (only MS1 feature tables are available),
  so no spectral similarity can be computed.
- You already have a structurally informed feature grouping (e.g. from molecular networking
  or annotation) and only need statistical testing — globaltest can be run on that
  grouping directly without re-scoring spectra.
edam_operation: http://edamontology.org/operation_3644
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: msFeaST
  role: Python/R pipeline that computes the modified-cosine spectral similarity matrix
    and feeds it into k-medoids clustering prior to globaltest feature group-wise
    testing
  repo: https://github.com/kevinmildau/msFeaST
- name: globaltest
  role: Bioconductor package (version 5.56.0 via Bioconductor 3.18) used downstream
    of the similarity-based clustering to run feature group-wise statistical testing
    on the feature clusters
  repo: https://github.com/kevinmildau/msFeaST
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:b7de8d52be9612965075268de7246625d2b6e545bc1ec39b9507534fa5f92028
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - globaltest
  - msFeaST
  edam_operation: http://edamontology.org/operation_3644
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# modified-cosine-spectral-similarity-scoring

## Summary

Compute pairwise modified-cosine spectral similarity scores between LC-MS/MS features so that structurally similar metabolites can be grouped (via k-medoids) and statistically tested with globaltest in the msFeaST pipeline.

## When to use

Use when LC-MS/MS untargeted metabolomics data include MS/MS peak lists and the goal is to group features that are structurally similar before feature group-wise statistical testing in msFeaST.

## When NOT to use

- The dataset lacks MS/MS fragment spectra (only MS1 feature tables are available), so no spectral similarity can be computed.
- You already have a structurally informed feature grouping (e.g. from molecular networking or annotation) and only need statistical testing — globaltest can be run on that grouping directly without re-scoring spectra.

## Inputs

- Preprocessed LC-MS/MS feature intensity/abundance table (mushroom-type or substrate-contrast dataset)
- Per-feature MS/MS peak lists (fragment spectra)
- Sample group metadata (e.g. 0% vs 80% olive mill solid waste contrast or mushroom-type labels)

## Outputs

- Pairwise spectral similarity matrix between LC-MS/MS features
- k-medoids feature clusters used as structurally similar feature groups for globaltest

## How to apply

In the msFeaST workflow, take the MS/MS peak lists of every detected LC-MS/MS feature and compute the pairwise spectral similarity matrix using the modified cosine score, which allows a mass offset between matching fragments so that precursor-related losses do not penalize the score. Feed the resulting similarity matrix into k-medoids clustering so the resulting clusters serve as a proxy for grouping structurally similar features. The article's two deposited runs used exactly this score: one on the Pleurotus eryngii 0% vs 80% olive mill solid waste contrast and one comparing the different mushroom types; in both cases the similarity matrix, clustering, and globaltest feature group-wise testing were run on the preprocessed feature intensity/abundance table together with sample group metadata. Because the modified cosine score is computed before any statistical testing, its quality directly determines which features end up co-clustered and therefore which groups globaltest evaluates.

## Related tools

- **msFeaST** (Python/R pipeline that computes the modified-cosine spectral similarity matrix and feeds it into k-medoids clustering prior to globaltest feature group-wise testing) — https://github.com/kevinmildau/msFeaST
- **globaltest** (Bioconductor package (version 5.56.0 via Bioconductor 3.18) used downstream of the similarity-based clustering to run feature group-wise statistical testing on the feature clusters) — https://github.com/kevinmildau/msFeaST

## Examples

```
conda activate msfeast_environment && jupyter-notebook notebooks/msfeast_pipeline_mushroom_type_comparison.ipynb
```

## Evaluation signals

- The pairwise similarity matrix is square, symmetric, and covers every analyzed feature (n x n where n = number of LC-MS/MS features).
- k-medoids clustering on the modified-cosine matrix yields clusters whose members share plausible fragment losses / precursor offsets — spot-check that clusters are not dominated by singletons.
- Downstream globaltest runs on the derived clusters must execute without error in the msFeaST pipeline (e.g. for the 0% vs 80% olive mill solid waste contrast).
- For the deposited mushroom-type run, the resulting differential-intensity patterns should qualitatively match the deposited Section 3 output showing wide-scale feature highlighting.

## Limitations

- The msFeaST pre-processing and pipeline workflow has been tested on macOS and Linux; Windows support is not yet released.
- Conda's R package environment is not reliable for the required packages, so R dependencies (including globaltest) must be installed via remotes/BiocManager with pinned versions; a cached rscript path from a pre-existing terminal can cause package installation into the wrong R.
- k-medoids clustering on the similarity matrix is a proxy for structural similarity — no spectral library annotation is performed, so cluster quality depends entirely on the raw spectral match quality.

## Evidence

- [results] modified cosine score as similarity measure on substrate contrast: [evidence span withheld — anchor status: not_found]
- [abstract] spectral clustering rationale: "spectral similarity-based feature groupings generated using k-medoids clustering, where the resulting clusters serve as a proxy for grouping structurally similar features"
- [abstract] globaltest downstream of clustering: "Spectral clustering done in this way allows for feature group-wise statistical testing using the globaltest package"
- [readme] msFeaST reproducibility / install: [evidence span withheld — anchor status: external_doc]
- [abstract] repository availability: "The msFeaST workflow is freely available through https://github.com/kevinmildau/msFeaST and built to work on MacOS and Linux systems."
