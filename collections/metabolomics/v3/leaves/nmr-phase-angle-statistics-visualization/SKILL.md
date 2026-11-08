---
name: nmr-phase-angle-statistics-visualization
description: Use when comparing automatic phase correction algorithms (e.g. MetaboLabPy
  baseline-based, MetaboLab apk5, TopSpin apk0.noe, MNova) on replicate NMR series
  and you need a quantitative within-series phase-consistency benchmark plus comparison
  figures.
when_to_use_negative:
- You need a single spectrum phased correctly rather than a cross-algorithm consistency
  benchmark over a replicate series.
- Your goal is downstream NMR pre-processing steps (segmental alignment, spectral
  scaling, variance stabilisation) rather than evaluating phase correction itself.
- The spectra have no replicate structure, so within-series dispersion cannot be computed
  meaningfully.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: MetaboLabPy
  role: Runs both per-spectrum automated and reference/baseline-based phase correction
    on the NMR series and supplies phase angles for benchmarking
  repo: https://github.com/ludwigc/MetaboLabPy
- name: numpy
  role: Numerical computation of within-series phase dispersion across replicates
- name: pandas
  role: Tabulation and export of the per-spectrum phase-angle table
- name: MetaboLab
  role: Comparison tool; its apk5 algorithm is run on both series (fails on the metabolite
    mix)
- name: TopSpin
  role: Comparison tool; apk0.noe automatic phase correction is run on both series
- name: MNova
  role: Comparison tool; series are phased manually in MNova for the benchmark
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:7221da6602b38bb095a5447c715caff419dacce9d8777546ad7751262fc960f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ludwigc/MetaboLabPy
  tools:
  - MNova
  - MetaboLab
  - MetaboLabPy
  - TopSpin
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# nmr-phase-angle-statistics-visualization

## Summary

Extracts per-spectrum phase angles from 1D-NMR replicate series processed with different automatic phase correction algorithms and quantifies within-series phase dispersion, then plots consistency comparisons in the style of the paper's Figures S4 and S5.

## When to use

Use when comparing automatic phase correction algorithms (e.g. MetaboLabPy baseline-based, MetaboLab apk5, TopSpin apk0.noe, MNova) on replicate NMR series and you need a quantitative within-series phase-consistency benchmark plus comparison figures.

## When NOT to use

- You need a single spectrum phased correctly rather than a cross-algorithm consistency benchmark over a replicate series.
- Your goal is downstream NMR pre-processing steps (segmental alignment, spectral scaling, variance stabilisation) rather than evaluating phase correction itself.
- The spectra have no replicate structure, so within-series dispersion cannot be computed meaningfully.

## Inputs

- Metabolite mix replicate 1D-NMR spectral series
- Daphnia extract CryoProbe 1D-NMR spectral series
- Per-spectrum phase angles exported from MetaboLabPy 0.9.66, MetaboLab (apk5), TopSpin (apk0.noe), and MNova

## Outputs

- Within-series phase dispersion (consistency) statistics per phasing method
- Phase-angle table exported per method and per spectrum
- Within-series phase-consistency comparison plots in the style of Figures S4 and S5

## How to apply

Load the metabolite mix replicate series and the Daphnia extract CryoProbe 1D-NMR series, then apply each phasing method: MetaboLabPy 0.9.66 per-spectrum automated phase correction and its reference/baseline-based phase correction, MetaboLab's apk5, TopSpin's apk0.noe, and manual phasing in MNova. Extract the per-spectrum phase angles from every tool and compute within-series phase dispersion across replicates as the consistency metric, using numpy and pandas for the tabulation and statistics. Record any algorithm failures — notably MetaboLab apk5 failing on the metabolite mix spectra. Plot within-series phase consistency per method in the style of Figures S4 and S5 and export the phase-angle table. The rationale is that lower dispersion across replicate spectra of the same series indicates a more reproducible, consistent phasing algorithm.

## Related tools

- **MetaboLabPy** (Runs both per-spectrum automated and reference/baseline-based phase correction on the NMR series and supplies phase angles for benchmarking) — https://github.com/ludwigc/MetaboLabPy
- **numpy** (Numerical computation of within-series phase dispersion across replicates)
- **pandas** (Tabulation and export of the per-spectrum phase-angle table)
- **MetaboLab** (Comparison tool; its apk5 algorithm is run on both series (fails on the metabolite mix))
- **TopSpin** (Comparison tool; apk0.noe automatic phase correction is run on both series)
- **MNova** (Comparison tool; series are phased manually in MNova for the benchmark)

## Examples

```
metabolabpy --help
```

## Evaluation signals

- MetaboLab's apk5 algorithm fails to produce an automatic phase correction on the metabolite mix spectra — this failure must be reproduced and recorded.
- TopSpin apk0.noe and MetaboLabPy's baseline-based algorithm should emerge as the most consistent (lowest within-series phase dispersion) methods on each series.
- MetaboLabPy's baseline-based correction should show a slightly better baseline than TopSpin apk0.noe.
- The exported phase-angle table must contain one phase angle per spectrum per tool for both series.
- Resulting plots should match the layout/content of the paper's Figures S4 and S5.

## Limitations

- MetaboLab's apk5 automatic phase correction cannot be applied to the metabolite mix spectra, so that method is missing for that series.
- Consistency (low dispersion across replicates) is the benchmark metric; it does not by itself prove absolute phase correctness.

## Evidence

- [results] apk5 fails on metabolite mix; TopSpin and MetaboLabPy most consistent: [evidence span withheld — anchor status: not_found]
- [supplementary] workflow steps for the benchmark: [evidence span withheld — anchor status: not_found]
- [abstract] MetaboLabPy provides automated phase correction: "This includes a choice of algorithms for automated phase correction, segmental alignment, spectral scaling, variance stabilisation"
- [abstract] MetaboLabPy implemented in Python with numpy, scipy, pandas: "The software is implemented in Python and uses numerous Python toolboxes, such as numpy, scipy, pandas, etc"
