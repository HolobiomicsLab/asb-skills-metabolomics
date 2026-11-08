---
name: affinity-propagation-gene-cluster-family-clustering
description: Use when benchmarking BGC clustering quality against manually curated
  MIBiG group labels (e.g., reproducing a V-score baseline), or when non-convex GCF
  structure is expected and pairwise network-based clustering is affordable for the
  dataset size.
when_to_use_negative:
- 'Datasets of hundreds of thousands to millions of BGCs (e.g., the 1,225,071-BGC
  RefSeq-scale run): pairwise comparison is O(n^2) and BiG-SCAPE was estimated at
  ~37,000 hours on a 36-core CPU — use BiG-SLiCE''s Euclidean-space near-linear clustering
  instead'
- Input BGCs lack antiSMASH-style GenBank/domain annotations and no converter step
  is possible, since BiG-SCAPE distances depend on domain architecture and sequence
  similarity
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: BiG-SCAPE
  role: Computes all pairwise BGC distances from domain architecture/sequence similarity,
    builds the weighted similarity network, and clusters it with Affinity Propagation
    to produce GCFs
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
- name: antiSMASH
  role: Preprocesses genomes into GenBank files with annotated BGCs and domain annotations
    that form the BiG-SCAPE input folder
  repo: https://antismash.secondarymetabolites.org/
- name: BiG-SLiCE
  role: Scalable partitional-clustering comparator whose Euclidean-space GCF result
    (V-score 0.81 at T = 1,100) is benchmarked against the Affinity Propagation baseline
  repo: https://github.com/medema-group/bigslice
- name: scikit-learn
  role: Provides the NMI and ARI metrics used to score predicted GCF assignments against
    curated MIBiG groups
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:5cdb42f6c262687d1c494982ff6d70c65b4bd0538bab67f29c0267d3048c5bd0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SCAPE
  - BiG-SLiCE
  - antiSMASH
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# affinity-propagation-gene-cluster-family-clustering

## Summary

Clusters biosynthetic gene clusters (BGCs) into gene cluster families (GCFs) by building a weighted pairwise similarity network with BiG-SCAPE and applying Affinity Propagation, which can recover non-convex cluster shapes that Euclidean-space partitional methods like BiG-SLiCE (T = 1,100, V-score = 0.81) miss. Serves as the pairwise-network baseline whose V-score of 0.91 on the 92 curated MIBiG v1.3 groups benchmarks scalable clustering approaches.

## When to use

Use when benchmarking BGC clustering quality against manually curated MIBiG group labels (e.g., reproducing a V-score baseline), or when non-convex GCF structure is expected and pairwise network-based clustering is affordable for the dataset size.

## When NOT to use

- Datasets of hundreds of thousands to millions of BGCs (e.g., the 1,225,071-BGC RefSeq-scale run): pairwise comparison is O(n^2) and BiG-SCAPE was estimated at ~37,000 hours on a 36-core CPU — use BiG-SLiCE's Euclidean-space near-linear clustering instead
- Input BGCs lack antiSMASH-style GenBank/domain annotations and no converter step is possible, since BiG-SCAPE distances depend on domain architecture and sequence similarity

## Inputs

- antiSMASH-processed GenBank files with BGC domain annotations (BiG-SCAPE input folder structure)
- Manually curated MIBiG v1.3 group labels for the 92 curated groups (ground-truth cluster assignments)
- BiG-SCAPE clustering output (GCF assignments per BGC)

## Outputs

- Gene cluster family (GCF) assignment per BGC from the Affinity Propagation clustering of the BiG-SCAPE similarity network
- v_score_bigslice.csv containing NMI, ARI, and their harmonic mean (V-score) relative to curated labels

## How to apply

