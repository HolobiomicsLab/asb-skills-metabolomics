---
name: large-cohort-spectra-processing
description: Use when a cohort of standard 1D 1H NMR serum/plasma spectra (hundreds
  to thousands, e.g. MTBLS395) must be converted to metabolite panel measurements
  under a tight per-spectrum time budget.
when_to_use_negative:
- The input is not a standard 1D 1H NMR spectrum (e.g. only 2D-Jres or CPMG data is
  available, or preprocessing such as deconvolution/baseline fitting is itself the
  goal)
- A targeted metabolite fitting/deconvolution workflow is explicitly required rather
  than fast direct integration of assigned spin systems
- Only a handful of spectra need processing and a heavyweight per-spectrum fitting
  approach's multi-minute runtime is acceptable
edam_operation: http://edamontology.org/operation_3214
edam_topics: []
tools:
- name: SMolESY-select
  role: Fully automated algorithm that assigns and directly integrates the 22 metabolite
    spin systems from 1D 1H NMR spectra using an 11-point moving average filter and
    predefined spectral bins
  repo: https://github.com/pantakis/SMolESY-select
- name: SMolESY
  role: Preprocessing transformation providing baseline suppression and resolution
    enhancement that SMolESY-select consumes
  repo: https://github.com/pantakis/SMolESY
- name: Bruker IVDr 600 MHz spectrometer
  role: Acquisition instrument producing the standard 1D 1H NMR spectra the pipeline
    expects
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:071a4d36964a5e13e0589f9ecae4bcb907b2c26f3760964be0fcba023122ce80
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pantakis/SMolESY-select
  tools:
  - Bruker IVDr 600 MHz spectrometer
  - SMolESY
  - SMolESY-select
  edam_operation: http://edamontology.org/operation_3214
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# large-cohort-spectra-processing

## Summary

A computationally lightweight processing route for large cohorts of standard 1D 1H NMR serum/plasma spectra (e.g. MTBLS395) that assigns and integrates 22 metabolite spin systems directly, precluding complex deconvolution or baseline fitting and running in <15 s per spectrum on a conventional laptop.

## When to use

Use when a cohort of standard 1D 1H NMR serum/plasma spectra (hundreds to thousands, e.g. MTBLS395) must be converted to metabolite panel measurements under a tight per-spectrum time budget.

## When NOT to use

- The input is not a standard 1D 1H NMR spectrum (e.g. only 2D-Jres or CPMG data is available, or preprocessing such as deconvolution/baseline fitting is itself the goal)
- A targeted metabolite fitting/deconvolution workflow is explicitly required rather than fast direct integration of assigned spin systems
- Only a handful of spectra need processing and a heavyweight per-spectrum fitting approach's multi-minute runtime is acceptable

## Inputs

- Standard 1D 1H NMR serum/plasma spectra (e.g. Bruker IVDr 600 MHz acquisition format)
- Public cohort data such as MTBLS395 downloaded from https://www.ebi.ac.uk/metabolights
- Spectra calibrated to the glucose anomeric proton at 5.233 ppm

## Outputs

- Assigned and directly integrated measurements for the 22-metabolite panel per spectrum
- Misassignment-detectable plots with each spin system aligned to a random ppm value within its predefined spectral window
- Per-spectrum wall-clock timing table (mean, min/max/percentiles, fraction below 15 s)

## How to apply

Acquire the raw 1D 1H NMR spectra (600 MHz Bruker IVDr acquisition for in-house data, or download public cohorts such as MTBLS395 from the Metabolights repository), then automatically calibrate every spectrum to the glucose anomeric proton signal at 5.233 ppm. Apply the SMolESY transformation for baseline suppression and resolution enhancement, and run SMolESY-select, which uses a moving average filter spanning 11 points within selected spectral windows to recover low-abundance metabolite signals (tyrosine, phenylalanine, histidine, creatinine, choline). The algorithm searches predefined spectral windows (bins) for each of the 24 1H NMR spin system patterns, correlating singlet δ values with abundant metabolites to narrow bins to 3–5 SMolESY Δv1/2 where needed, then performs direct integration of the assigned components — no deconvolution or baseline fitting is required. Record per-spectrum wall-clock time and aggregate into mean/min/max/percentile statistics to benchmark against the <15 s/spectrum target reported by the paper.

## Related tools

- **SMolESY-select** (Fully automated algorithm that assigns and directly integrates the 22 metabolite spin systems from 1D 1H NMR spectra using an 11-point moving average filter and predefined spectral bins) — https://github.com/pantakis/SMolESY-select
- **SMolESY** (Preprocessing transformation providing baseline suppression and resolution enhancement that SMolESY-select consumes) — https://github.com/pantakis/SMolESY
- **Bruker IVDr 600 MHz spectrometer** (Acquisition instrument producing the standard 1D 1H NMR spectra the pipeline expects)

## Examples

```
git clone https://github.com/pantakis/SMolESY-select
```

## Evaluation signals

- Mean per-spectrum wall-clock runtime is below 15 s, matching the paper's timing benchmark (compare against the several-minutes-per-spectrum cost of fitting/deconvolution approaches)
- All 22 metabolites achieve assignment accuracy consistent with the reported >99.5% on large cohorts
- Denoised SMolESY signals keep Δv1/2 slightly narrower than standard 1D 1H NMR with s/n at most ~10% less than CPMG spectra
- Every spectrum is successfully calibrated to the glucose anomeric proton at 5.233 ppm
- Random-ppm alignment plot of assigned spin systems shows no visually detectable misassignments within predefined spectral windows

## Limitations

- No software version is stated for SMolESY-select or SMolESY, complicating exact reproduction
- The >8800-sample validation dataset is not named or given an accession/repository; only the MTBLS395 serum cohort is identified
- The paper cites only qualitative timing for competing fitting/deconvolution software ('several minutes of computation per spectral set') without naming the specific tools
- GUI and usage video exist only as supplementary artifacts (Figure S9, Video S1) rather than documented command-line interfaces

## Evidence

- [abstract] The approach is highly efficient, requiring only standard one-dimensional 1H NMR spectra and avoiding the need for sample preprocessing, complex deconvolution, and spectral baseline fitting.: "The approach is highly efficient, requiring only standard one-dimensional 1H NMR spectra and avoiding the need for sample preprocessing, complex deconvolution, and spectral baseline fitting."
- [results] SMolESY-select utilizes a moving average filter spanning 11-points: "SMolESY-select utilizes a moving average filter spanning 11-points"
- [results] enables the direct integration of the assigned spectral components, precluding the need for complex signal deconvolution or baseline fitting: "enables the direct integration of the assigned spectral components, precluding the need for complex signal deconvolution or baseline fitting"
- [discussion] dramatically outperforming fitting and deconvolution approaches that require several minutes of computation per spectral set: "dramatically outperforming fitting and deconvolution approaches that require several minutes of computation per spectral set"
- [results] one serum cohort from the Metabolights repository (https://www.ebi.ac.uk/metabolights) (MTBLS395): "one serum cohort from the Metabolights repository (https://www.ebi.ac.uk/metabolights) (MTBLS395)"
