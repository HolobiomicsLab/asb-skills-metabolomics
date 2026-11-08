---
name: mass-spectrometry-peak-area-extraction
description: Use when you have macroMS-exported per-target mass spectra (peak areas
  at known m/z values) and need per-sample quantitative features, such as fatty acyl
  ratios, for comparing groups of samples.
when_to_use_negative:
- Samples are smaller than 300 μm, where the article states microMS is more effective
  than macroMS
- The analytes of interest are not represented by resolvable peaks at known m/z values
  in the MALDI-TOF spectra (macroMS provides only limited data analysis, not de novo
  peak identification)
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: macroMS
  role: Web platform that acquires spectra at selected target locations and provides
    data analysis and visualization tools for extracting and comparing peak areas
    across samples
  repo: github.com/macroMS/macroMS_web
- name: microMS
  role: Predecessor optically guided MS workflow; noted as more effective than macroMS
    for samples smaller than 300 μm
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:e2229fe0d3a6befc6ed350abc1204664cd6cc05fe112555784d9c3991b8c507e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - macroMS
  - microMS
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# mass-spectrometry-peak-area-extraction

## Summary

Extract MALDI-TOF peak areas at target m/z values from macroMS-acquired spectra (e.g., m/z ~253.2 for C16:1 and ~255.2 for C16 fatty acyls) and compute per-sample ratios to discriminate samples such as E. coli vs P. fluorescens cultures.

## When to use

Use when you have macroMS-exported per-target mass spectra (peak areas at known m/z values) and need per-sample quantitative features, such as fatty acyl ratios, for comparing groups of samples.

## When NOT to use

- Samples are smaller than 300 μm, where the article states microMS is more effective than macroMS
- The analytes of interest are not represented by resolvable peaks at known m/z values in the MALDI-TOF spectra (macroMS provides only limited data analysis, not de novo peak identification)

## Inputs

- macroMS-exported mass spectra with peak areas per sample/target (e.g., droplet-level peak areas at m/z ~253.2 and ~255.2)
- Sample metadata mapping each spectrum to its condition/species (e.g., P. fluorescens vs E. coli within a 5 × 50 droplet array)

## Outputs

- Per-sample ratio table (e.g., C16:1/C16 peak-area ratio per droplet) written to CSV
- Group-level summary statistics (mean ± std of ratios per species/condition)

## How to apply

Load the macroMS-exported mass spectra containing peak areas for the target ions — in the droplet-array demonstration, m/z ~253.2 represents C16:1 and m/z ~255.2 represents C16 fatty acyls. For each sample (e.g., each droplet in a 5 × 50 array), compute the ratio of the C16:1 peak area to the C16 peak area. Group the per-sample ratios by condition (P. fluorescens vs E. coli) and report mean ± standard deviation per group; the reported values were 1.11 ± 0.04 for P. fluorescens and 0.45 ± 0.06 for E. coli, so a computed group statistic within reasonable tolerance of those values validates the extraction. macroMS supports this step via its data analysis and visualization tools for searching spectra for specific compounds. Finally, write the per-sample ratio table to CSV for downstream inspection.

## Related tools

- **macroMS** (Web platform that acquires spectra at selected target locations and provides data analysis and visualization tools for extracting and comparing peak areas across samples) — github.com/macroMS/macroMS_web
- **microMS** (Predecessor optically guided MS workflow; noted as more effective than macroMS for samples smaller than 300 μm)

## Examples

```
df['ratio'] = df['C16:1_area_253.2'] / df['C16_area_255.2']; df.groupby('species')['ratio'].agg(['mean', 'std']).to_csv('ratios.csv')
```

## Evaluation signals

- Per-species mean ± std of computed C16:1/C16 ratios match reported values (1.11 ± 0.04 for P. fluorescens, 0.45 ± 0.06 for E. coli) within reasonable tolerance
- CSV output exists with the expected row count: per-droplet ratios for all 500 droplets (250 per species across the two 5 × 50 arrays)
- Peak assignment sanity check: C16:1 signal tracks m/z ~253.2 and C16 tracks m/z ~255.2 in the exported peak areas
- The two species' ratio distributions are distinct and reproducible across arrays, consistent with the reported discrimination

## Limitations

- macroMS provides only limited data analysis and visualization; full statistical treatment beyond group mean/std of ratios must be done externally
- The demonstrated discrimination is qualitative/empirical for the two tested species and depends on assigned peaks (m/z ~253.2 and ~255.2) being present
- High-throughput targeting is bounded (~10,000 targets, 1–2 s per sample); spectra for very large screens may need batched extraction

## Evidence

- [other] Load the macroMS-exported mass spectra (peak areas for m/z ~253.2 representing C16:1 and m/z ~255.2 representing C16 fatty acyls): [evidence span withheld — anchor status: not_found]
