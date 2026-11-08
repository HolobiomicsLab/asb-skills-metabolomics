---
name: replicate-series-phase-consistency-analysis
description: Use when a series of replicate 1D-NMR spectra (e.g. metabolite mix replicates
  or Daphnia extract CryoProbe spectra) must be phased automatically and you need
  to compare which algorithm gives the most reproducible phases.
when_to_use_negative:
- Spectra that will be phased manually rather than by automatic phase correction algorithms
  — the benchmark compares automated algorithms (apk5, apk0.noe, MetaboLabPy) only
- A single spectrum rather than a replicate series — within-series phase dispersion
  cannot be computed without multiple replicates
- 2D-NMR series or tracer/isotopomer analysis — this skill covers 1D-NMR phase consistency
  only, not the metabolabpytools isotopomer workflow
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3292
tools:
- name: MetaboLabPy 0.9.66
  role: 'Primary tool: runs per-spectrum automated and baseline-based (reference)
    phase correction on both spectra series and exports phases'
  repo: https://github.com/ludwigc/MetaboLabPy
- name: MetaboLab
  role: 'Comparison tool: provides the apk5 automatic phase correction algorithm,
    which fails on the metabolite mix spectra'
- name: TopSpin
  role: 'Comparison tool: provides the apk0.noe automatic phase correction algorithm,
    among the most consistent in the benchmark'
- name: MNova
  role: 'Comparison tool: series are phased manually in MNova for the benchmark'
- name: numpy
  role: Python toolbox used in MetaboLabPy for computing phase-angle dispersion across
    replicates
- name: pandas
  role: Python toolbox used in MetaboLabPy for assembling and exporting the phase-angle
    table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:fafb99aabd9f0f1c2ef8932d57f822ae795fab34d15c5a5a264061ab23db4adc
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
yes_rate: 0.6
n_claims: 10
---

# replicate-series-phase-consistency-analysis

## Summary

Benchmark within-series phase consistency of automatic phase correction algorithms (MetaboLabPy baseline-based and per-spectrum, MetaboLab apk5, TopSpin apk0.noe, MNova) across replicate 1D-NMR spectra series by extracting per-spectrum phase angles and comparing within-series dispersion.

## When to use

Use when a series of replicate 1D-NMR spectra (e.g. metabolite mix replicates or Daphnia extract CryoProbe spectra) must be phased automatically and you need to compare which algorithm gives the most reproducible phases.

## When NOT to use

- Spectra that will be phased manually rather than by automatic phase correction algorithms — the benchmark compares automated algorithms (apk5, apk0.noe, MetaboLabPy) only
- A single spectrum rather than a replicate series — within-series phase dispersion cannot be computed without multiple replicates
- 2D-NMR series or tracer/isotopomer analysis — this skill covers 1D-NMR phase consistency only, not the metabolabpytools isotopomer workflow

## Inputs

- metabolite mix replicate 1D-NMR spectra series
- Daphnia extract CryoProbe 1D-NMR spectra series
- per-spectrum phase angles exported from each phasing tool (MetaboLabPy, MetaboLab, TopSpin, MNova)

## Outputs

- within-series phase dispersion (consistency) metric per algorithm per series
- phase-angle table exported per spectrum and per method
- within-series phase consistency plots in the style of Figures S4 and S5

## How to apply

Load the metabolite mix replicate series and the Daphnia extract CryoProbe 1D-NMR series into MetaboLabPy 0.9.66 and the comparison tools (MetaboLab, TopSpin, MNova). Run MetaboLabPy per-spectrum automated phase correction and its reference/baseline-based phase correction on every spectrum in both series, then run MetaboLab's apk5 and TopSpin's apk0.noe, manually phasing the series in MNova; record that MetaboLab apk5 fails on the metabolite mix spectra. Extract per-spectrum phase angles from each tool and compute within-series phase dispersion across replicates as the consistency metric. Plot within-series phase consistency per method in the style of Figures S4 and S5 and export the phase-angle table; validation requires apk5 failure on the metabolite mix and lowest dispersion for TopSpin apk0.noe and MetaboLabPy baseline-based phasing, with MetaboLabPy yielding a slightly better baseline.

## Related tools

- **MetaboLabPy 0.9.66** (Primary tool: runs per-spectrum automated and baseline-based (reference) phase correction on both spectra series and exports phases) — https://github.com/ludwigc/MetaboLabPy
- **MetaboLab** (Comparison tool: provides the apk5 automatic phase correction algorithm, which fails on the metabolite mix spectra)
- **TopSpin** (Comparison tool: provides the apk0.noe automatic phase correction algorithm, among the most consistent in the benchmark)
- **MNova** (Comparison tool: series are phased manually in MNova for the benchmark)
- **numpy** (Python toolbox used in MetaboLabPy for computing phase-angle dispersion across replicates)
- **pandas** (Python toolbox used in MetaboLabPy for assembling and exporting the phase-angle table)

## Examples

```
metabolabpy --help
```

## Evaluation signals

- MetaboLab apk5 fails to produce an automatic phase correction on the metabolite mix spectra — confirm this failure is reproduced
- TopSpin apk0.noe and MetaboLabPy baseline-based phasing yield the lowest within-series phase dispersion across each series
- MetaboLabPy baseline-based correction yields a slightly better baseline than TopSpin apk0.noe
- Phase-angle table contains one phase angle per spectrum per method for every spectrum in both series
- Within-series consistency plots match the style of Figures S4 and S5

## Limitations

- MetaboLab's apk5 algorithm cannot perform automatic phase correction on the metabolite mix spectra, so it cannot be compared on that series
- MNova phasing is manual, so its consistency depends on the operator rather than an automatic algorithm

## Evidence

- [abstract] This includes a choice of algorithms for automated phase correction, segmental alignment, spectral scaling, variance stabilisation: "This includes a choice of algorithms for automated phase correction, segmental alignment, spectral scaling, variance stabilisation"
- [abstract] MetaboLabPy (version 0.9.66) is a free and open-source software package used to process 1D- and 2D-NMR spectra: "MetaboLabPy (version 0.9.66) is a free and open-source software package used to process 1D- and 2D-NMR spectra"
- [abstract] The software is implemented in Python and uses numerous Python toolboxes, such as numpy, scipy, pandas, etc: "The software is implemented in Python and uses numerous Python toolboxes, such as numpy, scipy, pandas, etc"
- [readme] Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis: "Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis"
