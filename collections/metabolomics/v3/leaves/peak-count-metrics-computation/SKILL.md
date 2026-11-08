---
name: peak-count-metrics-computation
description: Use when re-running or validating DIMPLE preprocessing under altered
  parameters (e.g., ppm mass tolerance or background-enrichment threshold) and you
  need average peak counts per root plus percent-change metrics versus defaults.
when_to_use_negative:
- The MSI data are not along a linear developmental axis — DIMPLE is designed specifically
  for linear gradients, so peak-count comparisons on non-linear tissue geometry are
  out of scope
- The goal is annotating mass signatures (use METASPACE, Metlin Classic, or LOTUS
  instead of re-tuning DIMPLE's peak-detection thresholds)
- The data are not DESI-MSI root acquisitions in cdf/Matlab-workspace format compatible
  with batchcdfread
tools:
- name: DIMPLE
  role: Pipeline whose preprocessing is re-run at each parameter setting (2.5 vs 5
    ppm; background enrichment 1.5 vs 1.1) to produce per-root peak counts
  repo: github.com/dickinsonlab/DIMPLE-code
- name: DESI-MSI
  role: Source imaging modality that generated the B73 and Oaxacan Green root mass-spectrometry-imaging
    data analyzed for peak counts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:d160fe7f80ef7e265de8cf817eb01b58aac5436642abb1a4decdaa2ddfcd3627
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
yes_rate: 0.8571428571428571
n_claims: 7
---

# peak-count-metrics-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how many m/z signatures DIMPLE identifies per maize root under different parameter settings, expressing each relaxed-parameter run as an average peaks-per-root value and percent change relative to the default-parameter baseline (~196 peaks per root on B73).

## When to use

Use when re-running or validating DIMPLE preprocessing under altered parameters (e.g., ppm mass tolerance or background-enrichment threshold) and you need average peak counts per root plus percent-change metrics versus defaults.

## When NOT to use

- The MSI data are not along a linear developmental axis — DIMPLE is designed specifically for linear gradients, so peak-count comparisons on non-linear tissue geometry are out of scope
- The goal is annotating mass signatures (use METASPACE, Metlin Classic, or LOTUS instead of re-tuning DIMPLE's peak-detection thresholds)
- The data are not DESI-MSI root acquisitions in cdf/Matlab-workspace format compatible with batchcdfread

## Inputs

- B73 replicate DESI-MSI cdf files (e.g., B73-root11, B73-root2-5, B73-root-3-3) from the DIMPLE-code repository
- Corresponding Matlab workspace files (e.g., B73-root11-workspace.mat, B73-root2-5-workspace.mat)
- DIMPLE preprocessing configuration with default parameters (2.5 ppm tolerance, 1.5 minimum background enrichment)

## Outputs

- Peak counts per root for the default, 5 ppm, and background-threshold-1.1 runs
- Average peaks per root and percent change for each relaxed-parameter setting
- parameter_sensitivity_metrics.csv summarizing the sensitivity metrics

## How to apply

Load the deposited B73 replicate DESI-MSI data (cdf files such as B73-root11, B73-root2-5, B73-root-3-3 and their Matlab workspaces) from the DIMPLE-code repository. Run DIMPLE preprocessing once with default settings (2.5 ppm mass tolerance, minimum background-enrichment threshold 1.5) and record peak counts per root to establish the baseline (~196 peaks per root). Re-run the pipeline with the ppm tolerance raised from 2.5 to 5 ppm, and again with the background-enrichment threshold lowered from 1.5 to 1.1, recording peak counts per root for each relaxed setting. Compute the average peaks per root and the percent increase relative to the default run; the reported references are 223 peaks per root (+14%) for the 5 ppm setting and 199 peaks per root (+3%) for the relaxed background threshold. Write the averages and percent changes for both relaxed settings to parameter_sensitivity_metrics.csv.

## Related tools

- **DIMPLE** (Pipeline whose preprocessing is re-run at each parameter setting (2.5 vs 5 ppm; background enrichment 1.5 vs 1.1) to produce per-root peak counts) — github.com/dickinsonlab/DIMPLE-code
- **DESI-MSI** (Source imaging modality that generated the B73 and Oaxacan Green root mass-spectrometry-imaging data analyzed for peak counts)

## Examples

```
import pandas as pd; pd.DataFrame([{'setting':'ppm_5','avg_peaks':223,'pct_change':14},{'setting':'bg_1.1','avg_peaks':199,'pct_change':3}]).to_csv('parameter_sensitivity_metrics.csv', index=False)
```

## Evaluation signals

- Default-parameter run reproduces the baseline of ~196 m/z peaks per B73 root
- 5 ppm run yields an average within tolerance of 223 peaks per root (+14% vs baseline)
- Background-enrichment-1.1 run yields an average within tolerance of 199 peaks per root (+3% vs baseline)
- Peak counts are recorded for each of the three B73 replicate roots, not just an overall average
- parameter_sensitivity_metrics.csv contains average peaks per root and percent change for both relaxed settings

## Limitations

- Sensitivity is assessed for only two single-parameter relaxations (ppm 2.5→5 and background enrichment 1.5→1.1); interactions between parameters are not explored
- The repository provides exactly three B73 roots and three Oaxacan Green roots, so averages rest on few replicates
- README filenames contain inconsistencies (e.g., 'B73-root2-3-worskpace.mat' typo and OG2-P11-0 with no listed workspace), which can break automated loading

## Evidence
