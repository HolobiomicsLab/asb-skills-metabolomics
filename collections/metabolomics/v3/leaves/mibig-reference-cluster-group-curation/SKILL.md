---
name: mibig-reference-cluster-group-curation
description: Use when validating that a BGC feature-space embedding (e.g., BiG-SLiCE
  GCF models at threshold T = 900) separates genomic distances within known compound-class
  groups from distances between groups, using MIBiG v2.0 as ground truth.
when_to_use_negative:
- The goal is de novo GCF discovery or clustering of uncharacterized BGCs — this skill
  is a validation step against curated MIBiG annotations, not a clustering method
- You are running BiG-SLiCE v2.0, which uses cosine-like (l2-normalization) distances
  rather than the Euclidean feature-space distances this validation was designed for
- Input BGCs are fragmented or not in the T = 900 model set — fragmented BGCs were
  mapped via query mode rather than used to build GCF models (--complete-only kept
  802,287 non-fragmented BGCs)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0085
tools:
- name: BiG-SLiCE
  role: Produces the Euclidean feature vectors, GCF models (T = 900 run, 29,955 GCFs),
    and SQLite3 output database that the MIBiG-based distance validation is computed
    on
  repo: https://github.com/medema-group/bigslice
- name: scikit-learn
  role: Provides the Euclidean distance metric used to compute within- and between-group
    pairwise distances between MIBiG BGC and GCF feature vectors
- name: matplotlib
  role: Plots the within-group vs between-group distance histograms reproducing the
    bimodal Fig. 3C separation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:7b20b9921eb8a690c0f87e15bb69973bbe6901f9eed80e92f7a61abc2010251f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/bigslice
  tools:
  - BiG-SLiCE
  - matplotlib
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# mibig-reference-cluster-group-curation

## Summary

Groups the 1,910 experimentally verified BGC entries from MIBiG v2.0 by their curated compound/function annotations to define within-group vs between-group BGC pairs, enabling validation that BiG-SLiCE's Euclidean feature space preserves the biosynthetic signal (bimodal distance distribution, Fig. 3C).

## When to use

Use when validating that a BGC feature-space embedding (e.g., BiG-SLiCE GCF models at threshold T = 900) separates genomic distances within known compound-class groups from distances between groups, using MIBiG v2.0 as ground truth.

## When NOT to use

- The goal is de novo GCF discovery or clustering of uncharacterized BGCs — this skill is a validation step against curated MIBiG annotations, not a clustering method
- You are running BiG-SLiCE v2.0, which uses cosine-like (l2-normalization) distances rather than the Euclidean feature-space distances this validation was designed for
- Input BGCs are fragmented or not in the T = 900 model set — fragmented BGCs were mapped via query mode rather than used to build GCF models (--complete-only kept 802,287 non-fragmented BGCs)

## Inputs

- BiG-SLiCE output SQLite3 database from a T = 900 clustering run (29,955 GCF models), containing preprocessed BGC and GCF feature vectors
- MIBiG v2.0 reference set (1,910 experimentally verified BGC entries with curated compound/function annotations)
- antiSMASH-predicted BGC entries mapped to GCF models via membership values d (n_ranks=1 best-hit basis)

## Outputs

- Within-MIBiG-group Euclidean distance distribution (BGC-to-BGC pairs sharing a curated compound/function annotation)
- Between-group Euclidean distance distribution (pairs from different curated groups / GCF models)
- Nearest-reference distance histogram for all 29,955 GCF models against closest MIBiG BGC features
- Matplotlib histogram reproducing the bimodal within-vs-between separation of Fig. 3C

## How to apply

