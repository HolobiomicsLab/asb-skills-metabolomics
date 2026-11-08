---
name: molecular-network-construction-from-similarity-scores
description: Use when you have all-pairs spectral similarity scores (Spec2Vec, cosine,
  or modified cosine) over a positive-ionization MS/MS dataset and want to cluster
  spectra into molecular networks whose edges reflect structural (Tanimoto) similarity.
when_to_use_negative:
- When your training set does not include a large fraction of the fragment ions and
  losses present in the query data — Spec2Vec needs training data and its missing-word
  issue becomes relevant on smaller datasets or with more m/z decimals in peak words
- For GC-MS data — the paper only demonstrated performance on LC-MS data
- When you only need pairwise library matching of a few query spectra rather than
  all-pairs network construction over a large dataset
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0121
tools:
- name: spec2vec
  role: Computes Spec2Vec spectral similarity scores from learnt spectral embeddings
    for network edge weights
  repo: https://github.com/iomega/spec2vec
- name: matchms
  role: Imports, cleans, and processes spectra and computes pairwise similarity measures
    (cosine, modified cosine) between spectra
  repo: https://github.com/matchms/matchms
- name: spec2vec_gnps_data_analysis
  role: Provides the analysis notebooks and plotting functions used to build networks
    and create the well/poorly-clustered comparison figures
  repo: https://github.com/iomega/spec2vec_gnps_data_analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:8aceb06004a811905d24609bfb4acce6d04d964b2245949c61100001bd6110bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/spec2vec
  tools:
  - matchms
  - spec2vec
  - spec2vec_gnps_data_analysis
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# molecular-network-construction-from-similarity-scores

## Summary

Builds molecular networks from all-pairs MS/MS spectral similarity scores (e.g. Spec2Vec vs modified cosine) by retaining top-k links per node, splitting large poorly connected clusters with the Louvain algorithm, and scoring clusters by Tanimoto structural similarity.

## When to use

Use when you have all-pairs spectral similarity scores (Spec2Vec, cosine, or modified cosine) over a positive-ionization MS/MS dataset and want to cluster spectra into molecular networks whose edges reflect structural (Tanimoto) similarity.

## When NOT to use

- When your training set does not include a large fraction of the fragment ions and losses present in the query data — Spec2Vec needs training data and its missing-word issue becomes relevant on smaller datasets or with more m/z decimals in peak words
- For GC-MS data — the paper only demonstrated performance on LC-MS data
- When you only need pairwise library matching of a few query spectra rather than all-pairs network construction over a large dataset

## Inputs

- Processed positive-ionization MS/MS spectra (e.g. AllPositive dataset, 95,320 spectra, Zenodo doi:10.5281/zenodo.3979010), parsed with matchms
- All-pairs spectral similarity matrix (Spec2Vec and/or modified cosine scores)
- Pre-trained Spec2Vec Word2Vec model (Zenodo record 4173596, trained on AllPositive dataset)
- Structural annotations (InChIKeys) to derive Tanimoto molecular fingerprint similarities

## Outputs

- Molecular network with top-10 links per node above score-specific similarity thresholds
- Louvain-split clusters of spectra
- Per-cluster mean Tanimoto structural similarity across edges
- Plot of well-clustered (mean Tanimoto >= 0.5) vs poorly-clustered spectrum fractions per scoring method (fig6_well_poorly_clustered_fractions.png)

## How to apply

Load the processed spectra (e.g. AllPositive dataset, 95,320 positive-ionization spectra from Zenodo 10.5281/zenodo.3979010, parsed with matchms) and compute all-pairs similarity scores — Spec2Vec using a pre-trained Word2Vec model (Zenodo record 4173596, trained on AllPositive), with modified cosine as the baseline. For each score, build a network by retaining the top-10 links per node above the respective similarity thresholds. Apply the Louvain community-detection algorithm to split up large, poorly connected clusters. For each resulting cluster, compute the mean Tanimoto structural similarity across its edges and classify each spectrum as well-clustered (mean Tanimoto >= 0.5) or poorly clustered; plot the fraction of well- versus poorly-clustered spectra per score to compare networking quality. The Spec2Vec all-pairs computation (~82 million pairs) completes in ~300 s on an Intel i7-8550U CPU, so timing serves as a scalability check.

## Related tools

- **spec2vec** (Computes Spec2Vec spectral similarity scores from learnt spectral embeddings for network edge weights) — https://github.com/iomega/spec2vec
- **matchms** (Imports, cleans, and processes spectra and computes pairwise similarity measures (cosine, modified cosine) between spectra) — https://github.com/matchms/matchms
- **spec2vec_gnps_data_analysis** (Provides the analysis notebooks and plotting functions used to build networks and create the well/poorly-clustered comparison figures) — https://github.com/iomega/spec2vec_gnps_data_analysis

## Examples

```
conda install --channel nlesc --channel bioconda --channel conda-forge spec2vec; jupyter notebook  # run spec2vec_gnps_data_analysis notebooks
```

## Evaluation signals

- The Spec2Vec network shows a higher fraction of well-clustered spectra (mean Tanimoto >= 0.5 across cluster edges) than the modified cosine network at comparable settings
- The all-pairs Spec2Vec similarity computation (~82 million pairs) completes on the order of a few minutes (~300 s) on a laptop-class CPU (Intel i7-8550U)
- Each node retains at most 10 links and only edges above the score-specific similarity threshold
- Louvain community detection successfully splits large, poorly connected clusters into smaller communities
- Per-cluster mean Tanimoto values are computed only across spectra with InChIKey-derived structural annotations

## Limitations

- Spec2Vec requires training data, unlike cosine scores, and the training set must include a large fraction of the fragment/loss features present in the query data
- Network analysis may benefit more from a massive training set or retraining the Spec2Vec model including the new experimental spectra
- Missing words become a relevant issue when models are trained on smaller datasets or when more m/z decimals are included in peak words
- Tanimoto on fingerprints was chosen as a practical structural-similarity measure; the best measure of structural similarity lies outside the scope of the study

## Evidence

- [intro] we further apply the Louvain algorithm to split up large, poorly connected clusters [26,27].: "we further apply the Louvain algorithm to split up large, poorly connected clusters [26,27]."
- [results] all-pairs Spec2Vec computation in 300s: [evidence span withheld — anchor status: not_found]
- [abstract] Spec2Vec scalability finding: "Spec2Vec is computationally more scalable allowing structural analogue searches in large databases within seconds."
- [discussion] raw data Zenodo deposition: "the raw data can be now be found on https://doi.org/10.5281/zenodo.3979010"
- [readme] analysis repo with plotting functions: [evidence span withheld — anchor status: external_doc]
