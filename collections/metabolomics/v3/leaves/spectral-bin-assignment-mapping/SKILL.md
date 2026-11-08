---
name: spectral-bin-assignment-mapping
description: Use when you have calibrated 1D 1H NMR serum/plasma spectra from a Bruker
  IVDr 600 MHz spectrometer and need to map 24 predefined spin-system bins to metabolite
  signals in a 22-metabolite panel.
when_to_use_negative:
- Input spectra are not calibrated 1D 1H NMR data from a Bruker IVDr 600 MHz instrument
  (e.g. 2D-Jres or CPMG-only datasets without SMolESY preprocessing).
- The target metabolite list extends beyond the 22 metabolites validated in this algorithm,
  since bins were defined for exactly 24 spin systems from a plasma/serum cohort.
- You already have a feature/intensity table and need statistical modelling (e.g.
  PCA of relative concentration values) rather than per-spectrum spin-system assignment.
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0003
tools:
- name: SMolESY-select
  role: Performs the pattern search within predefined spectral windows to assign 24
    1H NMR spin systems to metabolite signals
  repo: https://github.com/pantakis/SMolESY-select
- name: SMolESY
  role: Provides the processed spectrum (resolution-enhanced, macromolecule-suppressed)
    that the assignment module searches over
  repo: https://github.com/pantakis/SMolESY
- name: Bruker IVDr 600 MHz spectrometer
  role: Acquires the raw 1D 1H NMR serum/plasma spectra that are calibrated and SMolESY-processed
    before assignment
- name: STOCSY
  role: Used to check overlap of each spin system's SMolESY pattern with other SM
    signals during bin refinement
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:b2b68f45ea78415b12b4432686d79f9f63462799b25aaf65e030bdb8775ecf6a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pantakis/SMolESY-select
  tools:
  - Bruker IVDr 600 MHz spectrometer
  - SMolESY
  - SMolESY-select
  - STOCSY
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0003
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spectral-bin-assignment-mapping

## Summary

Assigns 1H NMR spin-system patterns to metabolite signals by searching the SMolESY-processed spectrum within predefined spectral windows (bins) for 24 1H NMR spin systems, enabling direct integration of assigned components without complex deconvolution or baseline fitting.

## When to use

Use when you have calibrated 1D 1H NMR serum/plasma spectra from a Bruker IVDr 600 MHz spectrometer and need to map 24 predefined spin-system bins to metabolite signals in a 22-metabolite panel.

## When NOT to use

- Input spectra are not calibrated 1D 1H NMR data from a Bruker IVDr 600 MHz instrument (e.g. 2D-Jres or CPMG-only datasets without SMolESY preprocessing).
- The target metabolite list extends beyond the 22 metabolites validated in this algorithm, since bins were defined for exactly 24 spin systems from a plasma/serum cohort.
- You already have a feature/intensity table and need statistical modelling (e.g. PCA of relative concentration values) rather than per-spectrum spin-system assignment.

## Inputs

- Calibrated 1D 1H NMR serum/plasma spectrum (SMolESY-processed)
- Predefined spectral windows (bins) for 24 1H NMR spin systems
- 22-metabolite target panel

## Outputs

- assignments.csv: 24 rows, one per spin-system bin, with bin identifier, metabolite signal, assignment status, detected ppm position

## How to apply

Load the calibrated 1D 1H NMR spectrum (calibrated to the anomeric proton of glucose at 5.233 ppm) and the 24 spin-system bin definitions derived from a 4023-sample plasma/serum cohort (3023 plasma, 1000 serum). For each bin, extract the SMolESY spectrum segment spanning the bin's spectral window and perform a pattern search, matching the expected spin-system multiplet pattern against the observed data using the SMolESY Δv1/2 linewidth range (0.0009–0.0011 ppm at 600 MHz) as the shape reference. Where a spin system's pattern is found, record the assignment linking the bin to the metabolite signal and its detected δ value; emit one row per spin-system bin (24 rows) with bin identifier, metabolite signal, assignment status, and detected ppm position. SMolESY-select aligns each assigned spin system to a random ppm value within its predefined spectral window to ease visual detection of misassignment. After assignment, direct integration of assigned spectral components replaces complex signal deconvolution or baseline fitting.

## Related tools

- **SMolESY-select** (Performs the pattern search within predefined spectral windows to assign 24 1H NMR spin systems to metabolite signals) — https://github.com/pantakis/SMolESY-select
- **SMolESY** (Provides the processed spectrum (resolution-enhanced, macromolecule-suppressed) that the assignment module searches over) — https://github.com/pantakis/SMolESY
- **Bruker IVDr 600 MHz spectrometer** (Acquires the raw 1D 1H NMR serum/plasma spectra that are calibrated and SMolESY-processed before assignment)
- **STOCSY** (Used to check overlap of each spin system's SMolESY pattern with other SM signals during bin refinement)

## Examples

```
python -m smolesy_select assign --input calibrated_spectrum_1d.csv --bins bins_24.json --output assignments.csv
```

## Evaluation signals

- Every output row corresponds to one of the 24 predefined spin-system bins.
- Assignment decisions reproduce the >99.5% assignment accuracy reported across the validated >8800-spectrum cohort.
- Each assigned spin system maps to a random ppm value within its corresponding predefined spectral window, easing visual detection of misassignment.
- Spot-check a subset of spectra against the 22-metabolite target panel to confirm per-bin assignments match expected metabolite signals.
- Detected ppm positions fall within the bin's spectral window boundaries (e.g. glycine/creatine/creatinine/choline/DMSO2/acetone/acetic acid bins span >0.01 ppm; narrow singlet bins span 3–5 SMolESY Δv1/2).

## Limitations

- The algorithm requires only standard one-dimensional 1H NMR spectra and cannot leverage 2D-Jres or STOCSY-derived information directly during assignment.
- Bins are fixed from a 4023-sample plasma/serum cohort; metabolite signals outside these 24 spin systems are not assigned.
- The validated >8800-spectrum dataset is not named or given an accession/repository in the article, limiting direct reproducibility of validation runs.
- GUI and usage video exist only as supplementary artifacts (Figure S9, Video S1).

## Evidence

- [results] Pattern search within bins: "the algorithm searches the SMolESY data for each spin system pattern within defined spectral windows (bins)"
- [results] Glucose calibration anchor: [evidence span withheld — anchor status: not_found]
- [results] Bin cohort definition: [evidence span withheld — anchor status: not_found]
- [results] Linewidth reference for pattern shape: "0.0009 < SMolESY Δv1/2 < 0.0011 ppm for the 600 MHz NMR instrument"
- [results] Direct integration replacing deconvolution: "enables the direct integration of the assigned spectral components, precluding the need for complex signal deconvolution or baseline fitting"
- [results] Misassignment visual aid: "aligns only each assigned spin system to a random ppm value within their corresponding predefined spectral window"
- [abstract] Accuracy metric: "yielding an assignment accuracy greater than 99.5% for all 22 metabolites"
- [abstract] Input requirement: "The approach is highly efficient, requiring only standard one-dimensional 1H NMR spectra"
