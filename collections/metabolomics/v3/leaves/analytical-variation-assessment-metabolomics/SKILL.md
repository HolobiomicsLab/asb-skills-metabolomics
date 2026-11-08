---
name: analytical-variation-assessment-metabolomics
description: Use when you have a post-XCMS RAMClustR clustered metabolomics object
  and need to demonstrate that spectral aggregation lowers analytical variation (CV)
  relative to single-feature measurements.
when_to_use_negative:
- Input is raw LC-MS data that has not yet been feature-detected with XCMS and clustered
  with RAMClustR — no SpecAbund or feature clusters exist yet
- The goal is spectral-matching-based annotation (findMain, MSFinder/Sirius export)
  rather than assessing quantitative precision
- The dataset lacks replicate or across-sample intensity variability such that CV
  (sd/mean) is undefined or unstable (e.g., clusters whose component features have
  near-zero means)
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: RAMClustR
  role: Provides the clustered ramclustObj whose SpecAbund matrix (compound-level
    aggregated abundances) is compared against per-feature CVs
  repo: https://github.com/cbroeckl/RAMClustR
- name: XCMS
  role: Upstream feature detection and alignment producing the aligned feature table
    that RAMClustR clusters
- name: R
  role: Environment in which RAMClustR runs and where CV computation and the comparison
    table are scripted
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/ac501530d
    title: 'RAMClust: A Novel Feature Clustering Method Enables Spectral-Matching-Based
      Annotation for Metabolomics Data'
  source_content_hash: sha256:f7227a07b6861ec413add3fcca958c203fd85cdbe21190eadedf6dd271c5fc42
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cbroeckl/RAMClustR
  tools:
  - R
  - RAMClustR
  - XCMS
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# analytical-variation-assessment-metabolomics

## Summary

Quantifies whether clustering single MS features into compound-level spectra (RAMClustR SpecAbund) reduces quantitative analytical variation, by comparing per-cluster SpecAbund CVs against the mean and median CVs of each cluster's component features.

## When to use

Use when you have a post-XCMS RAMClustR clustered metabolomics object and need to demonstrate that spectral aggregation lowers analytical variation (CV) relative to single-feature measurements.

## When NOT to use

- Input is raw LC-MS data that has not yet been feature-detected with XCMS and clustered with RAMClustR — no SpecAbund or feature clusters exist yet
- The goal is spectral-matching-based annotation (findMain, MSFinder/Sirius export) rather than assessing quantitative precision
- The dataset lacks replicate or across-sample intensity variability such that CV (sd/mean) is undefined or unstable (e.g., clusters whose component features have near-zero means)

## Inputs

- RAMClustR clustered object (R ramclustObj) containing the SpecAbund matrix and feature-by-sample intensity matrix
- Feature table from prior XCMS feature detection and alignment (e.g., faahKO NetCDF CDF data processed via XCMS)

## Outputs

- cv_comparison_table.csv with one row per cluster: cluster ID, SpecAbund CV, mean feature CV, median feature CV
- Reproduction of the Figure 3d comparison showing reduced CV at compound-spectrum level

## How to apply

Load the RAMClustR clustered object (e.g., from the post-XCMS clustering of the faahKO dataset), extracting the SpecAbund matrix and the feature-by-sample intensity matrix. For each cluster, compute the coefficient of variation (standard deviation / mean) of the SpecAbund values across all samples. Independently compute the CV of every individual feature across all samples, then aggregate per cluster to obtain the mean and median feature-level CV of each cluster's component features (~120,000 compound measurements in the paper). Assemble a one-row-per-cluster table (cluster ID, SpecAbund CV, mean feature CV, median feature CV) and write it to cv_comparison_table.csv. The rationale is that RAMClustR groups features derived from the same compound because they share retention time and quantitative trends, so summing/aggregating them should average out per-feature noise; validation hinges on SpecAbund CVs being predominantly lower than the corresponding mean/median feature CVs, reproducing the Figure 3d pattern.

## Related tools

- **RAMClustR** (Provides the clustered ramclustObj whose SpecAbund matrix (compound-level aggregated abundances) is compared against per-feature CVs) — https://github.com/cbroeckl/RAMClustR
- **XCMS** (Upstream feature detection and alignment producing the aligned feature table that RAMClustR clusters)
- **R** (Environment in which RAMClustR runs and where CV computation and the comparison table are scripted)

## Examples

```
print(ramclustobj$SpecAbund[,1:6])
```

## Evaluation signals

- For the majority of clusters (~120,000 compound measurements in the paper), the SpecAbund CV is lower than both the mean and median CV of the cluster's component features
- The comparison reproduces the Figure 3d pattern that spectral aggregation reduces quantitative analytical variation versus single-feature measures
- cv_comparison_table.csv has exactly one row per cluster in the ramclustObj, with non-missing CV values (sd/mean computed across all samples)
- SpecAbund matrix dimensions match the number of RAMClustR clusters × number of samples (e.g., inspect via ramclustobj$SpecAbund[,1:6])
- Per-cluster mean and median feature CVs differ from each other, confirming both aggregations were computed over the correct component features

## Limitations

- CV (sd/mean) is unreliable for features or clusters with very low mean intensity; consider excluding near-zero-abundance rows before comparing
- The comparison presupposes a valid prior RAMClustR clustering on post-XCMS features; mis-clustering of features from different compounds would distort the CV comparison
- The finding was demonstrated on the faahKO LC-MS dataset; CV reduction magnitudes may differ on other chromatographic-spectrometric platforms

## Evidence

- [abstract] The approach reduces quantitative analytical variation compared to single-feature measures: "reduces quantitative analytical variation compared to single-feature measures"
- [intro] XCMS as upstream feature generator: "XCMS is a commonly used tool to detect all the signals from a metabolomics dataset, generating aligned features"
- [readme] SpecAbund matrix accessible on the ramclust object: [evidence span withheld — anchor status: external_doc]
- [readme] RAMClustR scope: "A feature clustering algorithm for non-targeted mass spectrometric metabolomics data. This method is compatible with gas and liquid chromatography coupled mass spectrometry"
