---
name: clustering-agreement-evaluation-nmi-ari
description: Use when a BGC clustering result (GCF assignments from BiG-SCAPE, BiG-SLiCE,
  or similar) must be scored against curated MIBiG group labels to compare clustering
  quality across tools or thresholds.
when_to_use_negative:
- You have no curated reference labels (e.g., clustering 1.2M BGCs from RefSeq genomes
  with no ground truth) — NMI/ARI require a gold-standard partition.
- You want an internal clustering quality metric (e.g., silhouette on Euclidean-projected
  BGC feature space) rather than agreement with an external curated partition.
- Comparing label sets with unmatched BGC identifiers or different BGC sets — the
  two partitions must cover the same BGCs.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: BiG-SCAPE
  role: Source of the pairwise-network + Affinity Propagation GCF assignments evaluated
    against curated MIBiG groups (baseline V-score 0.91)
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
- name: BiG-SLiCE
  role: Source of Euclidean-space partitional GCF assignments benchmarked by this
    skill (V-score 0.81 at T = 1,100); predictions can also be read from its SQLite
    output
  repo: https://github.com/medema-group/bigslice
- name: scikit-learn
  role: Computes NMI and ARI between predicted and curated partitions; V-score taken
    as their harmonic mean
- name: antiSMASH
  role: Produces the antiSMASH-processed GenBank files and BGC domain annotations
    underlying the clustered BGCs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:faf1e127f8912a3dd24edd9d7d12a8483e2f006967085f99ee3c7ddc888783d9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SCAPE
  - BiG-SLiCE
  - antiSMASH
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# clustering-agreement-evaluation-nmi-ari

## Summary

Quantifies agreement between predicted BGC gene cluster family (GCF) assignments and manually curated MIBiG group labels using normalized mutual information (NMI) and adjusted Rand index (ARI) computed with scikit-learn, combined into a V-score (harmonic mean of NMI and ARI) to benchmark clustering tools such as BiG-SCAPE and BiG-SLiCE.

## When to use

Use when a BGC clustering result (GCF assignments from BiG-SCAPE, BiG-SLiCE, or similar) must be scored against curated MIBiG group labels to compare clustering quality across tools or thresholds.

## When NOT to use

- You have no curated reference labels (e.g., clustering 1.2M BGCs from RefSeq genomes with no ground truth) — NMI/ARI require a gold-standard partition.
- You want an internal clustering quality metric (e.g., silhouette on Euclidean-projected BGC feature space) rather than agreement with an external curated partition.
- Comparing label sets with unmatched BGC identifiers or different BGC sets — the two partitions must cover the same BGCs.

## Inputs

- Predicted cluster assignment table (BGC ID -> GCF label) from BiG-SCAPE Affinity Propagation or BiG-SLiCE (SQLite output / --query membership values)
- Curated MIBiG v1.3 group labels (BGC ID -> curated group, 92 groups)
- Matched list of BGC identifiers present in both partitions

## Outputs

- CSV of metrics (e.g., v_score_bigscape.csv) with NMI, ARI, and V-score per clustering run
- Comparative V-score values for tool/threshold benchmarking (e.g., 0.91 BiG-SCAPE vs 0.81 BiG-SLiCE at T = 1,100)

## How to apply

Collect the predicted cluster label for every BGC (e.g., BiG-SCAPE Affinity Propagation GCF assignments or BiG-SLiCE GCF memberships at a chosen threshold T) and the corresponding curated group label for the 92 MIBiG v1.3 groups. Compute normalized mutual information (NMI) and adjusted Rand index (ARI) with scikit-learn, then take the V-score as the harmonic mean of NMI and ARI. Write NMI, ARI, and V-score to a results file (e.g., v_score_bigscape.csv) so multiple tools/thresholds (e.g., BiG-SCAPE vs BiG-SLiCE at T = 1,100) can be compared. Validation: the reproduced BiG-SCAPE baseline must reach a V-score of 0.91 against the 92 curated groups, versus 0.81 for BiG-SLiCE at T = 1,100; BiG-SCAPE's higher score reflects its Affinity Propagation algorithm finding non-convex clusters.

## Related tools

- **BiG-SCAPE** (Source of the pairwise-network + Affinity Propagation GCF assignments evaluated against curated MIBiG groups (baseline V-score 0.91)) — https://git.wageningenur.nl/medema-group/BiG-SCAPE
- **BiG-SLiCE** (Source of Euclidean-space partitional GCF assignments benchmarked by this skill (V-score 0.81 at T = 1,100); predictions can also be read from its SQLite output) — https://github.com/medema-group/bigslice
- **scikit-learn** (Computes NMI and ARI between predicted and curated partitions; V-score taken as their harmonic mean)
- **antiSMASH** (Produces the antiSMASH-processed GenBank files and BGC domain annotations underlying the clustered BGCs)

## Examples

```
from sklearn.metrics import normalized_mutual_info_score, adjusted_rand_score; nmi = normalized_mutual_info_score(curated_labels, predicted_gcf); ari = adjusted_rand_score(curated_labels, predicted_gcf); v_score = 2*nmi*ari/(nmi+ari)
```

## Evaluation signals

- Reproduced BiG-SCAPE baseline V-score equals the reported 0.91 on the 92 MIBiG v1.3 curated groups
- BiG-SLiCE benchmark at T = 1,100 yields V-score 0.81, i.e., below the BiG-SCAPE baseline
- NMI and ARI are both in [0, 1] and the V-score equals 2*NMI*ARI/(NMI+ARI)
- Every BGC in the evaluation has exactly one predicted and one curated label; the two label vectors have equal length
- All three metrics (NMI, ARI, V-score) are written to the output CSV (e.g., v_score_bigscape.csv)

## Limitations

- The reference is only the 92 manually curated MIBiG v1.3 groups, so scores reflect agreement with a small curated subset, not with all known BGC diversity.
- BiG-SLiCE's Euclidean partitional clustering (optimal near T = 900-1,200 thresholds) scores lower than BiG-SCAPE's pairwise Affinity Propagation, which can find non-convex clusters; the metric penalizes convex-cluster assumptions.
- NMI and ARI behave differently for imbalanced cluster-size distributions; reporting the harmonic mean alone can mask divergence between the two indices.

## Evidence

- [other] BiG-SCAPE achieves a V-score of 0.91 on the 92 manually curated MIBiG v1.3 groups, outperforming BiG-SLiCE (V-score = 0.81 at T = 1,100): [evidence span withheld — anchor status: not_found]
- [other] Compare predicted vs curated labels with NMI and ARI: [evidence span withheld — anchor status: not_found]
- [abstract] BiG-SLiCE clusters BGCs into GCFs in a non-pairwise, near-linear fashion by representing them in Euclidean space: [evidence span withheld — anchor status: not_found]
- [results] its Affinity Propagation-based algorithm can find non-convex clusters: [evidence span withheld — anchor status: not_found]
- [results] Five different threshold values (T = {300, 600, 900, 1,200, 1,500}) were tested, producing a decreasing number of GCF models: "Five different threshold values (T = {300, 600, 900, 1,200, 1,500}) were tested, producing a decreasing number of GCF models"
