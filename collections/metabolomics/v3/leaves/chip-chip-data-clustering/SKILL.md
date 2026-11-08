---
name: chip-chip-data-clustering
description: Use when integrating a discretised ChIP-chip regulator-gene matrix with
  another omic dataset (e.g. yeast time-course expression) and you need a per-dataset
  kernel for multiple-kernel integrative clustering with KLIC.
when_to_use_negative:
- The input is already a similarity/kernel matrix or you want a single-dataset clustering
  rather than integration of multiple omic layers
- The ChIP-chip data are not discretised and you cannot run BHC-based consensus clustering
  on them
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0749
tools:
- name: klic
  role: R package used to build consensus kernels per dataset (Consensus Clustering
    with BHC base algorithm for ChIP-chip) and to run Kernel Learning Integrative
    Clustering with kernel weights
  repo: https://www.synapse.org/#!Synapse:syn2468297/wiki/64259
- name: coca
  role: R package used to run Cluster-Of-Clusters Analysis as the baseline integrative
    clustering for comparison against KLIC
- name: R
  role: Environment in which the klic and coca packages are run; dataset links are
    listed at the beginning of the R scripts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btaa593
    title: Multiple kernel learning for integrative consensus clustering of omic datasets
  source_content_hash: sha256:173281e8bc417b30da11ade3c0b22fd0d01e2ab4cc15fcca8998903dd2108124
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - coca
  - klic
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# chip-chip-data-clustering

## Summary

Cluster ChIP-chip regulatory data as one kernel layer of an integrative clustering analysis, by building a consensus kernel from the discretised ChIP-chip matrix (Harbison et al. 2004, 117 regulators) using Consensus Clustering (Monti et al. 2003) with BHC as the base algorithm, then letting Kernel Learning Integrative Clustering (KLIC) weight this layer against other datasets such as gene expression.

## When to use

Use when integrating a discretised ChIP-chip regulator-gene matrix with another omic dataset (e.g. yeast time-course expression) and you need a per-dataset kernel for multiple-kernel integrative clustering with KLIC.

## When NOT to use

- The input is already a similarity/kernel matrix or you want a single-dataset clustering rather than integration of multiple omic layers
- The ChIP-chip data are not discretised and you cannot run BHC-based consensus clustering on them

## Inputs

- Discretised ChIP-chip regulator-gene matrix (e.g. Harbison et al. 2004, 117 transcriptional regulators)
- Companion gene expression dataset (e.g. Granovskaia et al. 2010, 551 genes x 41 time points)
- R scripts with dataset links at their beginning

## Outputs

- Consensus kernel for the ChIP-chip layer
- KLIC clustering over K = 2..20 with average silhouette widths per K
- Learned per-dataset kernel weights
- GOTO scores per ontology (biological process, molecular function, cellular component)

## How to apply

Load the discretised ChIP-chip dataset (e.g. the Harbison et al. 2004 dataset with 117 regulators) and build one consensus kernel per dataset using Consensus Clustering (Monti et al. 2003) via the klic R package, choosing BHC as the base clustering algorithm for the ChIP-chip layer (PAM is used for the expression layer). Run KLIC over K = 2..20 on the two kernels and compute the average silhouette width for each K, selecting the K with the maximum silhouette. Because the ChIP-chip layer is often noisier, check the learned kernel weights: a correct analysis should down-weight the ChIP-chip kernel relative to the more informative expression kernel. Validate biology with Gene Ontology Term Overlap (GOTO) scores per ontology (biological process, molecular function, cellular component) comparing KLIC, COCA, and single-dataset clusterings.

## Related tools

- **klic** (R package used to build consensus kernels per dataset (Consensus Clustering with BHC base algorithm for ChIP-chip) and to run Kernel Learning Integrative Clustering with kernel weights) — https://www.synapse.org/#!Synapse:syn2468297/wiki/64259
- **coca** (R package used to run Cluster-Of-Clusters Analysis as the baseline integrative clustering for comparison against KLIC)
- **R** (Environment in which the klic and coca packages are run; dataset links are listed at the beginning of the R scripts)

## Evaluation signals

- The silhouette-vs-K table reaches its maximum at K = 4 for KLIC (expected K = 4)
- COCA, run over K = 2..10, selects K = 10
- The learned KLIC kernel weights down-weight the ChIP-chip kernel relative to the expression kernel
- GOTO scores per ontology reproduce Table 1 of Cabassi and Kirk (2020), showing KLIC's advantage over COCA and single-dataset clusterings
- The ChIP-chip consensus kernel was built with BHC (not PAM) as the base algorithm inside Consensus Clustering

## Limitations

- The contribution of noisy datasets is a known concern: COCA's robustness to the inclusion of noisy datasets was unclear prior to this work, motivating KLIC's learned down-weighting
- The quality of the ChIP-chip layer depends on prior discretisation of the regulator-gene binding data

## Evidence

- [abstract] R packages klic and coca are available on the Comprehensive R Archive Network: "R packages klic and coca are available on the Comprehensive R Archive Network"
- [abstract] This allows the contribution of noisy datasets to be down-weighted relative to more informative datasets: "This allows the contribution of noisy datasets to be down-weighted relative to more informative datasets"
- [readme] For each layer, one kernel is built using Consensus Clustering (Monti et al. 2003) and the most appropriate clustering algorithm.: "For each layer, one kernel is built using Consensus Clustering (Monti et al. 2003) and the most appropriate clustering algorithm."
- [methods] Integrative clustering is performed using Kernel Learning Integrative Clustering (Cabassi and Kirk, 2020) and the kernels built in the previous step.: "Integrative clustering is performed using Kernel Learning Integrative Clustering (Cabassi and Kirk, 2020) and the kernels built in the previous step."
