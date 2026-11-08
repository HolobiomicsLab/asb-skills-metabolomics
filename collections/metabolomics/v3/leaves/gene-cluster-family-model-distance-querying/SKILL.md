---
name: gene-cluster-family-model-distance-querying
description: Use when pre-computed BiG-SLiCE GCF models exist (e.g., the T=900 run
  yielding 29,955 GCFs) and you need to quantify how close each model is to curated
  MIBiG v2.0 reference BGCs, or to validate feature-space separation.
when_to_use_negative:
- Input is raw genome/contig data not yet processed by antiSMASH — no BGC feature
  vectors exist yet to query against
- You need gene-level pairwise BGC comparisons rather than Euclidean feature-space
  distances — the article positions BiG-SCAPE as the pairwise network-comparison alternative
- Working with BiG-SLiCE v2.0 outputs, which switched to cosine-like (l2-normalization)
  distances, so Euclidean-distance reproductions of the paper's Fig. 3C are no longer
  applicable
edam_operation: http://edamontology.org/operation_0291
tools:
- name: BiG-SLiCE
  role: Produces the GCF models and their Euclidean feature vectors; its --query mode
    places BGCs against pre-computed GCF models and its SQLite3 output stores the
    preprocessed features used for distance computation
  repo: https://github.com/medema-group/bigslice
- name: scikit-learn
  role: Supplies the Euclidean distance metric (scikit-learn implementation) used
    to compute BGC/GCF feature-vector distances and the average-linkage agglomerative
    clustering of GCF centroids
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:0aacac03d8ba67af3729cd6a6285c70cf738264e731c17cfb7119c7078602b66
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/bigslice
  tools:
  - BiG-SLiCE
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0291
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# gene-cluster-family-model-distance-querying

## Summary

Measuring Euclidean distances between BiG-SLiCE GCF model feature vectors and MIBiG reference BGC feature vectors to test whether the Euclidean feature space preserves the biosynthetic signal connecting BGC genomic similarity to chemical similarity.

## When to use

Use when pre-computed BiG-SLiCE GCF models exist (e.g., the T=900 run yielding 29,955 GCFs) and you need to quantify how close each model is to curated MIBiG v2.0 reference BGCs, or to validate feature-space separation.

## When NOT to use

- Input is raw genome/contig data not yet processed by antiSMASH — no BGC feature vectors exist yet to query against
- You need gene-level pairwise BGC comparisons rather than Euclidean feature-space distances — the article positions BiG-SCAPE as the pairwise network-comparison alternative
- Working with BiG-SLiCE v2.0 outputs, which switched to cosine-like (l2-normalization) distances, so Euclidean-distance reproductions of the paper's Fig. 3C are no longer applicable

## Inputs

- BiG-SLiCE run output at threshold T = 900 (29,955 GCF models), stored as the SQLite3 database in the output folder
- Preprocessed BGC and GCF feature vectors (GCF centroid features in Euclidean space)
- MIBiG v2.0 reference set (1,910 experimentally verified BGC entries with curated compound/function annotations)

## Outputs

- Within-group vs between-group Euclidean distance distributions for MIBiG-curated BGC pairs
- Nearest-reference distance distribution (distance of every GCF model to its closest MIBiG BGC features)
- Histogram reproducing the bimodal separation of Fig. 3C

## How to apply

Load the preprocessed BGC and GCF feature vectors from a BiG-SLiCE run performed at threshold T = 900 (which produced 29,955 GCF models) together with the 1,910 experimentally verified BGC entries from MIBiG v2.0. Retrieve the feature vectors from the run's SQLite3 output database, which BiG-SLiCE populates with processed input data. Group the MIBiG reference BGCs by curated compound/function annotations to define within-group pairs, then compute Euclidean distances between feature vectors of pairs within the same MIBiG group and between BGCs/GCF models from different groups using scikit-learn's Euclidean metric. For each GCF model, measure the distance against its closest MIBiG BGC features to obtain the nearest-reference distance distribution. Plot within-group versus between-group distance histograms (e.g., with matplotlib) and check for the bimodal separation reported as Fig. 3C, which demonstrates that the Euclidean feature representation delineates the essential biosynthetic signal.

## Related tools

- **BiG-SLiCE** (Produces the GCF models and their Euclidean feature vectors; its --query mode places BGCs against pre-computed GCF models and its SQLite3 output stores the preprocessed features used for distance computation) — https://github.com/medema-group/bigslice
- **scikit-learn** (Supplies the Euclidean distance metric (scikit-learn implementation) used to compute BGC/GCF feature-vector distances and the average-linkage agglomerative clustering of GCF centroids)

## Examples

```
from sklearn.metrics import pairwise_distances; D = pairwise_distances(gcf_features, mibig_features, metric='euclidean')
```

## Evaluation signals

- The resulting histogram shows a bimodal distance distribution in which within-MIBiG-group distances separate clearly from between-group distances, consistent with Fig. 3C
- The run at T = 900 yields exactly 29,955 GCF models against which distances are measured
- The reference set comprises 1,910 MIBiG v2.0 entries; all GCF-to-reference distances are non-negative Euclidean values
- GCF models mapping close to MIBiG references should correspond to known compound classes, matching the paper's conclusion that Euclidean features preserve genomic-to-chemical signal

## Limitations

- The paper's Euclidean-distance results correspond to BiG-SLiCE v1; BiG-SLiCE 2.0 changed clustering to cosine-like (via l2-normalization) distances and updated pHMM databases to PFAM 35.0, so distances are not directly comparable across versions
- GCF models were built only from non-fragmented BGCs (--complete-only; 802,287 of 1,225,071 BGCs, 65%); fragmented BGCs were mapped onto GCFs afterward via membership assignment with n_ranks=1
- Querying requires pre-computed GCF models; the pre-processed result on ~1.2M microbial BGCs is a 17GB zipped download and there is currently no pre-processed result for BiG-SLiCE v2

## Evidence

- [results] we took the 29,955 GCFs calculated from T = 900, measured the distance of every GCF model against their closest MIBiG BGC features: "we took the 29,955 GCFs calculated from T = 900, measured the distance of every GCF model against their closest MIBiG BGC features"
- [results] Those centroids were also subjected to an average-linkage agglomerative clustering analysis (sci-kit implementation, Euclidean distance): "Those centroids were also subjected to an average-linkage agglomerative clustering analysis (sci-kit implementation, Euclidean distance)"
- [readme] Using the `--query` mode, you can perform a blazing-fast query of a putative BGC against the pre-processed set of Gene Cluster Family (GCF) models: "Using the `--query` mode, you can perform a blazing-fast query of a putative BGC against the pre-processed set of Gene Cluster Family (GCF) models"