Load the preprocessed BGC and GCF feature vectors from the BiG-SLiCE run performed at threshold T = 900 (which produced 29,955 GCF models) together with the 1,910 MIBiG v2.0 reference entries; the features live in the run's SQLite3 output database, accessed via SQL queries. Group the MIBiG reference BGCs by their curated compound/function annotations to define within-group pairs, then compute Euclidean distances between feature vectors of BGC pairs within the same group and between BGCs/GCF models from different groups using scikit-learn's Euclidean metric — the same sci-kit Euclidean implementation used elsewhere in the paper for GCF centroid clustering. For each GCF model, also measure the distance against its closest MIBiG BGC features to obtain the nearest-reference distance distribution. Plot the within-group vs between-group histograms with matplotlib and check for bimodal separation. Note: this validation holds for BiG-SLiCE v1's Euclidean representation; BiG-SLiCE v2.0 switched to cosine-like (l2-normalized) distances, which changes the metric and expected distance scales.

## Related tools

- **BiG-SLiCE** (Produces the Euclidean feature vectors, GCF models (T = 900 run, 29,955 GCFs), and SQLite3 output database that the MIBiG-based distance validation is computed on) — https://github.com/medema-group/bigslice
- **scikit-learn** (Provides the Euclidean distance metric used to compute within- and between-group pairwise distances between MIBiG BGC and GCF feature vectors)
- **matplotlib** (Plots the within-group vs between-group distance histograms reproducing the bimodal Fig. 3C separation)

## Examples

```
from sklearn.metrics.pairwise import euclidean_distances; D_within = euclidean_distances(X_within_mibig); D_between = euclidean_distances(X_mibig, X_gcf)
```

## Evaluation signals

- The resulting within-group vs between-group distance histogram is bimodal: within-MIBiG-group distances separate clearly from between-group distances, matching Fig. 3C
- The T = 900 run yields exactly 29,955 GCF models as in the paper, and all 1,910 MIBiG v2.0 entries are present in the feature table
- Distances are computed with scikit-learn's Euclidean metric on the feature vectors extracted from the BiG-SLiCE SQLite database, not recomputed from raw BGC files
- Every GCF model has a defined nearest-MIBiG-BGC distance, so the nearest-reference histogram covers all 29,955 GCFs
- Within-group pair distances should be systematically smaller than between-group distances (separable modes), consistent with the paper's conclusion that Euclidean features delineate the biosynthetic signal

## Limitations

- Validation depends on the quality and coverage of MIBiG v2.0's curated compound/function annotations; BGCs lacking clear curated grouping cannot contribute within-group pairs
- The bimodal separation was demonstrated for the v1 Euclidean feature space at T = 900; BiG-SLiCE v2.0's cosine-like distance metric and updated PFAM 35.0 pHMM database shift the distance distribution and may alter the separation pattern
- Fragmented BGCs were excluded from model building (--complete-only; 802,287 of 1,225,071 BGCs used), so the validation reflects complete BGCs only

## Evidence

- [results] we incorporated all 1,910 entries from MIBiG v2.0 [78] as a reference set of known and experimentally verified BGCs: "we incorporated all 1,910 entries from MIBiG v2.0 [78] as a reference set of known and experimentally verified BGCs"
- [results] we took the 29,955 GCFs calculated from T = 900, measured the distance of every GCF model against their closest MIBiG BGC features: "we took the 29,955 GCFs calculated from T = 900, measured the distance of every GCF model against their closest MIBiG BGC features"
- [results] Those centroids were also subjected to an average-linkage agglomerative clustering analysis (sci-kit implementation, Euclidean distance): "Those centroids were also subjected to an average-linkage agglomerative clustering analysis (sci-kit implementation, Euclidean distance)"
- [readme] Clustering now uses __cosine-like__ (via l2-normalization) distances: "- Clustering now uses __cosine-like__ (via l2-normalization) distances (as in https://www.nature.com/articles/s41564-022-01110-2)"
- [readme] access BiG-SLiCE's preprocessed data, (advanced) users need to be able to [run SQL(ite) queries]: "To access **BiG-SLiCE**'s preprocessed data, (advanced) users need to be able to [run SQL(ite) queries](https://www.sqlitetutorial.net/sqlite-select/)"
