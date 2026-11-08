---
name: nmr-spectral-runtime-benchmarking
description: Use when you need to verify that a fully automated NMR metabolite-assignment
  pipeline (SMolESY-select) is fast enough for high-throughput processing of standard
  1D 1H NMR serum spectra, e.g. an external cohort like MTBLS395.
when_to_use_negative:
- The input is not a standard one-dimensional 1H NMR spectrum (e.g. 2D-Jres or pre-processed
  CPMG data), since SMolESY-select requires only standard 1D 1H NMR spectra by design
- The goal is to validate assignment accuracy or metabolite quantitation (>99.5% accuracy
  across 22 metabolites), not computational efficiency
- The input is already an aggregated timing table — you only need statistics, not
  re-benchmarking
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: SMolESY-select
  role: The algorithm whose per-spectrum runtime is benchmarked; applies an 11-point
    moving average filter and direct integration without deconvolution or baseline
    fitting
  repo: https://github.com/pantakis/SMolESY-select
- name: SMolESY
  role: Upstream spectral enhancement (resolution enhancement and macromolecular/baseline
    suppression) that produces the enhanced spectra SMolESY-select consumes
  repo: https://github.com/pantakis/SMolESY
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:83a42bf72242923f67e932880c49b39be57f4e4a9f9b8d85af804078156bed9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pantakis/SMolESY-select
  tools:
  - SMolESY
  - SMolESY-select
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# nmr-spectral-runtime-benchmarking

## Summary

Benchmarks the wall-clock per-spectrum runtime of SMolESY-select on standard 1D 1H NMR serum spectra to verify the algorithm's claim of computational lightness (<15 s per spectrum on a conventional laptop) versus minutes-long fitting/deconvolution approaches.

## When to use

Use when you need to verify that a fully automated NMR metabolite-assignment pipeline (SMolESY-select) is fast enough for high-throughput processing of standard 1D 1H NMR serum spectra, e.g. an external cohort like MTBLS395.

## When NOT to use

- The input is not a standard one-dimensional 1H NMR spectrum (e.g. 2D-Jres or pre-processed CPMG data), since SMolESY-select requires only standard 1D 1H NMR spectra by design
- The goal is to validate assignment accuracy or metabolite quantitation (>99.5% accuracy across 22 metabolites), not computational efficiency
- The input is already an aggregated timing table — you only need statistics, not re-benchmarking

## Inputs

- Standard 1D 1H NMR serum spectra (e.g. the MTBLS395 cohort from the Metabolights repository, https://www.ebi.ac.uk/metabolights)
- SMolESY-select executable from github.com/pantakis/SMolESY-select

## Outputs

- Per-spectrum wall-clock execution times
- Aggregate runtime table: mean runtime, min/max/percentiles, and fraction of spectra below 15 s

## How to apply

Download the external validation cohort (e.g. MTBLS395 serum 1D 1H NMR spectra) from the Metabolights repository and run SMolESY-select from github.com/pantakis/SMolESY-select on each spectrum on a conventional laptop, recording wall-clock execution time per spectrum. Because SMolESY-select uses only a moving average filter spanning 11-points within selected spectral windows and direct integration of assigned spin systems — precluding signal deconvolution and baseline fitting — timing should be taken without any manual preprocessing steps. Aggregate per-spectrum timings into a table reporting mean runtime, min/max/percentiles, and the fraction of spectra under 15 s. Judge success against the paper's benchmark: mean runtime below 15 s per spectrum, which should dramatically outperform fitting and deconvolution approaches that require several minutes of computation per spectral set.

## Related tools

- **SMolESY-select** (The algorithm whose per-spectrum runtime is benchmarked; applies an 11-point moving average filter and direct integration without deconvolution or baseline fitting) — https://github.com/pantakis/SMolESY-select
- **SMolESY** (Upstream spectral enhancement (resolution enhancement and macromolecular/baseline suppression) that produces the enhanced spectra SMolESY-select consumes) — https://github.com/pantakis/SMolESY

## Evaluation signals

- Mean per-spectrum runtime is below 15 s on a conventional laptop, matching the paper's reported timing benchmark
- A fraction-of-spectra-below-15-s statistic is reported alongside mean, min, max, and percentiles
- Timings were measured on standard 1D 1H NMR spectra (e.g. MTBLS395 serum spectra) acquired with Bruker IVDr 600 MHz spectrometers
- Benchmark context confirms SMolESY-select required no sample preprocessing, complex deconvolution, or spectral baseline fitting
- Reported runtimes are dramatically shorter than the several minutes per spectral set cited for fitting/deconvolution approaches

## Limitations

- No software version is stated for SMolESY-select or SMolESY, complicating exact reproduction
- The article's >8800-sample validation set is not named or given a repository accession; only MTBLS395 is explicitly identified
- Comparison to competing fitting/deconvolution software cites only qualitative timing ('several minutes') without naming the specific competing tools
- GUI and usage instructions exist only as supplementary artifacts (Figure S9, Video S1)

## Evidence

- [abstract] requiring only standard one-dimensional 1H NMR spectra and avoiding the need for sample preprocessing, complex deconvolution, and spectral baseline fitting: "The approach is highly efficient, requiring only standard one-dimensional 1H NMR spectra and avoiding the need for sample preprocessing, complex deconvolution, and spectral baseline fitting."
- [discussion] dramatically outperforming fitting and deconvolution approaches that require several minutes of computation per spectral set: "dramatically outperforming fitting and deconvolution approaches that require several minutes of computation per spectral set"
- [results] one serum cohort from the Metabolights repository (https://www.ebi.ac.uk/metabolights) (MTBLS395): "one serum cohort from the Metabolights repository (https://www.ebi.ac.uk/metabolights) (MTBLS395)"
- [results] a moving average filter spanning 11-points (see the Supporting Information) across the high resolution data of the selected windows: "a moving average filter spanning 11-points (see the Supporting Information) across the high resolution data of the selected windows"
