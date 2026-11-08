---
name: ppm-mass-tolerance-parameter-tuning
description: Use when running DIMPLE preprocessing on DESI-MSI root data and you need
  to choose or justify the 2.5 ppm mass tolerance (default) versus 5 ppm, or the 1.5
  versus 1.1 background-enrichment threshold, based on peak-count sensitivity.
when_to_use_negative:
- Data are not DESI-MSI root sections analyzed along a linear developmental gradient
  by DIMPLE
- You need absolute metabolite annotation confidence rather than relative peak-count
  sensitivity; relaxed tolerance inflates candidate mass signatures without validating
  them
- Input is already a peak table and only annotation or statistics are required — no
  preprocessing re-run is possible
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DIMPLE
  role: Preprocessing pipeline whose ppm mass tolerance and background-enrichment
    threshold parameters are swept and whose per-root peak counts are recorded
  repo: github.com/dickinsonlab/DIMPLE-code
- name: DESI-MSI
  role: Source mass spectrometry imaging data whose mass signatures are re-detected
    under each tolerance setting
- name: batchcdfread
  role: Matlab function developed in the Zare lab for reading the cdf root data into
    DIMPLE for each parameter run
  repo: github.com/dickinsonlab/DIMPLE-code
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:984710f36a62324dc47f78e1bfc9cca23884e17cc0b48f4d907599cee02dd906
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
  - batchcdfread
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# ppm-mass-tolerance-parameter-tuning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tuning procedure for the DIMPLE preprocessing mass tolerance (ppm) and background-enrichment thresholds, quantifying how relaxing them changes the average number of m/z signatures detected per DESI-MSI maize root.

## When to use

Use when running DIMPLE preprocessing on DESI-MSI root data and you need to choose or justify the 2.5 ppm mass tolerance (default) versus 5 ppm, or the 1.5 versus 1.1 background-enrichment threshold, based on peak-count sensitivity.

## When NOT to use

- Data are not DESI-MSI root sections analyzed along a linear developmental gradient by DIMPLE
- You need absolute metabolite annotation confidence rather than relative peak-count sensitivity; relaxed tolerance inflates candidate mass signatures without validating them
- Input is already a peak table and only annotation or statistics are required — no preprocessing re-run is possible

## Inputs

- B73 replicate DESI-MSI data files in cdf format (B73-root11, B73-root2-5, B73-root-3-3)
- Corresponding Matlab workspaces (B73-root11-workspace.mat, B73-root2-5-workspace.mat, B73-root2-3-worskpace.mat)
- DIMPLE preprocessing configuration with ppm mass tolerance and minimum background-enrichment threshold parameters

## Outputs

- Average m/z signatures identified per B73 root for each parameter setting
- Percent change in peak counts relative to the default-parameter baseline (~196 peaks/root)
- parameter_sensitivity_metrics.csv with average peaks per root and percent change for each relaxed setting

## How to apply

Load the deposited B73 replicate DESI-MSI data (NetCDF/cdf files plus the Matlab workspaces, e.g. B73-root11 and B73-root11-workspace.mat) from the DIMPLE-code repository and run the DIMPLE preprocessing once with the default 2.5 ppm mass tolerance to establish the baseline peak counts per root. Re-run the same preprocessing with the mass tolerance raised to 5 ppm and record the resulting per-root peak counts; then re-run again with the minimum background-enrichment threshold lowered from 1.5 to 1.1, recording peak counts. Compute the average peaks per root and the percent increase relative to the default-parameter runs; the reported results are 223 peaks per root (+14%) at 5 ppm and 199 peaks per root (+3%) at the relaxed enrichment threshold against a ~196-peak baseline. Decision point: keep the stricter 2.5 ppm tolerance when sensitivity analysis shows only a marginal gain, since tolerating looser mass accuracy admits more false-positive signature matches.

## Related tools

- **DIMPLE** (Preprocessing pipeline whose ppm mass tolerance and background-enrichment threshold parameters are swept and whose per-root peak counts are recorded) — github.com/dickinsonlab/DIMPLE-code
- **DESI-MSI** (Source mass spectrometry imaging data whose mass signatures are re-detected under each tolerance setting)
- **batchcdfread** (Matlab function developed in the Zare lab for reading the cdf root data into DIMPLE for each parameter run) — github.com/dickinsonlab/DIMPLE-code

## Examples

```
matlab -r "batchcdfread('B73-root11.cdf'); run_dimple({'ppm',2.5}); ... run_dimple({'ppm',5}); write_parameter_sensitivity_metrics('parameter_sensitivity_metrics.csv');"
```

## Evaluation signals

- Computed average peaks per root at 5 ppm tolerance reproduces the reported 223 (a 14% increase) within a small tolerance
- Computed average peaks per root at the 1.1 background-enrichment threshold reproduces the reported 199 (a 3% increase) within a small tolerance
- Default-parameter baseline averages approximately 196 peaks per root before relaxing tolerance
- Percent-change metrics for both relaxed settings are written to parameter_sensitivity_metrics.csv in comparable form
- Per-root peak counts are recorded for all three B73 replicates under each setting, not just pooled averages

## Limitations

- Relaxing the ppm tolerance increases peak counts substantially (+14%) but also admits signatures with looser mass accuracy, so peak-count gains should not be read as discovery of validated metabolites
- Sensitivity numbers are tied to the three deposited B73 replicates and may not transfer to other roots, varieties, or imaging platforms
- Peak-count increases at relaxed enrichment thresholds are small (+3%), limiting the practical benefit of lowering the 1.5 threshold

## Evidence

- [full_text] 2.5 to 5 ppm tolerance increases average peaks per root by 14%: [evidence span withheld — anchor status: not_found]
