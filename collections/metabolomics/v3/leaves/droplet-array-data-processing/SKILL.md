---
name: droplet-array-data-processing
description: Use when you have macroMS-exported spectra for a nanoliter droplet array
  (e.g., bacterial cultures spotted by a liquid handler) and need per-droplet fatty
  acyl peak-area ratios to classify samples.
when_to_use_negative:
- Samples smaller than 300 μm — the article states 'For samples smaller than 300 μm,
  microMS is more effective', so use the microMS workflow instead of the macroMS droplet-array
  route
- You need full statistical or spectral analysis beyond macroMS's limited data analysis/visualization
  tools — the platform offers target finding, filtering, and coordinate generation,
  not complete downstream statistics
- Arrays exceeding ~10,000 targets, since the stated upper limit for target number
  was found to be ~10,000
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: macroMS
  role: Web platform that generates device coordinates from scanned droplet-array
    images and exports mass spectra with peak areas used as the input for the ratio
    computation
  repo: github.com/macroMS/macroMS_web
- name: microMS
  role: Complementary suite for optically guided MS of samples smaller than 300 μm;
    not used for macroscopic droplet arrays
- name: flexImaging (Bruker Corp.)
  role: Contrasting instrument software for imaging MS that is not designed for large
    numbers of samples, motivating macroMS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:0a770b1633fd50f06d87a2326287f87aa3878c487907ce34f5d4a82c20f23179
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - flexImaging (Bruker Corp.)
  - macroMS
  - microMS
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# droplet-array-data-processing

## Summary

Post-acquisition processing of MALDI-TOF mass spectra from macroMS-guided nanoliter droplet arrays: extract per-droplet peak areas for fatty acyl ions and compute species-discriminating ratios (e.g., C16:1/C16).

## When to use

Use when you have macroMS-exported spectra for a nanoliter droplet array (e.g., bacterial cultures spotted by a liquid handler) and need per-droplet fatty acyl peak-area ratios to classify samples.

## When NOT to use

- Samples smaller than 300 μm — the article states 'For samples smaller than 300 μm, microMS is more effective', so use the microMS workflow instead of the macroMS droplet-array route
- You need full statistical or spectral analysis beyond macroMS's limited data analysis/visualization tools — the platform offers target finding, filtering, and coordinate generation, not complete downstream statistics
- Arrays exceeding ~10,000 targets, since the stated upper limit for target number was found to be ~10,000

## Inputs

- macroMS-exported mass spectra (peak areas) for each droplet in a 5 × 50 nanoliter droplet array (500 droplets total)
- Droplet-to-species mapping (e.g., 250 droplets per species for E. coli and P. fluorescens)
- Peak assignments (m/z ~253.2 → C16:1; m/z ~255.2 → C16 fatty acyls)

## Outputs

- Per-droplet C16:1/C16 ratio table (CSV, one row per droplet)
- Per-species mean ± standard deviation of the ratios (reported: 1.11 ± 0.04 for P. fluorescens, 0.45 ± 0.06 for E. coli)

## How to apply

Load the macroMS-exported mass spectra (peak areas) for the droplet array, locating the fatty acyl peaks at m/z ~253.2 for C16:1 and m/z ~255.2 for C16. For each droplet, calculate the ratio of the C16:1 peak area to the C16 peak area. Group per-droplet ratios by sample identity (e.g., P. fluorescens vs E. coli) and compute mean and standard deviation per group to quantify discrimination. Write the per-droplet ratio table to CSV with one row per droplet. macroMS itself provides 'limited data analysis and visualization tools' for screening spectra, so this ratio computation typically runs on its exported peak-area data; the platform acquired these droplet-array spectra at a rate of ~2 s/sample.

## Related tools

- **macroMS** (Web platform that generates device coordinates from scanned droplet-array images and exports mass spectra with peak areas used as the input for the ratio computation) — github.com/macroMS/macroMS_web
- **microMS** (Complementary suite for optically guided MS of samples smaller than 300 μm; not used for macroscopic droplet arrays)
- **flexImaging (Bruker Corp.)** (Contrasting instrument software for imaging MS that is not designed for large numbers of samples, motivating macroMS)

## Examples

```
import pandas as pd; df = pd.read_csv('macroMS_peak_areas.csv'); df['ratio'] = df['C16:1_mz253.2'] / df['C16_mz255.2']; df.groupby('species')['ratio'].agg(['mean','std']); df.to_csv('droplet_ratios.csv', index=False)
```

## Evaluation signals

- Per-species mean ± std of computed C16:1/C16 ratios match the reported values (1.11 ± 0.04 for P. fluorescens and 0.45 ± 0.06 for E. coli) within reasonable tolerance
- CSV output exists with the expected row count: per-droplet ratios for all 500 droplets (250 per species in the two 5 × 50 arrays)
- Per-droplet ratios are computed only from the expected fatty acyl peaks: C16:1 at m/z ~253.2 and C16 at m/z ~255.2
- The two species' ratio distributions are distinct and reproducible (no overlap of mean ± std between groups)
- Spectra were acquired for all targets in the macroMS-generated geometry file, consistent with the 1–2 s/sample screening rate

## Limitations

- The stated upper limit for target number was found to be ~10,000, presented without measurement details or uncertainty
- No version information for macroMS is stated
- Sample targeting reliability is only documented at 300 μm with no detail on number of replicates or statistical treatment; the microMS comparison is qualitative

## Evidence
