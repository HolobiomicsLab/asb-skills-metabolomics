---
name: nmr-spectral-recalibration-glucose-anomeric-proton
description: Use when raw 1D 1H NMR spectra from Bruker IVDr 600 MHz serum/plasma
  cohorts show ppm drift that must be corrected before metabolite assignment, bin-based
  integration, or cross-sample comparison.
when_to_use_negative:
- When spectra are already internally referenced to an alternative compound's chemical
  shift (e.g. TSP/DSS-anchored spectra), as this skill anchors exclusively to the
  glucose anomeric proton at 5.233 ppm.
- When the analytical goal is 2D NMR (e.g. 2D-Jres) quantitation or a workflow that
  requires extensive manual baseline correction before any calibration.
edam_topics: []
tools:
- name: SMolESY-select
  role: Runs the automated glucose-anomeric-proton recalibration as its first pipeline
    step before bin-based assignment and direct integration of 22 metabolite spin
    systems.
  repo: https://github.com/pantakis/SMolESY-select
- name: SMolESY
  role: Consumes the recalibrated spectra downstream to perform baseline suppression,
    resolution enhancement, and macromolecular signal suppression on the 1D 1H NMR
    data.
  repo: https://github.com/pantakis/SMolESY
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:03490279aecc616e0d5d928af8c673205d3f7fcf28c8d019ced1a5ee566f7805
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pantakis/SMolESY-select
  tools:
  - 2D-Jres
  - SMolESY
  - SMolESY-select
  - STOCSY
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# nmr-spectral-recalibration-glucose-anomeric-proton

## Summary

Automatically aligns 1D 1H NMR serum/plasma spectra by shifting each chemical shift scale so the glucose anomeric proton signal locks to 5.233 ppm, the calibration anchor used by SMolESY-select before bin definition and integration.

## When to use

Use when raw 1D 1H NMR spectra from Bruker IVDr 600 MHz serum/plasma cohorts show ppm drift that must be corrected before metabolite assignment, bin-based integration, or cross-sample comparison.

## When NOT to use

- When spectra are already internally referenced to an alternative compound's chemical shift (e.g. TSP/DSS-anchored spectra), as this skill anchors exclusively to the glucose anomeric proton at 5.233 ppm.
- When the analytical goal is 2D NMR (e.g. 2D-Jres) quantitation or a workflow that requires extensive manual baseline correction before any calibration.

## Inputs

- Raw 1D 1H NMR serum or plasma spectra (e.g. Bruker IVDr 600 MHz data from cohort MTBLS395)

## Outputs

- Chemical-shift-calibrated 1D 1H NMR spectra with glucose anomeric proton aligned at 5.233 ppm, ready for SMolESY processing and metabolite integration

## How to apply

After acquiring spectra on a Bruker IVDr 600 MHz instrument, automatically shift every spectrum's chemical shift scale so the anomeric proton signal of glucose is fixed at 5.233 ppm; this calibration is the first step of the SMolESY-select workflow and precedes bin-based pattern searching and direct integration of the 22 targeted metabolite spin systems. The recalibrated spectra then serve as the substrate for SMolESY resolution enhancement (with baseline suppression and macromolecular signal suppression), 11-point moving-average smoothing within predefined spectral windows, and direct integration without deconvolution or baseline fitting. Calibration is applied to standard 1D 1H NMR data only — no CPMG or 2D-Jres preprocessing is needed for this step — and it guarantees that the SMolESY Δv1/2 bin definitions (≥10 Δv1/2 for glycine, creatine/creatinine, choline, DMSO2, acetone, acetic acid; 3–5 Δv1/2 for correlating singlets vs lactic acid/alanine) remain valid across all samples in a cohort such as MTBLS395.

## Related tools

- **SMolESY-select** (Runs the automated glucose-anomeric-proton recalibration as its first pipeline step before bin-based assignment and direct integration of 22 metabolite spin systems.) — https://github.com/pantakis/SMolESY-select
- **SMolESY** (Consumes the recalibrated spectra downstream to perform baseline suppression, resolution enhancement, and macromolecular signal suppression on the 1D 1H NMR data.) — https://github.com/pantakis/SMolESY

## Examples

```
SMolESY-select calibrates automatically; no standalone CLI is documented. Example: run the released GUI or invoke the pipeline on Bruker IVDr data such that glucose anomeric proton is set to 5.233 ppm before SMolESY processing.
```

## Evaluation signals

- After recalibration, the glucose anomeric proton resonance is located at 5.233 ppm in every sample.
- The calibrated spectra align with the predefined SMolESY bin windows such that spin-system pattern searches across the full MTBLS395 cohort succeed without manual intervention.
- Post-calibration direct integration yields a metabolite table with exactly 22 rows × all cohort samples.
- Assignment accuracy on a subset of recalibrated spectra matches the paper's reported >99.5% benchmark.

## Limitations

- The approach requires standard 1D 1H NMR spectra only; it is not validated for CPMG or 2D data.
- Calibration accuracy depends on a sufficiently intense glucose anomeric proton signal; low-glucose or heavily abnormal samples may shift the anchor.
- SMolESY-select's GUI and usage video (Figure S9, Video S1) are supplementary artifacts, so calibration is documented only in the article/SI rather than a versioned software release.

## Evidence

- [results] Calibration of all spectra to the 1H NMR anomeric proton signal of glucose at 5.233 ppm: [evidence span withheld — anchor status: not_found]
- [results] all spectra are automatically calibrated to the 1H NMR signals of glucose: "all spectra are automatically calibrated to the 1H NMR signals of glucose"
- [results] the established commercial standard approach, Bruker IVDr: "the established commercial standard approach, Bruker IVDr"
- [abstract] The approach is highly efficient, requiring only standard one-dimensional 1H NMR spectra and avoiding the need for sample preprocessing, complex deconvolution, and spectral baseline fitting.: "The approach is highly efficient, requiring only standard one-dimensional 1H NMR spectra and avoiding the need for sample preprocessing, complex deconvolution, and spectral baseline fitting."
- [results] one serum cohort from the Metabolights repository (https://www.ebi.ac.uk/metabolights) (MTBLS395): "one serum cohort from the Metabolights repository (https://www.ebi.ac.uk/metabolights) (MTBLS395)"
