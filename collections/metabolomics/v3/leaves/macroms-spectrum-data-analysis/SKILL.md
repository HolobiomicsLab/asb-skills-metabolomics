---
name: macroms-spectrum-data-analysis
description: Use when you have macroMS-exported mass spectra (peak areas per target
  location) from a high-throughput screen — e.g., a 5 × 50 nanoliter droplet array
  — and need per-sample ratios and grouped statistics (mean ± std) to distinguish
  sample classes.
when_to_use_negative:
- The samples are smaller than 300 μm — the article states microMS is more effective
  for sub-300 μm targets, so use the microMS workflow instead
- You need targeted acquisition planning rather than analysis of already-acquired
  spectra — target finding, filtering, and device coordinate generation are upstream
  macroMS workflow steps, not part of this analysis skill
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: macroMS
  role: Web platform whose data analysis and visualization tools provide the pipeline
    for computing per-sample metrics (e.g., peak-area ratios) from the mass spectra
    acquired at the selected target locations
  repo: github.com/macroMS/macroMS_web
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:ebebdfbd2c097cbe320a9c343be9cd4e716321371a87d8c29d4d97cbc07978c8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - macroMS
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# macroms-spectrum-data-analysis

## Summary

Analyze the mass spectra exported from a macroMS screen (e.g., peak areas at m/z ~253.2 for C16:1 and ~255.2 for C16 fatty acyls) to compute per-sample metrics such as fatty acyl ratios, group them by sample class, and report group means and standard deviations as a per-sample table.

## When to use

Use when you have macroMS-exported mass spectra (peak areas per target location) from a high-throughput screen — e.g., a 5 × 50 nanoliter droplet array — and need per-sample ratios and grouped statistics (mean ± std) to distinguish sample classes.

## When NOT to use

- The samples are smaller than 300 μm — the article states microMS is more effective for sub-300 μm targets, so use the microMS workflow instead
- You need targeted acquisition planning rather than analysis of already-acquired spectra — target finding, filtering, and device coordinate generation are upstream macroMS workflow steps, not part of this analysis skill

## Inputs

- macroMS-exported mass spectra (peak areas per target, e.g., m/z ~253.2 for C16:1 and m/z ~255.2 for C16 fatty acyls)
- Sample-class labels mapping each droplet/target to its species or condition (e.g., P. fluorescens vs E. coli)
- Target geometry/coordinate information for the droplet array (e.g., 5 × 50 array layout)

## Outputs

- Per-droplet ratio table (one row per droplet; 500 rows for the reference 5 × 50 arrays, 250 per species)
- Per-group summary statistics (mean ± standard deviation of the C16:1/C16 ratio per species)
- CSV file of the per-droplet ratios

## How to apply

Load the macroMS-exported mass spectra as per-droplet peak areas for the compounds of interest (here, m/z ~253.2 representing C16:1 and m/z ~255.2 representing C16 fatty acyls). For each droplet, compute the C16:1 peak area divided by the C16 peak area. Group the per-droplet ratios by sample class (P. fluorescens vs E. coli) and compute the mean and standard deviation per group, expecting ~1.11 ± 0.04 for P. fluorescens and ~0.45 ± 0.06 for E. coli in the reference experiment. Write the per-droplet ratio table to CSV, keeping one row per droplet (500 droplets total, 250 per species in the reference 5 × 50 arrays). macroMS itself provides a data analysis and visualization pipeline for exactly this purpose: locating samples containing compounds of interest at ~1–2 s per sample throughput.

## Related tools

- **macroMS** (Web platform whose data analysis and visualization tools provide the pipeline for computing per-sample metrics (e.g., peak-area ratios) from the mass spectra acquired at the selected target locations) — github.com/macroMS/macroMS_web

## Examples

```
df["C16:1_over_C16"] = df["mz_253.2_area"] / df["mz_255.2_area"]; df.groupby("species")["C16:1_over_C16"].agg(["mean", "std"]); df.to_csv("droplet_ratios.csv", index=False)
```

## Evaluation signals

- Per-species mean ± std of the computed C16:1/C16 ratios match the reported values (1.11 ± 0.04 for P. fluorescens and 0.45 ± 0.06 for E. coli) within reasonable tolerance
- The output CSV exists and has the expected row count — one per-droplet ratio for all 500 droplets total, 250 per species
- The two species groups show distinct and reproducible separation (P. fluorescens ratio > 1, E. coli ratio < 0.5), consistent with the reported finding
- Each per-droplet ratio is computed as the C16:1 peak area (m/z ~253.2) divided by the C16 peak area (m/z ~255.2) for the same droplet, with no missing or NaN entries

## Limitations

- macroMS supports only limited data analysis — it is intended to aid locating samples containing compounds of interest, not for full statistical or downstream omics analysis
- The analysis depends on prior correct optical correction of the target image; moderate distortion and imperfect perspective must be resolved before coordinates are generated
- The comparison between approaches (e.g., microMS vs macroMS) is qualitative, with no quantitative benchmark provided in the article

## Evidence