Assemble antiSMASH-processed GenBank files (with BGC domain annotations) for the BGCs of interest into a BiG-SCAPE input folder, following the BiG-SLiCE/BiG-SCAPE input-folder preparation convention. Run BiG-SCAPE to compute all pairwise BGC distances based on domain architecture and sequence similarity and build the weighted similarity network — note this is O(n^2): on ~1.2M BGCs the authors estimated BiG-SCAPE would need ~37,000 hours on a 36-core CPU, so restrict this skill to curated/reference-scale sets (e.g., the 1,910-entry MIBiG v2.0 reference or the 92 curated MIBiG v1.3 groups). Cluster the network with Affinity Propagation to assign each BGC to a GCF. Evaluate against curated group labels using scikit-learn's normalized mutual information (NMI) and adjusted rand index (ARI), then report the V-score as the harmonic mean of NMI and ARI (expected baseline: 0.91 for BiG-SCAPE vs 0.81 for BiG-SLiCE at T = 1,100).

## Related tools

- **BiG-SCAPE** (Computes all pairwise BGC distances from domain architecture/sequence similarity, builds the weighted similarity network, and clusters it with Affinity Propagation to produce GCFs) — https://git.wageningenur.nl/medema-group/BiG-SCAPE
- **antiSMASH** (Preprocesses genomes into GenBank files with annotated BGCs and domain annotations that form the BiG-SCAPE input folder) — https://antismash.secondarymetabolites.org/
- **BiG-SLiCE** (Scalable partitional-clustering comparator whose Euclidean-space GCF result (V-score 0.81 at T = 1,100) is benchmarked against the Affinity Propagation baseline) — https://github.com/medema-group/bigslice
- **scikit-learn** (Provides the NMI and ARI metrics used to score predicted GCF assignments against curated MIBiG groups)

## Examples

```
from sklearn.metrics import normalized_mutual_info_score, adjusted_rand_score; import numpy as np; nmi = normalized_mutual_info_score(y_true, y_pred); ari = adjusted_rand_score(y_true, y_pred); v_score = 2*nmi*ari/(nmi+ari)
```

## Evaluation signals

- Reproduced BiG-SCAPE V-score must match the reported value of 0.91 against the 92 MIBiG v1.3 curated groups
- V-score computed as the harmonic mean of NMI and ARI must be ≥ both individual metrics; compare directly to BiG-SLiCE's 0.81 at T = 1,100 to confirm the non-convex-cluster advantage
- All three metrics (NMI, ARI, V-score) written to v_score_bigslice.csv with one row per evaluation
- Every input BGC in the antiSMASH GenBank set receives exactly one GCF assignment from the Affinity Propagation step (no unassigned singletons unless BiG-SCAPE reports them explicitly)

## Limitations

- Quadratic O(n^2) pairwise comparison makes the method infeasible at scale: BiG-SCAPE was estimated to require ~37,000 hours of runtime on a 36-core CPU for the full ~1.2M-BGC dataset
- BiG-SCAPE clustering quality depends on complete antiSMASH domain annotations; fragmented BGCs and contigs <5,000 bp filtered during antiSMASH processing (--minlength 5000) affect the input set
- Performance claims are anchored to the MIBiG v1.3 curated set (92 groups); results on uncurated or heavily fragmented inputs are not validated by this benchmark

## Evidence

- [intro] pairwise comparison is the bottleneck at scale: "even the currently fastest tool (a tool that we previously developed, BiG-SCAPE [22]) will require an estimated 37,000 hours of runtime on a 36-core CPU"
- [methods] input folder preparation convention: "our previous tool, [BiG-SCAPE](https://git.wageningenur.nl/medema-group/BiG-SCAPE)"
- [results] scikit-learn clustering/metric implementations used in this study: "Those centroids were also subjected to an average-linkage agglomerative clustering analysis (sci-kit implementation, Euclidean distance)"
- [results] BiG-SCAPE comparison steps assume antiSMASH-preprocessed GenBank input: "We then predicted their BGCs using antiSMASH v5.1.1, filtering out contigs <5,000 bp (“–minlength 5000”)"
