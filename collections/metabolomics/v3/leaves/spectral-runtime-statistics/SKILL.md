---
name: spectral-runtime-statistics
description: Use when assessing whether a fully automated NMR metabolite-quantitation
  algorithm is computationally efficient enough for high-throughput analysis of large
  serum/plasma cohorts such as MTBLS395.
when_to_use_negative:
- Input spectra are 2D (e.g. 2D-Jres) or non-standard 1D acquisitions; SMolESY-select
  requires only standard one-dimensional 1H NMR spectra.
- The goal is quantitation accuracy assessment (e.g. the >99.5% assignment accuracy
  for the 22 metabolites) rather than computational efficiency benchmarking.
- Comparing against targeted fitting/deconvolution software on quality of metabolite
  panels rather than runtime.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3292
tools:
- name: SMolESY-select
  role: Algorithm whose per-spectrum runtime is benchmarked; performs automated assignment
    and integration of 22 metabolites from 1D 1H NMR spectra
  repo: https://github.com/pantakis/SMolESY-select
- name: SMolESY
  role: Preprocessing step (spectral resolution enhancement and macromolecular signal
    suppression) that produces the SMolESY-transformed spectra consumed by SMolESY-select
  repo: https://github.com/pantakis/SMolESY
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:3fdfab0b4df915ea2aae7adce27aa64e406da53e19ed1d2f5658385f8f8eba55
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pantakis/SMolESY-select
  tools:
  - SMolESY
  - SMolESY-select
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# spectral-runtime-statistics

## Summary

Benchmarks the wall-clock runtime of the SMolESY-select algorithm on standard 1D 1H NMR serum spectra to verify it meets the paper's lightweight, high-throughput computation claim (<15 s per spectrum on a conventional laptop).

## When to use

Use when assessing whether a fully automated NMR metabolite-quantitation algorithm is computationally efficient enough for high-throughput analysis of large serum/plasma cohorts such as MTBLS395.

## When NOT to use

- Input spectra are 2D (e.g. 2D-Jres) or non-standard 1D acquisitions; SMolESY-select requires only standard one-dimensional 1H NMR spectra.
- The goal is quantitation accuracy assessment (e.g. the >99.5% assignment accuracy for the 22 metabolites) rather than computational efficiency benchmarking.
- Comparing against targeted fitting/deconvolution software on quality of metabolite panels rather than runtime.

## Inputs

- Standard 1D 1H NMR serum/plasma spectra (e.g. MTBLS395 cohort from the Metabolights repository)
- SMolESY-select executable or source from github.com/pantakis/SMolESY-select

## Outputs

- Per-spectrum wall-clock execution times
- Aggregated timing table with mean runtime, min/max/percentile distribution, and fraction of spectra below 15 s

## How to apply

Download the 1D 1H NMR serum spectra of an external cohort (e.g. MTBLS395 from the Metabolights repository at https://www.ebi.ac.uk/metabolights) and run SMolESY-select (github.com/pantakis/SMolESY-select) on each spectrum individually, recording the wall-clock execution time per spectrum. Aggregate the per-spectrum timings into a table reporting the mean runtime plus distribution statistics (min, max, percentiles) and the fraction of spectra completing below the 15 s threshold. The rationale is that SMolESY-select performs only spectral calibration to the glucose anomeric proton at 5.233 ppm, an 11-point moving average filter within selected spectral windows, pattern search within predefined bins, and direct integration of assigned components — precluding complex signal deconvolution or baseline fitting — so per-spectrum cost should stay low. Validate success by confirming the mean runtime is below 15 s per spectrum, matching the paper's reported timing benchmark and contrasting with fitting/deconvolution approaches that require several minutes of computation per spectral set.

## Related tools

- **SMolESY-select** (Algorithm whose per-spectrum runtime is benchmarked; performs automated assignment and integration of 22 metabolites from 1D 1H NMR spectra) — https://github.com/pantakis/SMolESY-select
- **SMolESY** (Preprocessing step (spectral resolution enhancement and macromolecular signal suppression) that produces the SMolESY-transformed spectra consumed by SMolESY-select) — https://github.com/pantakis/SMolESY

## Examples

```
for f in MTBLS395/*.fid; do /usr/bin/time -v SMolESY-select --input $f --output timings/$f.json; done
```

## Evaluation signals

- Mean per-spectrum runtime is below 15 s on a conventional laptop, matching the paper's reported timing benchmark.
- Timing table includes distribution statistics (min, max, percentiles) and the fraction of spectra under the 15 s threshold.
- Per-spectrum timings substantially outperform fitting and deconvolution approaches reported to require several minutes of computation per spectral set.
- Timings are recorded for every spectrum in the cohort (e.g. all MTBLS395 spectra), with no spectra silently excluded.

## Limitations

- No software version stated for SMolESY-select or SMolESY, so runtime comparisons across versions are uncontrolled.
- The comparison to competing approaches cites only qualitative timing ('several minutes') without naming specific competing software, so the benchmark baseline is not directly reproducible.
- Timing depends on the host hardware; 'conventional laptop' is not precisely defined, so absolute runtimes may vary across machines.

## Evidence

- [abstract] The approach is highly efficient, requiring only standard one-dimensional 1H NMR spectra and avoiding the need for sample preprocessing, complex deconvolution, and spectral baseline fitting.: "The approach is highly efficient, requiring only standard one-dimensional 1H NMR spectra and avoiding the need for sample preprocessing, complex deconvolution, and spectral baseline fitting."
- [results] one serum cohort from the Metabolights repository (https://www.ebi.ac.uk/metabolights) (MTBLS395): "one serum cohort from the Metabolights repository (https://www.ebi.ac.uk/metabolights) (MTBLS395)"
- [discussion] dramatically outperforming fitting and deconvolution approaches that require several minutes of computation per spectral set: "dramatically outperforming fitting and deconvolution approaches that require several minutes of computation per spectral set"
- [discussion] it is freely available to download at https://github.com/pantakis/SMolESY-select: "it is freely available to download at https://github.com/pantakis/SMolESY-select"
