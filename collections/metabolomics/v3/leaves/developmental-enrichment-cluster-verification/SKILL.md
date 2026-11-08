---
name: developmental-enrichment-cluster-verification
description: Use when DIMPLE linescan clustering has been run on DESI-MSI mass signatures
  and you must confirm assignments match reported B73 cluster contents (e.g., m/z
  149.0118 in Cluster 1, succinate in Cluster 2) before downstream use.
when_to_use_negative:
- Input is a spatially 2D image without a linear developmental axis — DIMPLE's linescan
  projection presumes a linear gradient such as the longitudinal root axis
- The goal is novel metabolite annotation rather than verifying enrichment-pattern
  assignments (use METASPACE, Metlin Classic, or LOTUS instead)
- Data come from a tissue with nonlinear geometry or non-linear gradient, where maximum-projection
  linescans along one axis will mix unrelated regions
edam_operation: http://edamontology.org/operation_0313
tools:
- name: DIMPLE
  role: Pipeline that projects DESI-MSI mass signatures along the linear root axis
    into 1D linescans whose clustered enrichment patterns are being verified
  repo: github.com/dickinsonlab/DIMPLE-code
- name: DESI-MSI
  role: Source imaging modality producing the ~50-100 µm spatial-resolution ion-intensity
    maps of the developing maize root
- name: scikit-learn
  role: AgglomerativeClustering with Ward linkage and Euclidean distance used to cut
    the linescan feature matrix into k=4 clusters
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:fe46e9a17247a44945e25b9703881c6b0043a106097c337398f5c67b65db929f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/dickinsonlab/DIMPLE-code
  tools:
  - DESI-MSI
  - DIMPLE
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0313
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# developmental-enrichment-cluster-verification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Verifies that agglomerative hierarchical clustering (Ward linkage, Euclidean distance, k=4) of DIMPLE 1D longitudinal linescans reproduces the reported B73 maize root developmental enrichment clusters, including meristem-enriched Cluster 1 membership and known-metabolite assignments.

## When to use

Use when DIMPLE linescan clustering has been run on DESI-MSI mass signatures and you must confirm assignments match reported B73 cluster contents (e.g., m/z 149.0118 in Cluster 1, succinate in Cluster 2) before downstream use.

## When NOT to use

- Input is a spatially 2D image without a linear developmental axis — DIMPLE's linescan projection presumes a linear gradient such as the longitudinal root axis
- The goal is novel metabolite annotation rather than verifying enrichment-pattern assignments (use METASPACE, Metlin Classic, or LOTUS instead)
- Data come from a tissue with nonlinear geometry or non-linear gradient, where maximum-projection linescans along one axis will mix unrelated regions

## Inputs

- Preprocessed harmonized DESI-MSI hyperspectral spectra (cdf files and MATLAB .mat workspaces from DIMPLE-code, e.g. B73-root11 and B73-root11-workspace.mat)
- Expected cluster reference table (B73: k=4; Cluster 1 = 48 mass signatures; succinate m/z 117.0195 -> Cluster 2; m/z 149.0118 -> Cluster 1)

## Outputs

- cluster_assignments.csv (mass signature -> cluster assignment table)
- Verification report of matches/mismatches against the published B73 cluster contents

## How to apply

Load the preprocessed harmonized DESI-MSI hyperspectral spectra (NetCDF/cdf files plus the MATLAB workspaces shipped with DIMPLE-code, e.g. B73-root11-workspace.mat) and project each 2D ion-intensity image along the x (developmental) axis by taking the maximum value per position to form 1D linescans. Normalize each linescan to unit sum or TIC-normalized scale and smooth with a 10-pixel running median filter to suppress pixel-level noise before clustering. Assemble the smoothed linescans into a mass-signatures x positions feature matrix and run agglomerative hierarchical clustering with Ward linkage and Euclidean distance, cutting into exactly k=4 clusters as reported for B73. Output the assignment table (cluster_assignments.csv) and check it against the published B73 reference: 48 mass signatures in Cluster 1 (strong meristem/root-tip enrichment), succinate (m/z 117.0195) in Cluster 2, and m/z 149.0118 in Cluster 1, reporting any mismatches.

## Related tools

- **DIMPLE** (Pipeline that projects DESI-MSI mass signatures along the linear root axis into 1D linescans whose clustered enrichment patterns are being verified) — github.com/dickinsonlab/DIMPLE-code
- **DESI-MSI** (Source imaging modality producing the ~50-100 µm spatial-resolution ion-intensity maps of the developing maize root)
- **scikit-learn** (AgglomerativeClustering with Ward linkage and Euclidean distance used to cut the linescan feature matrix into k=4 clusters)

## Examples

```
from sklearn.cluster import AgglomerativeClustering; labels = AgglomerativeClustering(n_clusters=4, linkage='ward', metric='euclidean').fit_predict(linescans)
```

## Evaluation signals

- Number of clusters is exactly 4 after cutting the Ward dendrogram
- Cluster 1 contains 48 mass signatures with strong meristem/root-tip enrichment
- m/z 149.0118 (meristem-enriched) is assigned to Cluster 1
- Succinate (m/z 117.0195) is assigned to Cluster 2 (mismatch expected to be reported if, as in two replicates, it lands elsewhere)
- Assignment table conforms to the cluster_assignments.csv schema with one row per mass signature

## Limitations

- Cluster assignments are replicate-sensitive: succinate was assigned to Cluster 2 in only two of the B73 replicates, so single-replicate runs may diverge from the reference
- Verification is anchored to the B73 (salt-susceptible) reference; Oaxacan Green exhibits different mass signatures and enrichment patterns, so B73 cluster contents are not transferable
- The pipeline depends on the earlier DIMPLE preprocessing stage having produced harmonized hyperspectral spectra; raw vendor formats other than the provided cdf files are not directly consumable

## Evidence
