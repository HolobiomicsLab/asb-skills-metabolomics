---
name: pairwise-similarity-network-construction
description: Use when you need to cluster a moderate-sized set of annotated BGCs (e.g.
  92 MIBiG v1.3 curated groups) into GCFs by exhaustive pairwise comparison, and need
  non-convex clusters that partitional Euclidean-space methods may miss.
when_to_use_negative:
- 'Input exceeds ~100k BGCs (e.g. genome-wide 1.2M BGC surveys): the O(n^2) pairwise
  comparison is intractable — the article estimates 37,000 hours of runtime on a 36-core
  CPU for BiG-SCAPE; use a near-linear Euclidean-space partitional approach like BiG-SLiCE
  instead.'
- BGCs are already projected into Euclidean feature space and convex (spherical) clusters
  suffice — partitional clustering on centroid features is cheaper and scales better.
- 'BGCs are unannotated or fragmented raw contigs: the network requires reliable Pfam
  domain architectures from processed GenBank files, not raw nucleotide input.'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: BiG-SCAPE
  role: Computes all pairwise BGC distances from domain architecture/sequence similarity,
    builds the weighted similarity network, and runs Affinity Propagation clustering
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
- name: antiSMASH
  role: Upstream predictor that produces the domain-annotated GenBank files consumed
    as network-construction input
  repo: https://antismash.secondarymetabolites.org/
- name: BiG-SLiCE
  role: Non-pairwise alternative/benchmark whose V-score (0.81 at T = 1,100) is compared
    against the network-based baseline (0.91)
  repo: https://github.com/medema-group/bigslice
- name: scikit-learn
  role: Provides Affinity Propagation clustering and the NMI/ARI metrics used for
    validation against curated group labels
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:590aa8d47a5aa630e123a237dbbc78f4d236c4c998e77ba2832c2420033e58b4
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# pairwise-similarity-network-construction

## Summary

Constructs a weighted BGC similarity network by computing all pairwise distances between biosynthetic gene clusters based on their Pfam domain architectures and sequences, enabling clustering algorithms like Affinity Propagation that can find non-convex gene cluster families (GCFs).

## When to use

Use when you need to cluster a moderate-sized set of annotated BGCs (e.g. 92 MIBiG v1.3 curated groups) into GCFs by exhaustive pairwise comparison, and need non-convex clusters that partitional Euclidean-space methods may miss.

## When NOT to use

- Input exceeds ~100k BGCs (e.g. genome-wide 1.2M BGC surveys): the O(n^2) pairwise comparison is intractable — the article estimates 37,000 hours of runtime on a 36-core CPU for BiG-SCAPE; use a near-linear Euclidean-space partitional approach like BiG-SLiCE instead.
- BGCs are already projected into Euclidean feature space and convex (spherical) clusters suffice — partitional clustering on centroid features is cheaper and scales better.
- BGCs are unannotated or fragmented raw contigs: the network requires reliable Pfam domain architectures from processed GenBank files, not raw nucleotide input.

## Inputs

- antiSMASH-processed GenBank files with BGC domain annotations (BiG-SCAPE input folder)
- Curated reference labels for validation (e.g. the 92 manually curated MIBiG v1.3 BGC groups)

## Outputs

- Weighted BGC similarity network (all pairwise distances)
- GCF assignments per BGC from Affinity Propagation
- NMI, ARI, and V-score metrics (e.g. v_score_bigscape.csv)

## How to apply

Assemble antiSMASH-processed GenBank files with BGC domain annotations into an input folder and run BiG-SCAPE to compute the full matrix of pairwise BGC distances based on domain architecture and sequence similarity, building a weighted similarity network. Cluster that network with Affinity Propagation to assign each BGC to a GCF; Affinity Propagation is chosen over partitional clustering because it can find non-convex clusters, which is why BiG-SCAPE reaches V-score = 0.91 on the 92 curated MIBiG v1.3 groups versus BiG-SLiCE's 0.81 at T = 1,100. Validate by comparing predicted GCF assignments against curated group labels with normalized mutual information (NMI) and adjusted Rand index (ARI) via scikit-learn, reporting the V-score as their harmonic mean (e.g. to v_score_bigscale.csv). Be aware the approach has quadratic O(n^2) complexity — the article estimates BiG-SCAPE would need ~37,000 hours on a 36-core CPU for ~1.2M BGCs — so reserve it for small-to-moderate sets or use it as a baseline benchmark.

## Related tools

- **BiG-SCAPE** (Computes all pairwise BGC distances from domain architecture/sequence similarity, builds the weighted similarity network, and runs Affinity Propagation clustering) — https://git.wageningenur.nl/medema-group/BiG-SCAPE
- **antiSMASH** (Upstream predictor that produces the domain-annotated GenBank files consumed as network-construction input) — https://antismash.secondarymetabolites.org/
- **BiG-SLiCE** (Non-pairwise alternative/benchmark whose V-score (0.81 at T = 1,100) is compared against the network-based baseline (0.91)) — https://github.com/medema-group/bigslice
- **scikit-learn** (Provides Affinity Propagation clustering and the NMI/ARI metrics used for validation against curated group labels)

## Examples

```
from sklearn.cluster import AffinityPropagation; labels = AffinityPropagation(affinity='precomputed').fit_predict(S); from sklearn.metrics import normalized_mutual_info_score, adjusted_rand_score
```

## Evaluation signals

- Every BGC pair has a computed distance: the network must be complete (all n^2 comparisons) and weighted, not a k-nearest-neighbor graph.
- Clustering is done on the similarity network with Affinity Propagation (precomputed affinities), not on Euclidean feature vectors.
- External validation: NMI and ARI against the 92 curated MIBiG v1.3 group labels, with V-score = harmonic mean of NMI and ARI.
- Reproduced baseline V-score should match the reported 0.91 for BiG-SCAPE, outperforming BiG-SLiCE's 0.81 at T = 1,100.
- Resulting metrics are written to a CSV (e.g. v_score_bigscape.csv) with all three metrics (NMI, ARI, V-score) present.

## Limitations

- Quadratic O(n^2) time complexity: the article estimates BiG-SCAPE would need an estimated 37,000 hours of runtime on a 36-core CPU for the full ~1.2M BGC dataset, making this approach infeasible at global survey scale.
- The V-score advantage over BiG-SLiCE (0.91 vs 0.81) was demonstrated on only 92 manually curated MIBiG v1.3 groups — a small, curated benchmark not representative of the uncharted BGC space.
- Quality depends on upstream antiSMASH domain annotations; fragmented BGCs or contigs <5,000 bp filtered elsewhere can distort pairwise distances if included.

## Evidence

- [results] BiG-SCAPE achieves a V-score of 0.91 on the 92 manually curated MIBiG v1.3 groups: [evidence span withheld — anchor status: not_found]
- [intro] Pairwise comparison complexity estimate: "even the currently fastest tool (a tool that we previously developed, BiG-SCAPE [22]) will require an estimated 37,000 hours of runtime on a 36-core CPU"
- [intro] Bottleneck of network construction: [evidence span withheld — anchor status: not_found]
- [readme] GCFs from domain-architecture comparative analysis: "by doing a large scale comparative analysis of homologous BGCs sharing similar domain architectures (we call them "Gene Cluster Families (GCFs)")"
