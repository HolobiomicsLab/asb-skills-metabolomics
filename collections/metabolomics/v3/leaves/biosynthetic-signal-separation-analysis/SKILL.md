---
name: biosynthetic-signal-separation-analysis
description: Use when you need to validate that a BGC feature-space embedding (e.g.,
  BiG-SLiCE GCF models at T = 900) preserves biosynthetic signal, by testing whether
  within-MIBiG-group distances separate from between-group distances.
when_to_use_negative:
- When the goal is GCF querying or novelty assessment of a new BGC (use BiG-SLiCE
  --query mode instead of distance-distribution validation)
- When using BiG-SLiCE v2.0, whose clustering uses cosine-like (l2-normalized) distances
  rather than the Euclidean metric this validation was built on
- When input BGCs are not yet clustered into GCF models at a defined threshold (T
  = 900 here); the validation requires preprocessed feature vectors from a completed
  run
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_0154
tools:
- name: BiG-SLiCE
  role: Produces the Euclidean feature-space representation of BGCs and the GCF models
    (T = 900) whose distances are analyzed for biosynthetic signal preservation
  repo: https://github.com/medema-group/bigslice
- name: scikit-learn
  role: Provides the Euclidean distance implementation used to compute within- and
    between-group BGC/GCF distances
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:e06f451f76feef3e613707c9fac9a3633b2580aa889be03f14ca5804183e8035
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/bigslice
  tools:
  - BiG-SLiCE
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# biosynthetic-signal-separation-analysis

## Summary

Validates that BiG-SLiCE's Euclidean feature-space representation of BGCs preserves the biosynthetic signal linking genomic similarity to chemical similarity, by showing that distances within curated MIBiG groups separate from distances between groups (bimodal distribution, Fig. 3C).

## When to use

Use when you need to validate that a BGC feature-space embedding (e.g., BiG-SLiCE GCF models at T = 900) preserves biosynthetic signal, by testing whether within-MIBiG-group distances separate from between-group distances.

## When NOT to use

- When the goal is GCF querying or novelty assessment of a new BGC (use BiG-SLiCE --query mode instead of distance-distribution validation)
- When using BiG-SLiCE v2.0, whose clustering uses cosine-like (l2-normalized) distances rather than the Euclidean metric this validation was built on
- When input BGCs are not yet clustered into GCF models at a defined threshold (T = 900 here); the validation requires preprocessed feature vectors from a completed run

## Inputs

- Preprocessed BGC and GCF feature vectors from a BiG-SLiCE clustering run at T = 900 (29,955 GCF models)
- 1,910 experimentally verified BGC entries from MIBiG v2.0 as a reference set of known function
- Curated compound/function annotations used to group MIBiG reference BGCs

## Outputs

- Within-group vs between-group Euclidean distance distributions for MIBiG-curated BGC pairs
- Nearest-reference distance distribution (each GCF model vs its closest MIBiG BGC features)
- Histogram reproducing the bimodal separation reported in Fig. 3C

## How to apply

Load the preprocessed BGC and GCF feature vectors from a BiG-SLiCE run at threshold T = 900 (which produced 29,955 GCF models) together with the 1,910 experimentally verified BGC entries from MIBiG v2.0. Group the MIBiG reference BGCs by their curated compound/function annotations to define within-group pairs, then compute Euclidean distances between feature vectors of BGC pairs within the same group and between groups using the scikit-learn Euclidean metric. For each of the 29,955 GCF models, measure the distance against its closest MIBiG BGC features to obtain the nearest-reference distance distribution. Plot a histogram of within-group versus between-group distances with matplotlib; the rationale is that a bimodal distribution with clear separation demonstrates the Euclidean feature representation delineates the essential biosynthetic signal connecting the genomic space of BGCs to the chemical space of their products.

## Related tools

- **BiG-SLiCE** (Produces the Euclidean feature-space representation of BGCs and the GCF models (T = 900) whose distances are analyzed for biosynthetic signal preservation) — https://github.com/medema-group/bigslice
- **scikit-learn** (Provides the Euclidean distance implementation used to compute within- and between-group BGC/GCF distances)

## Examples

```
from sklearn.metrics.pairwise import euclidean_distances; D_within = euclidean_distances(X_within_mibig); D_between = euclidean_distances(X_gcf, X_mibig)
```

## Evaluation signals

- The within-vs-between distance histogram must be bimodal, with within-MIBiG-group distances separating clearly from between-group distances (Fig. 3C reproduction)
- The run used must be the T = 900 threshold clustering, yielding exactly 29,955 GCF models
- The reference set must comprise all 1,910 MIBiG v2.0 entries labeled as experimentally verified
- Distances must be computed with the Euclidean metric of the scikit-learn implementation referenced in the paper
- Nearest-reference distances must be measured for every GCF model against its closest MIBiG BGC features

## Limitations

- The validation is tied to BiG-SLiCE v1 Euclidean feature space; BiG-SLiCE 2.0 switched to cosine-like (l2-normalization) distances and PFAM 35.0 models, so the exact separation result may not transfer
- Separation depends on the quality of the manually curated MIBiG compound/function groupings; mis-annotated entries would blur the bimodality
- Fragmented BGCs were excluded from model building (--complete-only), so the signal-separation assessment reflects only non-fragmented BGC-based GCF models

## Evidence

- [results] bimodal within-vs-between distance separation: [evidence span withheld — anchor status: not_found]
- [results] GCF vs MIBiG distance measurement: "we took the 29,955 GCFs calculated from T = 900, measured the distance of every GCF model against their closest MIBiG BGC features"
- [results] MIBiG reference set: "we incorporated all 1,910 entries from MIBiG v2.0 [78] as a reference set of known and experimentally verified BGCs"
- [results] scikit-learn Euclidean distance: "Those centroids were also subjected to an average-linkage agglomerative clustering analysis (sci-kit implementation, Euclidean distance)"
- [readme] BiG-SLiCE 2.0 distance metric change: "Clustering now uses __cosine-like__ (via l2-normalization) distances (as in https://www.nature.com/articles/s41564-022-01110-2)"
- [results] fragmented BGC exclusion: "the “–complete-only” parameter was used for the clustering phase, using 802,287 (65%) non-fragmented BGCs from the input data to build the GCF models"
