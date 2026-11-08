---
name: gene-ontology-term-overlap-scoring
description: Use when evaluating whether an integrative clustering (e.g. KLIC vs COCA)
  better exploits the biological information in multiple omic datasets, by scoring
  cluster coherence through shared Gene Ontology terms across BP, MF and CC ontologies.
when_to_use_negative:
- When cluster members are not genes or lack Gene Ontology annotations, since GOTO
  scores are undefined without GO term overlap
- When only a numeric cluster-quality metric is needed and biological coherence is
  not in question — average silhouette width alone suffices
- When comparing clusterings of samples/patients (e.g. pan-cancer subtyping of Hoadley
  et al. 2014) rather than gene modules, unless GO-based coherence is specifically
  required
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0089
tools:
- name: klic
  role: Runs Kernel Learning Integrative Clustering over the consensus kernels, yielding
    the KLIC clusterings and kernel weights whose biological quality GOTO scores assess
  repo: https://www.synapse.org/#!Synapse:syn2468297/wiki/64259
- name: coca
  role: Runs Cluster-Of-Clusters Analysis on per-dataset clusterings to produce the
    COCA-10 clustering used as the comparison baseline in the GOTO score table
- name: R
  role: Platform in which the klic and coca packages and the GOTO scoring workflow
    are implemented
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btaa593
    title: Multiple kernel learning for integrative consensus clustering of omic datasets
  source_content_hash: sha256:b30c12425f498cef6955512dbd5030576deb40996fc528dacbd02da2bc27e441
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - coca
  - klic
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# gene-ontology-term-overlap-scoring

## Summary

Computes Gene Ontology Term Overlap (GOTO) scores per ontology (biological process, molecular function, cellular component) to assess the biological quality of clusterings produced by integrative clustering methods such as KLIC and COCA, and to compare them against single-dataset clusterings.

## When to use

Use when evaluating whether an integrative clustering (e.g. KLIC vs COCA) better exploits the biological information in multiple omic datasets, by scoring cluster coherence through shared Gene Ontology terms across BP, MF and CC ontologies.

## When NOT to use

- When cluster members are not genes or lack Gene Ontology annotations, since GOTO scores are undefined without GO term overlap
- When only a numeric cluster-quality metric is needed and biological coherence is not in question — average silhouette width alone suffices
- When comparing clusterings of samples/patients (e.g. pan-cancer subtyping of Hoadley et al. 2014) rather than gene modules, unless GO-based coherence is specifically required

## Inputs

- Gene-to-cluster assignments from KLIC (selected K, e.g. KLIC-4) and KLIC-10
- Gene-to-cluster assignments from COCA (selected K = 10)
- Single-dataset clusterings (expression-only and ChIP-chip-only)
- Per-dataset clusterings from the Granovskaia et al. (2010) expression data (551 genes x 41 time points) and Harbison et al. (2004) ChIP-chip data (117 regulators)
- KLIC kernel weights per dataset

## Outputs

- GOTO score table with scores per ontology (biological process, molecular function, cellular component) for each clustering
- Comparison table (reproducing Table 1 of Cabassi and Kirk 2020) showing KLIC down-weighting the noisier ChIP-chip kernel
- Silhouette-vs-K table with maximum at K = 4 for KLIC and K = 10 for COCA

## How to apply

After generating candidate clusterings — KLIC over K = 2..20 on per-dataset consensus kernels (PAM base for expression, BHC for ChIP-chip) and COCA over K = 2..10 on per-dataset clusterings — compute GOTO scores for the KLIC-4, KLIC-10, COCA-10 and single-dataset clusterings separately for each of the three ontologies: biological process, molecular function, and cellular component. Use the klic and coca R packages to run the clustering steps and obtain cluster assignments, and verify kernel weights: KLIC's multiple kernel learning should down-weight the noisier ChIP-chip kernel relative to the expression kernel. Assemble the per-ontology GOTO scores into a table (analogous to Table 1 of Cabassi and Kirk 2020) and compare across methods; higher GOTO scores indicate clusterings whose members share more GO terms. Decision points: select K by maximum average silhouette width (expected K = 4 for KLIC on the yeast data, K = 10 for COCA) before scoring.

## Related tools

- **klic** (Runs Kernel Learning Integrative Clustering over the consensus kernels, yielding the KLIC clusterings and kernel weights whose biological quality GOTO scores assess) — https://www.synapse.org/#!Synapse:syn2468297/wiki/64259
- **coca** (Runs Cluster-Of-Clusters Analysis on per-dataset clusterings to produce the COCA-10 clustering used as the comparison baseline in the GOTO score table)
- **R** (Platform in which the klic and coca packages and the GOTO scoring workflow are implemented)

## Evaluation signals

- The silhouette-vs-K table shows its maximum at K = 4 for KLIC on the yeast expression + ChIP-chip kernels
- COCA selects K = 10 when run over K = 2..10 on the per-dataset clusterings
- The GOTO score table shows KLIC down-weighting the noisier ChIP-chip kernel, reproducing Table 1 of Cabassi and Kirk (2020)
- GOTO scores are reported separately for each of the three ontologies (BP, MF, CC) and for all four clusterings (KLIC-4, KLIC-10, COCA-10, single-dataset)
- KLIC kernel weights confirm the ChIP-chip kernel weight is lower than the expression kernel weight

## Limitations

- COCA's properties had never been systematically explored and its robustness to the inclusion of noisy datasets is unclear, so COCA-based clusterings may be more sensitive to a noisy layer
- GOTO scores depend on Gene Ontology annotation coverage, which may bias comparisons between clusterings over different gene subsets
- The expected K selection (KLIC K = 4, COCA K = 10) is dataset-specific and must be re-derived via silhouette width for new data

## Evidence

- [abstract] KLIC's weighting allows noisy datasets to be down-weighted: "This allows the contribution of noisy datasets to be down-weighted relative to more informative datasets"
- [methods] COCA step producing comparable clusterings: "Integrative clustering is performed using Cluster-Of-Clusters Analysis (Cabassi and Kirk, 2020) and the clusters found in the previous step."
- [other] GOTO scoring per ontology over all clusterings: [evidence span withheld — anchor status: not_found]
- [other] Validation criteria for the full workflow: [evidence span withheld — anchor status: not_found]
- [abstract] Tool availability for the workflow: "R packages klic and coca are available on the Comprehensive R Archive Network"
