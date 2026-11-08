---
name: fatty-acyl-peak-ratio-calculation
description: Use when MALDI-TOF spectra from macroMS screens (S. cerevisiae colonies
  or nanoliter droplet arrays) need per-sample fatty acyl ratios, e.g., C16:1/C16
  or C16:1/C18:1, to identify colonies or species with shifted fatty acyl profiles.
when_to_use_negative:
- Samples are smaller than 300 μm — the article notes 'For samples smaller than 300
  μm, microMS is more effective', so use the microMS workflow instead
- Input is already a ratio table or aggregate statistics — this skill operates on
  raw per-sample spectra/peak areas
- Spectra lack the diagnostic fatty acyl peaks (e.g., no m/z ~253.2/255.2 signal),
  making the ratio undefined or uninformative
tools:
- name: macroMS
  role: Web platform whose data analysis tools extract ion counts at target m/z values
    from MALDI-TOF spectra and compute per-sample peak ratios for screening
  repo: github.com/macroMS/macroMS_web
- name: microMS
  role: Alternative optically guided targeting software for samples smaller than 300
    μm, when the ratio calculation must be applied to microscopic samples
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:842cff88f68c664202cd12291e072bd7cf8848cb942df9dde0dd3f87f5bffba5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - macroMS
  - microMS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# fatty-acyl-peak-ratio-calculation

## Summary

Computes per-sample fatty acyl peak-area ratios (e.g., C16:1 at m/z ~253.2 over C16 at m/z ~255.2) from MALDI-TOF mass spectra collected with macroMS, and aggregates them into species- or genotype-level statistics to flag medium-chain fatty acid producers.

## When to use

Use when MALDI-TOF spectra from macroMS screens (S. cerevisiae colonies or nanoliter droplet arrays) need per-sample fatty acyl ratios, e.g., C16:1/C16 or C16:1/C18:1, to identify colonies or species with shifted fatty acyl profiles.

## When NOT to use

- Samples are smaller than 300 μm — the article notes 'For samples smaller than 300 μm, microMS is more effective', so use the microMS workflow instead
- Input is already a ratio table or aggregate statistics — this skill operates on raw per-sample spectra/peak areas
- Spectra lack the diagnostic fatty acyl peaks (e.g., no m/z ~253.2/255.2 signal), making the ratio undefined or uninformative

## Inputs

- macroMS-exported MALDI-TOF mass spectra (ion counts per m/z) keyed by sample/colony/droplet position
- Sample metadata mapping positions to species or genotype (E. coli vs P. fluorescens; Gly1250 SSM library)
- Diagnostic m/z definitions: ~253.2 (C16:1), ~255.2 (C16), C18:1 for yeast screen

## Outputs

- Per-sample fatty acyl ratio table (CSV, one row per droplet or colony)
- Group statistics: mean ± std per species/genotype (e.g., 1.11 ± 0.04 vs 0.45 ± 0.06)
- Ranked list of top-ratio colonies with genotype identification (e.g., G1250S)

## How to apply

Load the macroMS-exported MALDI-TOF spectra keyed by sample position (749 S. cerevisiae Gly1250 SSM colonies or two 5 × 50 droplet arrays of E. coli and P. fluorescens). Extract ion counts/peak areas at the diagnostic m/z values — ~253.2 for C16:1 and ~255.2 for C16 fatty acyls (droplet task) or C16:1 vs C18:1 (yeast screen) — using macroMS data analysis tools. Compute the per-sample ratio of the short-chain peak area to the long-chain peak area, then group by species/genotype and report mean ± standard deviation. In the yeast screen, rank colonies by the C16:1/18:1 ratio and take the top five as candidate MCFA-producing mutants; the droplet workflow simply compares per-species ratios. Write per-sample ratio tables to CSV for downstream verification against reported statistics.

## Related tools

- **macroMS** (Web platform whose data analysis tools extract ion counts at target m/z values from MALDI-TOF spectra and compute per-sample peak ratios for screening) — github.com/macroMS/macroMS_web
- **microMS** (Alternative optically guided targeting software for samples smaller than 300 μm, when the ratio calculation must be applied to microscopic samples)

## Examples

```
df["C16:1_over_C16"] = df["peak_area_mz_253.2"] / df["peak_area_mz_255.2"]; df.groupby("species")["C16:1_over_C16"].agg(["mean", "std"])
```

## Evaluation signals

- Droplet-array validation: per-species C16:1/C16 means reproduce 1.11 ± 0.04 for P. fluorescens and 0.45 ± 0.06 for E. coli, with per-droplet rows totaling 500 (250 per species)
- Yeast screen validation: population mean ± std of C16:1/18:1 ratios is 3.8 ± 2.9 and the top-five mean is 18.1 ± 1.1
- Top-ranked yeast colony by ratio corresponds to the G1250S genotype confirmed by DNA sequencing
- CSV output exists with expected row count (749 colonies or 500 droplets) and ratio column values are finite and positive
- Peak-area extraction targets exactly the reported m/z windows (~253.2 for C16:1, ~255.2 for C16) rather than arbitrary bins

## Limitations

- macroMS supports only 'limited data analysis' — deep statistical treatment must be done downstream
- The platform is designed for macroscopic (>300 μm) samples; smaller objects require microMS
- The upper limit for target number is ~10,000, so very large screens may exceed platform capacity
- Ratio-based screening identifies candidate mutants (e.g., short-chain producers) but genotype confirmation requires sequencing

## Evidence
