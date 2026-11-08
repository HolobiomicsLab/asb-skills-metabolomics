---
name: binary-indicator-encoding-of-cluster-assignments
description: Use when M datasets each carry a cluster label vector c_m with K_m clusters
  and you must summarise these heterogeneous clusterings into one binary matrix (MOC)
  before integrative consensus clustering such as COCA.
when_to_use_negative:
- Inputs are already kernel matrices or similarity matrices — in that case KLIC (multiple
  kernel learning), not the COCA MOC construction, is the appropriate integration
  route.
- You need a single clustering per dataset (e.g. one label per observation aggregated
  across datasets) rather than a per-dataset, per-cluster binary membership encoding.
- Observations are shared across all datasets with no missingness and identical clusterings,
  making the indicator encoding redundant.
edam_operation: http://edamontology.org/operation_3432
tools:
- name: klic
  role: R package (CRAN) from which per-dataset cluster label vectors can be loaded
    or generated for MOC construction
- name: coca
  role: R package (CRAN) providing Cluster-Of-Clusters Analysis, the downstream integrative
    step that consumes the MOC
- name: R
  role: environment in which the per-layer clusterings and the binary indicator matrix
    are computed and exported
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaa593
    title: Multiple kernel learning for integrative consensus clustering of omic datasets
  source_content_hash: sha256:aafc959c156806e5512e340d594792f78ac971fdc5286ca1592300329fbd4de1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - coca
  - klic
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# binary-indicator-encoding-of-cluster-assignments

## Summary

Constructs the Matrix Of Clusters (MOC) — a K̄×N binary matrix encoding each observation's cluster membership across M independently clustered datasets — as the input summarisation step of Cluster-Of-Clusters Analysis (COCA). It reconciles per-dataset clusterings with potentially different numbers of clusters before the consensus clustering integration step.

## When to use

Use when M datasets each carry a cluster label vector c_m with K_m clusters and you must summarise these heterogeneous clusterings into one binary matrix (MOC) before integrative consensus clustering such as COCA.

## When NOT to use

- Inputs are already kernel matrices or similarity matrices — in that case KLIC (multiple kernel learning), not the COCA MOC construction, is the appropriate integration route.
- You need a single clustering per dataset (e.g. one label per observation aggregated across datasets) rather than a per-dataset, per-cluster binary membership encoding.
- Observations are shared across all datasets with no missingness and identical clusterings, making the indicator encoding redundant.

## Inputs

- per-dataset cluster label vectors c_m (M datasets, K_m clusters each)
- observation count N (union of observations across datasets)
- optional synthetic cluster assignments for validation

## Outputs

- Matrix Of Clusters (MOC): K_bar × N binary matrix, K_bar = sum_m K_m
- moc_matrix.csv — plain CSV, binary (0/1) entries only

## How to apply

Load or generate the per-dataset cluster label vectors c_m for the M datasets (e.g. clusters found per omics layer as in the Hoadley et al. 2014 pan-cancer workflow, available via the klic R package from CRAN). Count K_m per dataset and compute K_bar = sum_m K_m, the total number of rows of the MOC. For each dataset m and each of its K_m clusters, build a length-N binary indicator row with 1 for observations assigned to that cluster and 0 otherwise, entering zeros for observations missing from that dataset. Stack all K_bar indicator rows into a single K_bar×N binary matrix in dataset-then-cluster order, then export it (e.g. moc_matrix.csv) with only binary 0/1 entries.

## Related tools

- **klic** (R package (CRAN) from which per-dataset cluster label vectors can be loaded or generated for MOC construction)
- **coca** (R package (CRAN) providing Cluster-Of-Clusters Analysis, the downstream integrative step that consumes the MOC)
- **R** (environment in which the per-layer clusterings and the binary indicator matrix are computed and exported)

## Examples

```
install.packages("coca"); install.packages("klic"); library(coca); MOC <- matrix(0, nrow = K_bar, ncol = N); write.csv(MOC, "moc_matrix.csv", row.names = FALSE)
```

## Evaluation signals

- moc_matrix.csv exists and is parseable plain CSV
- Matrix has exactly K_bar = sum_m K_m rows and N columns
- All entries are binary (0/1) — no non-binary entries present
- Matrix matches the hand-computed reference MOC for a small synthetic example
- Each cluster indicator row sums to the cluster's size, with zeros entered for observations missing from that dataset

## Limitations

- The MOC only encodes cluster membership, not cluster quality; downstream COCA performance depends on the quality of the per-dataset clusterings.
- Observations missing from a dataset must be encoded as zeros, which conflates 'absent' with 'unassigned to that cluster'.
- The properties of COCA had not been systematically explored, and its robustness to the inclusion of noisy datasets was unclear (motivating the KLIC alternative).

## Evidence

- [abstract] R packages klic and coca are available on the Comprehensive R Archive Network.: "R packages klic and coca are available on the Comprehensive R Archive Network"
- [methods] Integrative clustering is performed using Cluster-Of-Clusters Analysis (Cabassi and Kirk, 2020) and the clusters found in the previous step.: "Integrative clustering is performed using Cluster-Of-Clusters Analysis (Cabassi and Kirk, 2020) and the clusters found in the previous step."
- [readme] A clustering of the patient is found in each layer using the same clustering algorithm as Hoadley et al. (2014).: "A clustering of the patient is found in each layer using the same clustering algorithm as Hoadley et al. (2014)."
- [abstract] The properties of COCA had never been systematically explored and its robustness to noisy datasets was unclear: "the properties of COCA have never been systematically explored, and its robustness to the inclusion of noisy datasets is unclear"
