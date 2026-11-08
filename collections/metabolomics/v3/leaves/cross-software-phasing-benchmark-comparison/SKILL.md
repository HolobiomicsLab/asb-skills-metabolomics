---
name: cross-software-phasing-benchmark-comparison
description: Use when you have a replicate series of 1D-NMR spectra (e.g. metabolite
  mix replicates or Daphnia extract CryoProbe spectra) and must decide which software's
  automatic phase correction yields the most reproducible phases across the series.
when_to_use_negative:
- You only need to phase a single spectrum for visual inspection — no replicate series
  exists to compute within-series dispersion.
- The task is isotopomer distribution analysis from HSQC + GC-MS data (metabolabpytools
  territory), not 1D phase correction.
- Only one software package is available, so a cross-software comparison is impossible.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3292
tools:
- name: MetaboLabPy 0.9.66
  role: Primary processing environment; runs both per-spectrum automated phase correction
    and baseline-based/reference phase correction on both series
  repo: https://github.com/ludwigc/MetaboLabPy
- name: MetaboLab
  role: Comparison tool; its apk5 automatic phase correction algorithm is benchmarked
    (it failed on the metabolite mix spectra)
- name: TopSpin
  role: Comparison tool; apk0.noe automatic phase correction is benchmarked and gave
    the most consistent results
- name: MNova
  role: Comparison tool; series are manually phased and phase angles recorded
- name: numpy
  role: Numerical computation of per-spectrum phase angles and within-series phase
    dispersion
- name: pandas
  role: Tabulation and export of the phase-angle table per method
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:8ab437a3af6b8fcd2b6957687726c4bf003c3b96ae307336e399a02b47ec4d39
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ludwigc/MetaboLabPy
  tools:
  - MNova
  - MetaboLab
  - MetaboLabPy 0.9.66
  - TopSpin
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# cross-software-phasing-benchmark-comparison

## Summary

Benchmark the within-series phase consistency of automatic phase correction algorithms across NMR software packages (MetaboLabPy, MetaboLab apk5, TopSpin apk0.noe, MNova) on replicate spectra series. It quantifies and compares phase dispersion per tool rather than trusting a single package's phasing.

## When to use

Use when you have a replicate series of 1D-NMR spectra (e.g. metabolite mix replicates or Daphnia extract CryoProbe spectra) and must decide which software's automatic phase correction yields the most reproducible phases across the series.

## When NOT to use

- You only need to phase a single spectrum for visual inspection — no replicate series exists to compute within-series dispersion.
- The task is isotopomer distribution analysis from HSQC + GC-MS data (metabolabpytools territory), not 1D phase correction.
- Only one software package is available, so a cross-software comparison is impossible.

## Inputs

- Metabolite mix replicate 1D-NMR spectra series
- Daphnia extract CryoProbe 1D-NMR spectra series
- MetaboLabPy 0.9.66, MetaboLab (apk5), TopSpin (apk0.noe), MNova installations

## Outputs

- Per-spectrum phase-angle table per software/method
- Within-series phase dispersion values per phasing method
- Consistency plots in the style of Figures S4 and S5

## How to apply

Load the metabolite mix replicate series and the Daphnia extract CryoProbe 1D-NMR series into MetaboLabPy 0.9.66 and into the comparison tools (MetaboLab, TopSpin, MNova). Run each phasing method on every spectrum: MetaboLabPy's per-spectrum automated phase correction, MetaboLabPy's reference/baseline-based phase correction, MetaboLab's apk5, TopSpin's apk0.noe, and manual phasing in MNova. Record any algorithm failures explicitly — in the benchmark MetaboLab's apk5 could not perform automatic phase correction on the metabolite mix spectra. Extract per-spectrum phase angles from each tool, compute within-series phase dispersion across replicates as the consistency metric, and plot per-method consistency in the style of Figures S4 and S5 alongside an exported phase-angle table. The rationale is that a phasing algorithm suited to a processing workflow should minimise phase variance across replicates, not just optimise each spectrum in isolation.

## Related tools

- **MetaboLabPy 0.9.66** (Primary processing environment; runs both per-spectrum automated phase correction and baseline-based/reference phase correction on both series) — https://github.com/ludwigc/MetaboLabPy
- **MetaboLab** (Comparison tool; its apk5 automatic phase correction algorithm is benchmarked (it failed on the metabolite mix spectra))
- **TopSpin** (Comparison tool; apk0.noe automatic phase correction is benchmarked and gave the most consistent results)
- **MNova** (Comparison tool; series are manually phased and phase angles recorded)
- **numpy** (Numerical computation of per-spectrum phase angles and within-series phase dispersion)
- **pandas** (Tabulation and export of the phase-angle table per method)

## Examples

```
metabolabpy --help  # then load the 1D series and run per-spectrum automated or baseline-based phase correction on each replicate
```

## Evaluation signals

- MetaboLab apk5 fails to perform automatic phase correction on the metabolite mix spectra — this expected failure must be reproduced and recorded.
- TopSpin apk0.noe and MetaboLabPy's baseline-based phase correction yield the lowest within-series phase dispersion across each series.
- MetaboLabPy's baseline-based phasing gives a slightly better baseline than TopSpin apk0.noe.
- The exported phase-angle table contains one phase angle per spectrum per method, allowing dispersion to be recomputed independently.
- Consistency plots match the style of Figures S4 and S5 (per-method within-series phase consistency).

## Limitations

- MetaboLab's apk5 algorithm cannot perform automatic phase correction on metabolite mix spectra, so no apk5 consistency value exists for that series.
- Benchmark conclusions are specific to the metabolite mix replicates and Daphnia extract CryoProbe spectra tested; other sample types or instruments may rank the algorithms differently.
- Comparison requires the same spectra to be processed in four separate packages (MetaboLabPy, MetaboLab, TopSpin, MNova), which is operationally demanding.

## Evidence

- [results] MetaboLab's algorithm could not perform an automatic phase correction on the metabolite mix spectra, while TopSpin and MetaboLabPy's baseline-based algorithm gave the most consistent results: [evidence span withheld — anchor status: not_found]
- [abstract] choice of algorithms for automated phase correction: "This includes a choice of algorithms for automated phase correction, segmental alignment, spectral scaling, variance stabilisation, export to various software platforms"
- [abstract] MetaboLabPy 0.9.66 processes the spectra: "MetaboLabPy (version 0.9.66) is a free and open-source software package used to process 1D- and 2D-NMR spectra"
- [abstract] Python toolboxes used for the analysis: "The software is implemented in Python and uses numerous Python toolboxes, such as numpy, scipy, pandas, etc."
- [readme] Repository README for the package: "Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis"
