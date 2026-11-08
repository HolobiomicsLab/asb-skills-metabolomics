---
name: parameter-sensitivity-analysis
description: Use when a published pipeline result (e.g., DIMPLE peak counts per B73
  root) must be stress-tested by re-running preprocessing at altered thresholds to
  report percent changes relative to the default-parameter baseline.
when_to_use_negative:
- The goal is de novo annotation or identification of mass signatures (use Metlin
  Classic, LOTUS, or METASPACE instead of re-tuning DIMPLE thresholds).
- The data are not from a linear developmental gradient (e.g., MALDI or SIMS imaging
  of non-linear tissue), since DIMPLE is designed to leverage the linear root developmental
  axis.
tools:
- name: DIMPLE
  role: Pipeline whose preprocessing is re-run at relaxed parameters (5 ppm tolerance;
    1.1 background-enrichment threshold) to measure sensitivity of per-root peak counts
  repo: github.com/dickinsonlab/DIMPLE-code
- name: DESI-MSI
  role: Imaging modality that produced the maize-root mass spectrometry data whose
    peak counts are the sensitivity metric
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:77a0c17c6e3fc6180c6c15f1ed112f3a8421054e66f7c6fbfed2d8545fe67bb1
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# parameter-sensitivity-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how relaxing analysis thresholds in the DIMPLE DESI-MSI pipeline changes output metrics, by re-running preprocessing of B73 maize root data at relaxed ppm mass tolerance (2.5 → 5 ppm) and lowered background-enrichment thresholds (1.5 → 1.1) and comparing per-root peak counts against the ~196 peaks/root default baseline.

## When to use

Use when a published pipeline result (e.g., DIMPLE peak counts per B73 root) must be stress-tested by re-running preprocessing at altered thresholds to report percent changes relative to the default-parameter baseline.

## When NOT to use

- The goal is de novo annotation or identification of mass signatures (use Metlin Classic, LOTUS, or METASPACE instead of re-tuning DIMPLE thresholds).
- The data are not from a linear developmental gradient (e.g., MALDI or SIMS imaging of non-linear tissue), since DIMPLE is designed to leverage the linear root developmental axis.

## Inputs

- NetCDF .cdf DESI-MSI imaging files for B73 root replicates (e.g., B73-root11, B73-root2-5, B73-root-3-3)
- Matlab workspace files accompanying each root (e.g., B73-root11-workspace.mat)
- DIMPLE preprocessing configuration with default parameters (2.5 ppm mass tolerance, 1.5 minimum background-enrichment threshold)

## Outputs

- Per-root m/z peak counts for each relaxed-parameter run
- Average peaks per root and percent change versus default (expected: 223 peaks/root, +14% at 5 ppm; 199 peaks/root, +3% at enrichment 1.1)
- parameter_sensitivity_metrics.csv

## How to apply

Load the deposited B73 replicate DESI-MSI data (NetCDF .cdf files such as B73-root11, B73-root2-5, B73-root-3-3) and the corresponding Matlab workspaces from the DIMPLE-code repository. Re-run DIMPLE preprocessing with the ppm mass tolerance raised from 2.5 to 5 ppm, recording the number of m/z signatures identified per root; then re-run again with the minimum background-enrichment threshold lowered from 1.5 to 1.1. Compute the average peaks per root for each relaxed setting and the percent increase relative to the default-parameter runs (~196 peaks per root). The rationale is that a 14% gain (223 peaks/root at 5 ppm) versus only a 3% gain (199 peaks/root at enrichment 1.1) reveals which threshold dominates sensitivity, guiding parameter choices without forfeiting stringency.

## Related tools

- **DIMPLE** (Pipeline whose preprocessing is re-run at relaxed parameters (5 ppm tolerance; 1.1 background-enrichment threshold) to measure sensitivity of per-root peak counts) — github.com/dickinsonlab/DIMPLE-code
- **DESI-MSI** (Imaging modality that produced the maize-root mass spectrometry data whose peak counts are the sensitivity metric)

## Examples

```
cdf = batchcdfread('B73-root11.cdf'); ws = load('B73-root11-workspace.mat'); run DIMPLE with ppmTolerance=5 and record peaks per root
```

## Evaluation signals

- Relaxed 5 ppm run averages 223 peaks per B73 root, a +14% increase over the ~196 peaks/root default baseline (within small tolerance).
- Relaxed enrichment-threshold run (1.5 → 1.1) averages 199 peaks per root, a +3% increase, confirming enrichment threshold is far less sensitive than ppm tolerance.
- Per-root peak counts are computed for all three B73 replicates, matching the cdf/workspace file pairs listed in the README (B73-root11, B73-root2-5, B73-root-3-3).
- parameter_sensitivity_metrics.csv contains average peaks per root and percent change for both relaxed settings.
- Default-parameter re-runs reproduce the ~196 peaks per root baseline before relaxed runs are compared.

## Limitations

- README file-naming inconsistencies (e.g., 'B73-root2-3-worskpace.mat' typo) can cause workspace-loading failures if paths are assumed regular.
- OG2-P11-0 (Oaxacan Green root 3) appears to lack a listed workspace .mat file, limiting cross-variety sensitivity checks.
- Reported values (223/+14% and 199/+3%) are specific to the B73 root data and DIMPLE version described in Sama et al. 2025; other tissues or pipeline versions may not reproduce them.

## Evidence
