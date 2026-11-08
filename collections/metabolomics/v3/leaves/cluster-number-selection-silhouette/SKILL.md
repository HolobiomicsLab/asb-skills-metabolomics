---
name: cluster-number-selection-silhouette
description: Use when an integrative clustering (KLIC on consensus kernels, or COCA
  on cluster-of-clusters) has been run over multiple candidate K values and the number
  of clusters must be chosen by maximum average silhouette width.
when_to_use_negative:
- The observation labels for a single fixed K are already given (e.g. the clusterings
  from Hoadley et al. (2014) are used directly as COCA inputs) — silhouette selection
  over K applies only to the integrative step, not to fixed per-layer clusterings.
- Comparing silhouette maxima across datasets to judge which dataset is more informative
  — cluster quality there is assessed with GOTO scores per ontology (biological process,
  molecular function, cellular component), not silhouette.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: klic
  role: Runs Kernel Learning Integrative Clustering over K = 2..20 on the consensus
    kernels, whose output partitions are scored by average silhouette width
  repo: https://www.synapse.org/#!Synapse:syn2468297/wiki/64259
- name: coca
  role: Runs Cluster-Of-Clusters Analysis over K = 2..10 on per-layer clusterings;
    the resulting clusterings are evaluated by silhouette versus K
- name: R
  role: Language in which the silhouette-versus-K computation and the KLIC/COCA pipeline
    scripts are implemented; both packages are on CRAN
  repo: https://www.synapse.org/#!Synapse:syn2468297/wiki/64259
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btaa593
    title: Multiple kernel learning for integrative consensus clustering of omic datasets
  source_content_hash: sha256:d73f19a2e618655835e2eac9f0df23b8cca86dc45a60ddfab76c78ca765964b7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - coca
  - klic
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cluster-number-selection-silhouette

## Summary

Selects the number of clusters K for KLIC and COCA integrative clusterings by computing the average silhouette width over a range of K (e.g. K = 2..20 for KLIC, K = 2..10 for COCA) and choosing the K that maximises it, as in Cabassi and Kirk (2020).

## When to use

Use when an integrative clustering (KLIC on consensus kernels, or COCA on cluster-of-clusters) has been run over multiple candidate K values and the number of clusters must be chosen by maximum average silhouette width.

## When NOT to use

- The observation labels for a single fixed K are already given (e.g. the clusterings from Hoadley et al. (2014) are used directly as COCA inputs) — silhouette selection over K applies only to the integrative step, not to fixed per-layer clusterings.
- Comparing silhouette maxima across datasets to judge which dataset is more informative — cluster quality there is assessed with GOTO scores per ontology (biological process, molecular function, cellular component), not silhouette.

## Inputs

- Partition (cluster labels per observation) for each candidate K from a KLIC run over K = 2..20 on consensus kernels
- Partition for each candidate K from a COCA run over K = 2..10 on per-layer clusterings
- Consensus similarity matrices (one per dataset/layer) used by KLIC

## Outputs

- Silhouette-versus-K table (e.g. coca_silhouette_table.csv) with average silhouette width per K per method
- Selected K for each integrative clustering (expected K = 4 for yeast KLIC, K = 10 for pan-cancer KLIC/COCA)
- Final clustering of observations at the selected K

## How to apply

Run the integrative clustering over a range of candidate cluster numbers: for KLIC use K = 2..20 on the per-dataset consensus kernels built with Consensus Clustering (Monti et al. 2003); for COCA use K = 2..10 on the per-layer patient clusterings. For each K, compute the average silhouette width of the resulting partition and record it in a silhouette-versus-K table (e.g. coca_silhouette_table.csv). Select the K with the maximum silhouette as the final cluster number, then cross-check it against the downstream interpretation (e.g. KLIC kernel weights and GOTO or per-layer cluster quality). The rationale is that silhouette balances cohesion and separation without requiring a likelihood, and the expected maxima differ per method — K = 4 for KLIC in the yeast Granovskaia/Harbison analysis versus K = 10 for COCA — so the table must be produced separately for each method.

## Related tools

- **klic** (Runs Kernel Learning Integrative Clustering over K = 2..20 on the consensus kernels, whose output partitions are scored by average silhouette width) — https://www.synapse.org/#!Synapse:syn2468297/wiki/64259
- **coca** (Runs Cluster-Of-Clusters Analysis over K = 2..10 on per-layer clusterings; the resulting clusterings are evaluated by silhouette versus K)
- **R** (Language in which the silhouette-versus-K computation and the KLIC/COCA pipeline scripts are implemented; both packages are on CRAN) — https://www.synapse.org/#!Synapse:syn2468297/wiki/64259

## Evaluation signals

- The silhouette-versus-K table attains its maximum at K = 4 for the KLIC solution on the yeast Granovskaia/Harbison data.
- COCA selects K = 10 (expected maximum of the silhouette curve), and the pan-cancer silhouette curve attains its maximum at K = 10.
- The output CSV (e.g. coca_silhouette_table.csv) contains one average silhouette width row per candidate K within the scanned range (2..20 for KLIC, 2..10 for COCA).
- The selected-K clustering is consistent with downstream checks, e.g. KLIC kernel weights down-weighting the noisier ChIP-chip kernel relative to the expression kernel in the yeast analysis.

## Limitations

- The expected K differs between methods on the same data (KLIC K = 4 vs COCA K = 10 in the yeast transcriptional module analysis), so silhouette tables must be computed per method and cannot be pooled.
- The candidate K range must cover the optimum (K = 2..20 for KLIC, K = 2..10 for COCA as run in the paper); a truncated range can miss the true maximum.
- Silhouette selection was validated on the Hoadley et al. (2014) pan-cancer data (12 cancer types, five platforms) and the yeast data; behaviour on other omic combinations is not guaranteed by these source cards.

## Evidence

- [methods] Silhouette K range for KLIC: [evidence span withheld — anchor status: not_found]
- [methods] COCA K range and expected selection: [evidence span withheld — anchor status: not_found]
- [methods] Pan-cancer silhouette expectation: [evidence span withheld — anchor status: not_found]
- [methods] Downstream validation against kernel weights: [evidence span withheld — anchor status: not_found]
- [readme] Kernel construction feeding the KLIC step: "For each layer, one kernel is built using Consensus Clustering (Monti et al. 2003) and the most appropriate clustering algorithm."
