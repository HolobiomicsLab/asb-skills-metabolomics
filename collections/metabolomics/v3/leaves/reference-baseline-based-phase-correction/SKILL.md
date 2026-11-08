---
name: reference-baseline-based-phase-correction
description: Use when processing a series of 1D-NMR metabolomics spectra (e.g. replicate
  mixtures or extract series in MetaboLabPy 0.9.66) where within-series phase consistency
  across replicates matters, or when comparing against apk0.noe/apk5/MNova phasing.
when_to_use_negative:
- When the goal is isotopomer/tracer analysis (metabolabpytools HSQC + GC-MS neural-network
  workflow) rather than 1D-NMR phasing
- When only a single spectrum needs phasing with no replicate series — the consistency
  advantage over per-spectrum correction is not the deciding factor
- When the input is 2D-NMR data or raw instrument formats unsupported by MetaboLabPy's
  NMR import
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3292
tools:
- name: MetaboLabPy 0.9.66
  role: Provides the reference/baseline-based automated phase correction algorithm
    applied to the 1D-NMR spectra series
  repo: https://github.com/ludwigc/MetaboLabPy
- name: numpy
  role: Python toolbox used by MetaboLabPy for numerical processing of spectra and
    phase-angle extraction
  repo: https://github.com/ludwigc/MetaboLabPy
- name: pandas
  role: Python toolbox used to tabulate and export per-spectrum phase angles
  repo: https://github.com/ludwigc/MetaboLabPy
- name: TopSpin
  role: Comparison tool whose apk0.noe algorithm is benchmarked against the baseline-based
    correction for phase consistency
- name: MetaboLab
  role: Comparison tool whose apk5 algorithm fails on the metabolite mix spectra during
    the benchmark
- name: MNova
  role: Comparison tool used to phase the series manually for the consistency benchmark
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:33e6f54e316a131ad4d916600487f7187eacd2e527471cd6cbb4e31b4043033f
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
yes_rate: 0.9166666666666666
n_claims: 12
---

# reference-baseline-based-phase-correction

## Summary

Applies MetaboLabPy's reference/baseline-based automated phase correction, which uses baseline information rather than purely per-spectrum optimization to phase 1D-NMR spectra. In the benchmark of automatic phase correction algorithms, this baseline-based algorithm gave the most consistent (lowest-dispersion) phases across metabolite mix replicate series and Daphnia extract CryoProbe spectra.

## When to use

Use when processing a series of 1D-NMR metabolomics spectra (e.g. replicate mixtures or extract series in MetaboLabPy 0.9.66) where within-series phase consistency across replicates matters, or when comparing against apk0.noe/apk5/MNova phasing.

## When NOT to use

- When the goal is isotopomer/tracer analysis (metabolabpytools HSQC + GC-MS neural-network workflow) rather than 1D-NMR phasing
- When only a single spectrum needs phasing with no replicate series — the consistency advantage over per-spectrum correction is not the deciding factor
- When the input is 2D-NMR data or raw instrument formats unsupported by MetaboLabPy's NMR import

## Inputs

- Series of 1D-NMR spectra (e.g. metabolite mix replicates, Daphnia extract CryoProbe spectra) loadable in MetaboLabPy 0.9.66
- Per-spectrum phase angles exported from comparison tools (MetaboLab apk5, TopSpin apk0.noe, MNova)

## Outputs

- Phase-corrected 1D-NMR spectra series ready for segmental alignment, spectral scaling, and variance stabilisation
- Per-spectrum phase-angle table (exportable via pandas)
- Within-series phase dispersion/consistency metrics per phasing method (e.g. Figures S4/S5 style plots)

## How to apply

Load the 1D-NMR spectrum series (metabolite mix replicates or Daphnia extract CryoProbe spectra) into MetaboLabPy 0.9.66 and run the reference/baseline-based phase correction on every spectrum in the series. The algorithm derives phase angles from baseline information rather than correcting each spectrum independently, which reduces within-series phase dispersion across replicates. Extract the per-spectrum phase angles with numpy/pandas and compute within-series phase dispersion to quantify consistency. Compare against MetaboLabPy's per-spectrum automated phase correction, MetaboLab apk5, TopSpin apk0.noe, and manual phasing in MNova; note that MetaboLab apk5 fails outright on the metabolite mix spectra. Plot within-series phase consistency per method in the style of Figures S4/S5 and export the phase-angle table for validation.

## Related tools

- **MetaboLabPy 0.9.66** (Provides the reference/baseline-based automated phase correction algorithm applied to the 1D-NMR spectra series) — https://github.com/ludwigc/MetaboLabPy
- **numpy** (Python toolbox used by MetaboLabPy for numerical processing of spectra and phase-angle extraction) — https://github.com/ludwigc/MetaboLabPy
- **pandas** (Python toolbox used to tabulate and export per-spectrum phase angles) — https://github.com/ludwigc/MetaboLabPy
- **TopSpin** (Comparison tool whose apk0.noe algorithm is benchmarked against the baseline-based correction for phase consistency)
- **MetaboLab** (Comparison tool whose apk5 algorithm fails on the metabolite mix spectra during the benchmark)
- **MNova** (Comparison tool used to phase the series manually for the consistency benchmark)

## Examples

```
metabolabpy  # then load the 1D-NMR series and apply the baseline-based phase correction to every spectrum before exporting phase angles
```

## Evaluation signals

- Within-series phase dispersion across replicates is lowest (most consistent) for the baseline-based correction compared with apk0.noe, apk5, MNova, and per-spectrum MetaboLabPy correction
- MetaboLab apk5 fails (cannot perform automatic phase correction) on the metabolite mix spectra, confirming the benchmark setup
- MetaboLabPy baseline-based phasing yields a slightly better baseline than TopSpin apk0.noe
- Per-spectrum phase-angle table exports cleanly via pandas and matches the plotted consistency figures (S4/S5 style)

## Limitations

- MetaboLab's apk5 algorithm could not perform an automatic phase correction on the metabolite mix spectra, so it contributes no phase angles for that series
- Consistency results are specific to the benchmark datasets (metabolite mix replicates and Daphnia extract CryoProbe 1D-NMR series) and MetaboLabPy version 0.9.66

## Evidence

- [results] baseline-based algorithm most consistent: [evidence span withheld — anchor status: not_found]
- [results] apk5 failure on metabolite mix: [evidence span withheld — anchor status: not_found]
- [abstract] phase correction in workflow: "This includes a choice of algorithms for automated phase correction, segmental alignment, spectral scaling, variance stabilisation, export to various software platforms"
- [abstract] software and toolbox stack: "The software is implemented in Python and uses numerous Python toolboxes, such as numpy, scipy, pandas, etc"
