---
name: distance-distribution-histogram-visualization
description: Use when you have feature vectors (e.g., BiG-SLiCE Euclidean pHMM-domain
  features) plus known group labels (e.g., curated MIBiG compound/function classes)
  and need to check — visually and quantitatively — that within-group distances separate
  from between-group distances.
when_to_use_negative:
- Input is already a clustered result or distance matrix — this skill requires raw
  feature vectors plus group labels to compute the distances
- The feature space is not Euclidean (e.g., BiG-SCAPE pairwise network similarities,
  or BiG-SLiCE v2 cosine-like l2-normalized distances) — the bimodality criterion
  was validated for the v1 Euclidean representation
- No curated reference labels (e.g., MIBiG compound/function classes) exist to define
  within-group vs between-group pairs
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: BiG-SLiCE
  role: Produces the Euclidean feature vectors and GCF models (T=900 run, 29,955 GCFs)
    whose distances are histogrammed; --query mode assigns BGCs to GCFs on a best-hit
    basis (n_ranks=1)
  repo: https://github.com/medema-group/bigslice
- name: scikit-learn
  role: Supplies the Euclidean distance metric used to compute within- vs between-group
    feature-vector distances
- name: matplotlib
  role: Plots the within-group vs between-group distance histograms reproducing Fig.
    3C
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:73e2f435404b749d60cc102a10ca1e047ffdef2b71ab9004c864215279d3be01
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/bigslice
  tools:
  - BiG-SLiCE
  - matplotlib
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# distance-distribution-histogram-visualization

## Summary

Validates that a feature-space representation preserves biological signal by computing pairwise distances within vs between curated groups (here, MIBiG v2.0 reference BGCs) and plotting the two distributions as histograms, as done for BiG-SLiCE's Euclidean BGC feature vectors in Fig. 3C.

## When to use

Use when you have feature vectors (e.g., BiG-SLiCE Euclidean pHMM-domain features) plus known group labels (e.g., curated MIBiG compound/function classes) and need to check — visually and quantitatively — that within-group distances separate from between-group distances.

## When NOT to use

- Input is already a clustered result or distance matrix — this skill requires raw feature vectors plus group labels to compute the distances
- The feature space is not Euclidean (e.g., BiG-SCAPE pairwise network similarities, or BiG-SLiCE v2 cosine-like l2-normalized distances) — the bimodality criterion was validated for the v1 Euclidean representation
- No curated reference labels (e.g., MIBiG compound/function classes) exist to define within-group vs between-group pairs

## Inputs

- BiG-SLiCE T=900 clustering result (SQLite3 output database with BGC and GCF feature vectors, 29,955 GCF models)
- MIBiG v2.0 reference set (1,910 experimentally verified BGC entries with curated compound/function annotations)
- Group labels mapping MIBiG BGCs to curated groups

## Outputs

- Histogram of within-MIBiG-group vs between-group Euclidean distances (bimodal distribution, cf. Fig. 3C)
- Nearest-reference distance distribution for the 29,955 GCF models against their closest MIBiG BGC features

## How to apply

Load the preprocessed BGC/GCF feature vectors from the BiG-SLiCE output SQLite database for the T = 900 run (29,955 GCF models), together with the 1,910 experimentally verified MIBiG v2.0 entries used as the reference set. Group MIBiG BGCs by their curated compound/function annotations, then compute Euclidean distances between feature vectors of within-group pairs and between-group pairs using scikit-learn's Euclidean metric. Separately, for each of the 29,955 GCF models, measure the distance against its closest MIBiG BGC feature vector to obtain a nearest-reference distance distribution (best-hit basis, n_ranks=1). Plot the within-group and between-group distributions as matplotlib histograms and inspect for bimodality: a low-distance within-group mode that separates clearly from the between-group mode confirms the Euclidean feature space delineates the biosynthetic signal.

## Related tools

- **BiG-SLiCE** (Produces the Euclidean feature vectors and GCF models (T=900 run, 29,955 GCFs) whose distances are histogrammed; --query mode assigns BGCs to GCFs on a best-hit basis (n_ranks=1)) — https://github.com/medema-group/bigslice
- **scikit-learn** (Supplies the Euclidean distance metric used to compute within- vs between-group feature-vector distances)
- **matplotlib** (Plots the within-group vs between-group distance histograms reproducing Fig. 3C)

## Examples

```
from sklearn.metrics.pairwise import euclidean_distances; import matplotlib.pyplot as plt; plt.hist([d_within, d_between], bins=50, label=['within','between']); plt.legend()
```

## Evaluation signals

- Histogram shows a bimodal distribution in which within-MIBiG-group distances separate clearly from between-group distances, consistent with Fig. 3C
- Distance computation uses the scikit-learn Euclidean metric on BiG-SLiCE feature vectors (not cosine or other metrics)
- The GCF set analyzed matches the T = 900 run (29,955 GCF models) and the reference set contains all 1,910 MIBiG v2.0 entries
- Nearest-reference distances are computed against each GCF's closest MIBiG BGC feature (best-hit, n_ranks=1), not against all references
- Result is visually cross-checked against the paper's figures/scripts deposited in GigaDB

## Limitations

- The bimodal-separation criterion was demonstrated for BiG-SLiCE v1's Euclidean space at T = 900; other thresholds (T = 300/600/1200/1500) yield different GCF counts and were not the basis of this validation
- BiG-SLiCE v2.0 replaced Euclidean clustering with cosine-like (l2-normalized) distances and updated pHMM models to PFAM 35.0, so distances from a v2 run will not reproduce v1-based distributions
- Fragmented BGCs were excluded from model building (--complete-only; 802,287 of 1,225,071 BGCs used), so the feature vectors reflect complete clusters only

## Evidence

- [results] GCF-vs-MIBiG distance measurement: "we took the 29,955 GCFs calculated from T = 900, measured the distance of every GCF model against their closest MIBiG BGC features"
- [results] Euclidean metric via scikit-learn: "Those centroids were also subjected to an average-linkage agglomerative clustering analysis (sci-kit implementation, Euclidean distance)"
- [results] MIBiG v2.0 reference set: "we incorporated all 1,910 entries from MIBiG v2.0 [78] as a reference set of known and experimentally verified BGCs"
- [other] Core finding supported by the histogram: [evidence span withheld — anchor status: not_found]
- [readme] v2 distance-metric change caveat: "Clustering now uses __cosine-like__ (via l2-normalization) distances"
