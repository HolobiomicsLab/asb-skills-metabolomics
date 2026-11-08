---
name: background-enrichment-threshold-configuration
description: Use when re-running DIMPLE preprocessing on maize-root DESI-MSI replicate
  data and you need to set or sensitivity-test the minimum background-enrichment threshold
  (default 1.5) or the ppm mass tolerance (default 2.5).
when_to_use_negative:
- The data are non-linear imaging datasets (e.g. MALDI or SIMS acquisitions without
  a linear developmental axis), since DIMPLE is designed to leverage the linear developmental
  gradient of the root
- The input is already a processed peak/feature table rather than raw cdf files or
  Matlab workspaces, so threshold re-configuration would double-count peaks
- The goal is metabolite annotation (e.g. METASPACE, Metlin Classic, LOTUS searching)
  rather than tuning the number of m/z signatures retained by preprocessing
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: DIMPLE
  role: Preprocessing pipeline whose configuration exposes the minimum background-enrichment
    threshold (default 1.5) and ppm mass tolerance (default 2.5) that this skill sets
    and sensitivity-tests
  repo: github.com/dickinsonlab/DIMPLE-code
- name: DESI-MSI
  role: Imaging modality that produced the maize-root mass spectrometry data on which
    the threshold sensitivity is evaluated
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:cd88c1738e17222d57d9e21eb32fb480fc3dcf1a88075d7deb8d88a91b7415ed
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# background-enrichment-threshold-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure and sensitivity-test the minimum background-enrichment threshold (default 1.5) in DIMPLE preprocessing of DESI-MSI maize root data, quantifying how relaxing it to 1.1 changes the number of m/z signatures identified per root relative to the default-parameter baseline.

## When to use

Use when re-running DIMPLE preprocessing on maize-root DESI-MSI replicate data and you need to set or sensitivity-test the minimum background-enrichment threshold (default 1.5) or the ppm mass tolerance (default 2.5).

## When NOT to use

- The data are non-linear imaging datasets (e.g. MALDI or SIMS acquisitions without a linear developmental axis), since DIMPLE is designed to leverage the linear developmental gradient of the root
- The input is already a processed peak/feature table rather than raw cdf files or Matlab workspaces, so threshold re-configuration would double-count peaks
- The goal is metabolite annotation (e.g. METASPACE, Metlin Classic, LOTUS searching) rather than tuning the number of m/z signatures retained by preprocessing

## Inputs

- B73 replicate DESI-MSI cdf files from the DIMPLE-code repository (B73-root11, B73-root2-5, B73-root-3-3)
- Corresponding Matlab workspace files (e.g. B73-root11-workspace.mat, B73-root2-5-workspace.mat)
- DIMPLE preprocessing configuration specifying the default parameters (2.5 ppm tolerance, background-enrichment threshold 1.5)

## Outputs

- Peak counts per root for the default and relaxed background-enrichment (1.1) and relaxed ppm (5) settings
- Average peaks per root and percent change relative to default parameters (expected: 199 peaks/root, +3% for background-enrichment 1.1; 223 peaks/root, +14% for ppm 5)
- parameter_sensitivity_metrics.csv containing average peaks per root and percent change for both relaxed settings

## How to apply

Load the deposited B73 replicate DESI-MSI data (cdf files such as B73-root11, B73-root2-5, B73-root-3-3 and their Matlab workspaces) together with the DIMPLE preprocessing configuration from the dickinsonlab/DIMPLE-code repository. Run the DIMPLE preprocessing once at defaults — ppm mass tolerance 2.5 and minimum background-enrichment threshold 1.5 — recording peak counts per root to establish the baseline (~196 m/z peaks per B73 root). Re-run preprocessing with the background-enrichment threshold lowered from 1.5 to 1.1 (and, in a parallel sensitivity arm, with ppm tolerance raised from 2.5 to 5), recording peak counts per root for each setting. Compute the average peaks per root for each relaxed configuration and the percent change versus the default runs; the reported results are 199 peaks per root (+3%) for the relaxed background-enrichment threshold and 223 peaks per root (+14%) for the relaxed ppm tolerance. Accept the configuration if the computed averages reproduce those reported values within a small tolerance, confirming that the threshold choice materially gates how many mass signatures enter downstream gradient-clustering analysis.

## Related tools

- **DIMPLE** (Preprocessing pipeline whose configuration exposes the minimum background-enrichment threshold (default 1.5) and ppm mass tolerance (default 2.5) that this skill sets and sensitivity-tests) — github.com/dickinsonlab/DIMPLE-code
- **DESI-MSI** (Imaging modality that produced the maize-root mass spectrometry data on which the threshold sensitivity is evaluated)

## Evaluation signals

- Computed average peaks per root under background-enrichment threshold 1.1 reproduces the reported 199 peaks per root (+3%) within a small tolerance
- Computed average peaks per root under 5 ppm tolerance reproduces the reported 223 peaks per root (+14%) within a small tolerance
- Default-parameter baseline run reproduces approximately 196 m/z peaks per B73 root
- Peak counts are recorded for each of the three B73 replicates before averaging
- parameter_sensitivity_metrics.csv exists and contains average peaks per root and percent change for both relaxed settings

## Limitations

- Relaxing the background-enrichment threshold from 1.5 to 1.1 changes peak counts by only ~3%, so over-loosening may admit background/noise signatures rather than true developmental enrichment
- Sensitivity was validated on B73 (salt-susceptible) replicates; Oaxacan Green files in the repository (e.g. OG2-P11-0, which lacks a paired workspace file listed) were not used for this metric
- The relaxation trade-off differs by parameter: ppm tolerance relaxation (+14%) is far more permissive than background-enrichment relaxation (+3%), so conclusions about robustness must be reported per parameter

## Evidence

- [readme] cdf files and available Matlab workspaces: "cdf files and available Matlab workspaces are provided for the roots that were analyzed in Sama et al. 2025."
- [abstract] DIMPLE gradient processing: "DIMPLE processes mass signatures along linear gradients, generating clusters of metabolites with specific developmental enrichment patterns in the maize root"
- [intro] MSI data volume: "MSI datasets are large; each pixel contains a full mass spectrum with hundreds to thousands of peaks."
- [intro] DIMPLE definition: "we created a computational method called Developmental Imaging Mass Spectrometry Pipeline for Linear Evaluation (DIMPLE)"
