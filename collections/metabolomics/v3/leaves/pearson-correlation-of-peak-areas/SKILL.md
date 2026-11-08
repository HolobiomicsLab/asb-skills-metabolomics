---
name: pearson-correlation-of-peak-areas
description: Use when validating a GC×GC–TOFMS peak-picking/quantification workflow
  against reference software using paired peak-area tables, especially with a serial
  dilution series (e.g. 1 ppm Breath Mix QC) where both correlation and dilution linearity
  must be shown.
when_to_use_negative:
- Input is already a processed feature/area table — correlation is computed directly
  without rerunning GcDUO peak picking.
- No reference peak areas from a gold-standard tool or no dilution series exist —
  neither the correlation nor the linearity statistic can be computed.
- Goal is peak identification/annotation quality rather than quantitative area agreement
  (use cosine-similarity library matching evaluation instead).
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GcDUO
  role: R package that reads the CDF folder, picks peaks, and produces the $area matrix
    compared against ChromaTOF
  repo: https://github.com/mariallr/GcDuo
- name: ChromaTOF
  role: Gold-standard commercial GC×GC software whose peak areas serve as the reference
    in the correlation
- name: R
  role: Language in which GcDUO is implemented and in which the correlation and per-peak
    regressions are computed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:84b2a8b2c893055af02b7882e3ff93a09621d02329d949ecc509925beefb31c2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mariallr/GcDuo
  repo_url: https://github.com/mariallr/GcDuo
  tools:
  - ChromaTOF
  - GcDUO
  - R
  - dplyr
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# pearson-correlation-of-peak-areas

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarking skill that quantifies agreement between peak areas produced by an open-source GC×GC–MS processor (GcDUO) and a gold-standard tool (ChromaTOF) using Pearson correlation on a shared sample, plus linear-regression R² across a serial dilution series to confirm quantitative linearity.

## When to use

Use when validating a GC×GC–TOFMS peak-picking/quantification workflow against reference software using paired peak-area tables, especially with a serial dilution series (e.g. 1 ppm Breath Mix QC) where both correlation and dilution linearity must be shown.

## When NOT to use

- Input is already a processed feature/area table — correlation is computed directly without rerunning GcDUO peak picking.
- No reference peak areas from a gold-standard tool or no dilution series exist — neither the correlation nor the linearity statistic can be computed.
- Goal is peak identification/annotation quality rather than quantitative area agreement (use cosine-similarity library matching evaluation instead).

## Inputs

- GC×GC–TOFMS serial dilution raw data as a folder of CDF files (e.g. in-house Breath Mix QC dilution series)
- ChromaTOF reference peak-area table for the matched sample (gold-standard areas)
- Imported spectral library (MSP) for GcDUO annotation

## Outputs

- Per-peak area table across the dilution series (finalGcDuo$area)
- Pearson's correlation coefficient between GcDUO and ChromaTOF 1 ppm peak areas (~0.904)
- Mean dilution-curve R² from per-peak log-area vs dilution-level regressions (~0.950)
- Combined results file, e.g. breath_mix_linearity_results.csv

## How to apply

Process the raw CDF-folder serial dilution data through the full GcDUO pipeline: GcDUO::readFolderCDF (modulation time, m/z range, e.g. c(35, 440)), GcDUO::procesData with signal-to-noise thresholds in RT1/RT2 and win_size = 3 modulations, then GcDUO::ConsProcessing with the imported library to obtain the $area matrix (peak × sample). Extract the 1 ppm sample column from $area and compute Pearson's correlation of GcDUO peak areas against the ChromaTOF gold-standard areas (target ~0.904). For linearity, fit for each peak a linear regression of log-transformed area versus dilution level, compute per-peak R², and average across peaks (target mean R² > 0.950). The rationale is that matching the gold standard on both absolute area agreement and dilution-response linearity demonstrates quantitative consistency, not merely peak detection overlap.

## Related tools

- **GcDUO** (R package that reads the CDF folder, picks peaks, and produces the $area matrix compared against ChromaTOF) — https://github.com/mariallr/GcDuo
- **ChromaTOF** (Gold-standard commercial GC×GC software whose peak areas serve as the reference in the correlation)
- **R** (Language in which GcDUO is implemented and in which the correlation and per-peak regressions are computed)

## Examples

```
finalGcDuo <- GcDuo::ConsProcessing(GcDuoObject, DuoResults, lib, win_width = 20); r <- cor(finalGcDuo$area[, "1ppm"], chromaTOF_areas_1ppm, method = "pearson")
```

## Evaluation signals

- Pearson's correlation between GcDUO 1 ppm peak areas and ChromaTOF areas reproduces ~0.904.
- Mean dilution-curve R² (log area vs dilution level, averaged over peaks) reproduces ~0.950 (>0.95).
- $area matrix from GcDUO::ConsProcessing has one row per consensus feature and one column per dilution sample, with no features detected in only one sample included in the evaluation.
- Per-peak regression fits are computed on log-transformed areas so dilution response is assessed on the correct scale.

## Limitations

- Correlation was computed on a single sample (1 ppm) of the dilution series; agreement at other concentrations is not directly certified by this statistic.
- Decane was only detected when the signal-to-noise parameter was reduced from 3 to 1, so low-abundance peaks may be missing from the area matrix at default thresholds.
- Small chromatographic signals with too few characteristic fragments may fail deconvolution and be absent from the compared areas.
- R is not optimized for large memory loads or multi-core parallel processing; large batches may require high-performance computing environments.

## Evidence

- [results] 1 ppm Pearson r and dilution linearity: [evidence span withheld — anchor status: not_found]
