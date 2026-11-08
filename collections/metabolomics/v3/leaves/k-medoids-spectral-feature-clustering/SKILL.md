---
name: k-medoids-spectral-feature-clustering
description: Use when processing untargeted LC-MS/MS metabolomics feature tables paired
  with MS/MS peak lists and you need structurally similar features grouped so clusters
  can serve as feature sets in group-wise statistical testing.
when_to_use_negative:
- Input is a single-feature (non-grouped) analysis where spectral structural similarity
  is irrelevant, e.g. targeted quantification of known analytes.
- The dataset lacks MS/MS spectra — k-medoids here clusters on spectral similarity,
  so features without MS/MS peak lists cannot be meaningfully clustered this way.
- Features are already assigned to chemically informed substructure groups (e.g. substructure-based
  grouping) and spectral clustering would duplicate that grouping.
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3520
tools:
- name: msFeaST
  role: Provides the k-medoids spectral clustering step on the modified-cosine similarity
    matrix within its Python/R pipeline notebooks
  repo: https://github.com/kevinmildau/msFeaST
- name: globaltest
  role: Consumes the k-medoids clusters as feature sets for group-wise statistical
    testing
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:2946475e4fc46a551c453aee1e830c87ad654323e216c2488ecffcf38b57c4a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - globaltest
  - msFeaST
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# k-medoids-spectral-feature-clustering

## Summary

Groups LC-MS/MS features into structurally similar clusters by applying k-medoids clustering to a pairwise spectral similarity matrix (e.g. modified cosine score), producing cluster labels used downstream for feature group-wise statistical testing with globaltest.

## When to use

Use when processing untargeted LC-MS/MS metabolomics feature tables paired with MS/MS peak lists and you need structurally similar features grouped so clusters can serve as feature sets in group-wise statistical testing.

## When NOT to use

- Input is a single-feature (non-grouped) analysis where spectral structural similarity is irrelevant, e.g. targeted quantification of known analytes.
- The dataset lacks MS/MS spectra — k-medoids here clusters on spectral similarity, so features without MS/MS peak lists cannot be meaningfully clustered this way.
- Features are already assigned to chemically informed substructure groups (e.g. substructure-based grouping) and spectral clustering would duplicate that grouping.

## Inputs

- Quantitative LC-MS/MS feature abundance table (samples × features)
- MS/MS peak lists per feature (mz/intensity pairs)
- Sample group metadata defining the biological contrast (e.g. 0% vs 80% olive mill solid waste)

## Outputs

- Cluster assignments mapping each feature to a k-medoids cluster (structural similarity grouping)
- Cluster labels consumed by globaltest for feature group-wise statistical testing
- JSON dashboard payload (dashboard_data.json) carrying clusters, statistics and networking information

## How to apply

First compute a pairwise spectral similarity matrix over all features using a mass-spectral similarity measure such as the modified cosine score implemented in msFeaST (ms2deepscore-compatible); the Pleurotus eryngii olive mill solid waste (0% vs 80%) notebook uses this path. Then run k-medoids clustering on the similarity matrix so the resulting clusters act as a proxy for grouping structurally similar features — the article notes this spectral clustering 'allows for feature group-wise statistical testing using the globaltest package', exploiting globaltest's power to detect small concordant effects with reduced multiplicity adjustment penalties. Choose k so clusters are biologically/spectrally interpretable; the notebook pipeline (msfeast_pipeline_mushroom_type_comparison.ipynb) wires the clusters into subsequent globaltest runs and dashboard export. Validate by checking that cluster members share plausible MS/MS similarity and that downstream globaltest cluster P-values plus per-feature log-2 fold changes land in a comparable range to the article's deposited statistics tables.

## Related tools

- **msFeaST** (Provides the k-medoids spectral clustering step on the modified-cosine similarity matrix within its Python/R pipeline notebooks) — https://github.com/kevinmildau/msFeaST
- **globaltest** (Consumes the k-medoids clusters as feature sets for group-wise statistical testing)

## Examples

```
jupyter-notebook notebooks/msfeast_pipeline_mushroom_type_comparison.ipynb  # run k-medoids spectral clustering + globaltest, writes dashboard_data.json
```

## Evaluation signals

- Cluster members should show high pairwise modified-cosine scores relative to non-members in the similarity matrix.
- Resulting cluster-wise globaltest P-values and per-feature log-2 fold changes should be comparable to the statistics tables from the article's additional deposited run (Section 3).
- The exported dashboard_data.json must load in the msFeaST_Dashboard_bundle.html without errors.
- Pipeline runs end-to-end only when the R configuration check (globaltest 5.56.0, survival 3.5-8, listenv 0.9.1, readr 2.1.5, tibble 3.2.1, dplyr 1.1.4) succeeds.
- Clusters should span a mix of features with elevated and non-significant log-2 fold changes, mirroring the article's observation of many elevated but fewer statistically enriched features.

## Limitations

- Pipeline (pre-processing and clustering) has only been tested on macOS and Linux; Windows support is 'currently being worked on'.
- R path caching issues can break setup if rscript is run before the conda R installation — the README explicitly warns to reopen the terminal after installing the python module.
- globaltest version is pinned via Bioconductor 3.18 (globaltest 5.56.0, R 4.3.3); other versions may not be compatible.
- The illustrative pipeline ties the modified-cosine + k-medoids path to ms2deepscore; swapping similarity measures may need additional integration work.

## Evidence

- [abstract] k-medoids clustering produces structural feature groups: "spectral similarity-based feature groupings generated using k-medoids clustering, where the resulting clusters serve as a proxy for grouping structurally similar features"
- [abstract] clusters enable globaltest feature group-wise testing: "Spectral clustering done in this way allows for feature group-wise statistical testing using the globaltest package"
- [readme] README documents the notebooks and data files: [evidence span withheld — anchor status: external_doc]
- [readme] README specifies the dashboard data location: [evidence span withheld — anchor status: external_doc]
