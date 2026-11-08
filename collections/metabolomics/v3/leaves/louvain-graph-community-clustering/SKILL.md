---
name: louvain-graph-community-clustering
description: Use when you have an all-pairs spectral similarity matrix (e.g., Spec2Vec
  or modified cosine over ~95,320 positive-ionization MS/MS spectra) and need to decompose
  the molecular network into well-connected communities for downstream structural
  clustering analysis.
when_to_use_negative:
- Input is raw, unfiltered spectra with no similarity scores computed yet — Louvain
  must be applied to an already-constructed similarity network, not directly to spectra
- Data is GC-MS or outside LC-MS positive-ionization scope — the article only demonstrated
  performance on LC-MS data and notes GC-MS utility is unassessed
- The training set for Spec2Vec embeddings does not cover the fragment/loss features
  in query data — clustering quality depends on the underlying score, which degrades
  with high missing-word fractions
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Spec2Vec
  role: Produces the spectral similarity scores from which the molecular network edges
    are built before Louvain clustering
  repo: https://github.com/iomega/spec2vec
- name: matchms
  role: Imports, cleans, and processes MS/MS spectra (metadata cleaning, peak filtering)
    and computes pairwise similarity matrices used for network construction
  repo: https://github.com/matchms/matchms
- name: Word2Vec
  role: Underlying model (via gensim) trained on spectra-as-documents to learn fragment/loss
    embeddings used for the Spec2Vec score
  repo: https://github.com/iomega/spec2vec
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:ab515f3a3ec065957518b3b56bb94d2aad1251811b25cf6941c61ebe20dafd7f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/spec2vec
  tools:
  - Louvain
  - Spec2Vec
  - Word2Vec
  - matchms
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# louvain-graph-community-clustering

## Summary

Applies Louvain community detection to molecular networks built from spectral similarity scores (e.g., Spec2Vec, modified cosine) to split large, poorly connected clusters into smaller communities, enabling per-cluster structural-similarity assessment via Tanimoto scores.

## When to use

Use when you have an all-pairs spectral similarity matrix (e.g., Spec2Vec or modified cosine over ~95,320 positive-ionization MS/MS spectra) and need to decompose the molecular network into well-connected communities for downstream structural clustering analysis.

## When NOT to use

- Input is raw, unfiltered spectra with no similarity scores computed yet — Louvain must be applied to an already-constructed similarity network, not directly to spectra
- Data is GC-MS or outside LC-MS positive-ionization scope — the article only demonstrated performance on LC-MS data and notes GC-MS utility is unassessed
- The training set for Spec2Vec embeddings does not cover the fragment/loss features in query data — clustering quality depends on the underlying score, which degrades with high missing-word fractions

## Inputs

- All-pairs spectral similarity matrix (Spec2Vec or modified cosine scores) over the AllPositive dataset (95,320 positive-ionization MS/MS spectra)
- Molecular network edges: top-10 links per node above similarity threshold
- Structure annotations (InChIKeys) for spectra, used to compute edge-wise Tanimoto structural similarity

## Outputs

- Cluster/community assignments (partition of spectra into Louvain communities)
- Per-cluster mean Tanimoto structural similarity across edges
- Per-spectrum classification as well-clustered (mean Tanimoto >= 0.5) or poorly clustered
- Plots of well- vs poorly-clustered spectrum fractions per scoring method (e.g., fig6_well_poorly_clustered_fractions.png)

## How to apply

First compute all-pairs spectral similarity scores (Spec2Vec from a pre-trained Word2Vec model, or modified cosine as baseline) and build a molecular network by retaining the top-10 links per node above the respective similarity thresholds. Then apply the Louvain community-detection algorithm to split up large, poorly connected clusters in the network — this is the article's stated rationale, since raw top-k networks can contain sprawling, weakly-linked groups. After clustering, compute the mean Tanimoto structural similarity across each cluster's edges and classify each spectrum as well-clustered (mean Tanimoto >= 0.5) or poorly clustered. The rationale is that Louvain partitions optimize within-community connectivity, which should correspond to groups of structurally related molecules when the underlying spectral score correlates with structural similarity. Evaluate success by comparing the fraction of well-clustered spectra across scoring methods — Spec2Vec is expected to yield a higher fraction of well-clustered spectra than modified cosine.

## Related tools

- **Spec2Vec** (Produces the spectral similarity scores from which the molecular network edges are built before Louvain clustering) — https://github.com/iomega/spec2vec
- **matchms** (Imports, cleans, and processes MS/MS spectra (metadata cleaning, peak filtering) and computes pairwise similarity matrices used for network construction) — https://github.com/matchms/matchms
- **Word2Vec** (Underlying model (via gensim) trained on spectra-as-documents to learn fragment/loss embeddings used for the Spec2Vec score) — https://github.com/iomega/spec2vec

## Evaluation signals

- Spec2Vec curve shows a higher fraction of well-clustered spectra than modified cosine at comparable settings
- Each cluster's mean edge Tanimoto similarity is computable; spectra with mean Tanimoto >= 0.5 are classified as well-clustered
- All-pairs Spec2Vec computation over 95,320 spectra (~82 million pairs) completes in ~300 s on an Intel i7-8550U CPU
- Network construction retains exactly the top-10 links per node above the similarity threshold before Louvain is run
- Louvain partitions contain no spectra belonging to multiple communities (a valid disjoint partition)

## Limitations

- Spec2Vec requires training data, unlike cosine scores, and the training set must include a large fraction of the fragment/loss features present in the query data; network analysis may benefit from a massive training set or retraining including new experimental spectra
- Missing words become a relevant issue when models are trained on smaller datasets or when more decimals from peak m/z are included in words; a missing-fraction threshold (e.g., <0.05) should be set to avoid scoring spectra far outside the learned peaks and losses
- Tanimoto is a pragmatic but not definitive measure of structural similarity; finding the best measure was stated to lie outside the scope of the study

## Evidence

- [intro] Louvain applied to split poorly connected clusters: "we further apply the Louvain algorithm to split up large, poorly connected clusters [26,27]."
- [abstract] Spec2Vec correlates better with structural similarity: "we show how Spec2Vec scores correlate better with structural similarity than cosine-based scores"
- [readme] Analysis notebooks for benchmarking spectral similarity measures: [evidence span withheld — anchor status: external_doc]
