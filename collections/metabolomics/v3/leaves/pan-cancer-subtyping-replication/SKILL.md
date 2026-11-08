---
name: pan-cancer-subtyping-replication
description: Use when reproducing the Hoadley et al. (2014) multiplatform cancer subtyping
  with klic/coca, or when integrative clustering of five omic layers (copy number,
  methylation, miRNA, mRNA, protein) with per-dataset kernel weights is needed.
when_to_use_negative:
- Input is a single omic layer or a feature table rather than multiple patient-by-patient
  similarity/consensus kernels from several platforms — KLIC requires one kernel per
  dataset
- The goal is de novo subtyping on a different cohort without the Hoadley et al. (2014)
  preprocessing, or with a number of datasets/platforms that does not map onto the
  five-layer Synapse syn2468297 setup
- You need methods other than COCA or KLIC for integrative clustering — this skill
  is specific to replicating the Cabassi and Kirk (2020) pan-cancer analysis
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: klic
  role: Builds per-layer consensus kernels (Monti et al. 2003 consensus clustering)
    and runs Kernel Learning Integrative Clustering to learn per-dataset kernel weights
  repo: https://github.com/acabassi/klic-pancancer-analysis
- name: coca
  role: Performs Cluster-Of-Clusters Analysis on the per-layer patient clusterings
    and supports silhouette-based selection of K
  repo: https://github.com/acabassi/klic-pancancer-analysis
- name: R
  role: Runtime environment; klic and coca are R packages on CRAN and the analysis
    scripts are R scripts
  repo: https://github.com/acabassi/klic-pancancer-analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btaa593
    title: Multiple kernel learning for integrative consensus clustering of omic datasets
  source_content_hash: sha256:bda731c7d611531f8ada75f90a300f46714891bde63519dbf6acc4ea4837ebba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/acabassi/klic-pancancer-analysis
  tools:
  - R
  - coca
  - klic
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# pan-cancer-subtyping-replication

## Summary

Replicates the Cabassi & Kirk (2020) pan-cancer subtyping analysis on the Hoadley et al. (2014) 12-cancer-type multiplatform data, running COCA (with silhouette-based K selection) and KLIC over five per-layer consensus kernels to recover clusters and learned per-platform kernel weights.

## When to use

Use when reproducing the Hoadley et al. (2014) multiplatform cancer subtyping with klic/coca, or when integrative clustering of five omic layers (copy number, methylation, miRNA, mRNA, protein) with per-dataset kernel weights is needed.

## When NOT to use

- Input is a single omic layer or a feature table rather than multiple patient-by-patient similarity/consensus kernels from several platforms — KLIC requires one kernel per dataset
- The goal is de novo subtyping on a different cohort without the Hoadley et al. (2014) preprocessing, or with a number of datasets/platforms that does not map onto the five-layer Synapse syn2468297 setup
- You need methods other than COCA or KLIC for integrative clustering — this skill is specific to replicating the Cabassi and Kirk (2020) pan-cancer analysis

## Inputs

- Hoadley et al. (2014) pan-cancer multiplatform data from Synapse syn2468297 (12 cancer types)
- Preprocessed omic layers: DNA copy number, methylation, mRNA, miRNA, protein (one matrix per platform)
- Per-layer patient clusterings from the Hoadley et al. (2014) clustering algorithm (input to COCA)
- Five consensus similarity kernels built per layer via Monti et al. (2003) consensus clustering (input to KLIC)

## Outputs

- coca_silhouette_table.csv: average silhouette width versus candidate number of clusters K (maximum expected at K=10)
- klic_kernel_weights.csv: average per-dataset kernel weights (copy number 31.4%, methylation 19.2%, miRNA 17.8%, mRNA 16.4%, protein 15.2%)
- Global integrative clustering (KLIC) and COCA cluster assignments of patients

## How to apply

Download the Hoadley et al. (2014) pan-cancer data from Synapse syn2468297 and load the preprocessed layers (DNA copy number, methylation, mRNA, miRNA, protein) using the R scripts from github.com/acabassi/klic-pancancer-analysis, preprocessing each layer the same way as Hoadley et al. (2014). First run integrative clustering with the Cluster-Of-Clusters Analysis (COCA) implementation in the coca R package on the per-layer patient clusterings, computing the average silhouette width over candidate K and selecting the K that maximises it (expected maximum at K=10). Then, for each of the five layers, build one similarity kernel via Consensus Clustering (Monti et al. 2003) as implemented in the klic R package, using the most appropriate clustering algorithm for that layer. Finally, run Kernel Learning Integrative Clustering (KLIC) on the five consensus kernels to obtain the global clustering plus learned per-dataset kernel weights, which should average to copy number 31.4%, methylation 19.2%, miRNA 17.8%, mRNA 16.4% and protein 15.2%.

## Related tools

- **klic** (Builds per-layer consensus kernels (Monti et al. 2003 consensus clustering) and runs Kernel Learning Integrative Clustering to learn per-dataset kernel weights) — https://github.com/acabassi/klic-pancancer-analysis
- **coca** (Performs Cluster-Of-Clusters Analysis on the per-layer patient clusterings and supports silhouette-based selection of K) — https://github.com/acabassi/klic-pancancer-analysis
- **R** (Runtime environment; klic and coca are R packages on CRAN and the analysis scripts are R scripts) — https://github.com/acabassi/klic-pancancer-analysis

## Examples

```
install.packages(c("klic", "coca"))
```

## Evaluation signals

- Average silhouette curve from COCA attains its maximum at K=10
- KLIC average kernel weights match the reported values within rounding tolerance: copy number 31.4%, methylation 19.2%, miRNA 17.8%, mRNA 16.4%, protein 15.2%
- Kernel weight percentages sum to 100% across the five platforms
- The pipeline produces both coca_silhouette_table.csv and klic_kernel_weights.csv with the expected columns (K vs silhouette; platform vs average weight)
- Each of the five layers yields exactly one consensus kernel before KLIC is run

## Limitations

- COCA's properties had never been systematically explored and its robustness to noisy datasets is unclear; KLIC's weighting addresses this by down-weighting noisy datasets
- Each layer must be preprocessed exactly as in Hoadley et al. (2014) and clustered with the same per-layer algorithm as the original study for the comparison to hold
- Data access depends on Synapse (syn2468297) and the exact dataset links are only given at the beginning of the R scripts where needed

## Evidence

- [readme] The code contained in this folder can be used to reproduce the pancancer analysis presented in Cabassi and Kirk (2020).: "The code contained in this folder can be used to reproduce the pancancer analysis presented in Cabassi and Kirk (2020)."
- [other] KLIC assigned the following average weights to the observations in each dataset: copy number 31.4%, methylation 19.2%, miRNA 17.8%, mRNA 16.4% and protein 15.2%.: [evidence span withheld — anchor status: not_found]
- [methods] For each layer, one kernel is built using Consensus Clustering (Monti et al. 2003) and the most appropriate clustering algorithm.: "For each layer, one kernel is built using Consensus Clustering (Monti et al. 2003) and the most appropriate clustering algorithm."
- [abstract] This allows the contribution of noisy datasets to be down-weighted relative to more informative datasets: "This allows the contribution of noisy datasets to be down-weighted relative to more informative datasets"
- [readme] Each layer of the dataset is preprocessed in the same way as in Hoadley et al. (2014): "Each layer of the  dataset is preprocessed in the same way as in Hoadley et al. (2014)."
