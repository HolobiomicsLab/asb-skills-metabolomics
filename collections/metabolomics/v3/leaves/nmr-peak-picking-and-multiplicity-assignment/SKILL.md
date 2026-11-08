---
name: nmr-peak-picking-and-multiplicity-assignment
description: Use when a series of 1D or pseudo-2D NMR spectra (1H, 13C, 31P, 15N)
  must be quantified and each signal of interest needs peak positions and multiplicity
  defined before multiplet-model fitting, especially with overlapping peaks.
when_to_use_negative:
- The data are not spectra (or spectra series) that can be modeled as multiplets with
  baseline correction — e.g. already-integrated peak tables or pre-quantified concentrations
- A fully automated, no-visual-inspection quantification pipeline is required, since
  MultiNMRFit prescribes semi-automated peak picking and visual validation of fitted
  curves
- The signals of interest require spectral deconvolution beyond multiplet models with
  peak overlaps and zero-order baseline correction as implemented in MultiNMRFit
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: MultiNMRFit
  role: Provides the semi-automated peak picking and multiplicity definition step,
    plus the built-in multiplet signal models fitted to each spectrum in the series
  repo: https://github.com/NMRTeamTBI/MultiNMRFit/
- name: pip
  role: Installation channel for multinmrfit (pip install multinmrfit), requiring
    Python 3.8 or higher
- name: Python
  role: Runtime for MultiNMRFit, which can also be used as a Python library instead
    of the GUI
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf463
    title: 'MultiNMRFit: a software to fit 1D and pseudo-2D NMR spectra'
  source_content_hash: sha256:d89e8d983fd78195e33729e34518add753e6f598ea9b2136bbe6aefc0e27349f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMRTeamTBI/MultiNMRFit
  tools:
  - MultiNMRFit
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# nmr-peak-picking-and-multiplicity-assignment

## Summary

Semi-automated peak picking and multiplicity definition in MultiNMRFit: identify resonance positions and assign multiplet signal models (e.g. doublets of triplets for U-13C-labeled GLC/G6P, overlapping ADP/ATP multiplets) prior to fitting series of 1D or pseudo-2D NMR spectra.

## When to use

Use when a series of 1D or pseudo-2D NMR spectra (1H, 13C, 31P, 15N) must be quantified and each signal of interest needs peak positions and multiplicity defined before multiplet-model fitting, especially with overlapping peaks.

## When NOT to use

- The data are not spectra (or spectra series) that can be modeled as multiplets with baseline correction — e.g. already-integrated peak tables or pre-quantified concentrations
- A fully automated, no-visual-inspection quantification pipeline is required, since MultiNMRFit prescribes semi-automated peak picking and visual validation of fitted curves
- The signals of interest require spectral deconvolution beyond multiplet models with peak overlaps and zero-order baseline correction as implemented in MultiNMRFit

## Inputs

- Series of 1D NMR spectra acquired individually, as a pseudo-2D experiment, or as tabulated text files
- 1H pseudo-2D glycolysis dataset (1024 spectra, metabolites GLC, G6P, F6P, FBP, ATP, ADP)
- 31P spectra series acquired simultaneously during the glycolysis assay
- 13C-detected pseudo-2D spectra of uniformly 13C-labeled glucose and glucose-6-phosphate

## Outputs

- Peak-picked resonance positions and multiplicity definitions for each signal of interest
- Assigned multiplet signal models (e.g. doublet of triplets) per signal per spectrum
- Fitted parameters (intensity, area, chemical shift, linewidth, coupling constants) with standard-deviation uncertainties, exported as CSV (e.g. 31p_fit_results.csv)
- Fitted-curve overlay plots for visual inspection (e.g. 31p_fitted_spectra.png)

## How to apply

Load the spectra series (acquired as pseudo-2D experiments or tabulated text files) into MultiNMRFit and launch the nmrfit graphical interface, which supports semi-automated peak picking and definition of multiplicity for each signal of interest. Assign built-in multiplet signal models matched to the observed splitting pattern — for example, doublet-of-triplets models for uniformly 13C-labeled glucose and G6P signals, or distinct multiplet components for the overlapping ADP and ATP peaks in glycolysis 1H time-courses. Decide which resonances to model per nucleus: the workflow demonstrates this on 1H (GLC, G6P, F6P, FBP, ATP, ADP), 31P resonances detected simultaneously during the assay, and 13C-detected labeled reactants. The defined models are then fitted with zero-order baseline correction and explicit overlap handling, so correct multiplicity assignment at this stage directly determines fit convergence and the accuracy of extracted intensity, area, chemical shift, linewidth, and coupling constants. Validate assignments by visually inspecting the fitted curves overlaid on the experimental spectra before accepting parameter estimates and their standard-deviation uncertainties.

## Related tools

- **MultiNMRFit** (Provides the semi-automated peak picking and multiplicity definition step, plus the built-in multiplet signal models fitted to each spectrum in the series) — https://github.com/NMRTeamTBI/MultiNMRFit/
- **pip** (Installation channel for multinmrfit (pip install multinmrfit), requiring Python 3.8 or higher)
- **Python** (Runtime for MultiNMRFit, which can also be used as a Python library instead of the GUI)

## Examples

```
pip install multinmrfit; nmrfit  # then load the pseudo-2D spectra series and run semi-automated peak picking and multiplicity definition in the GUI
```

## Evaluation signals

- Visual inspection confirms fitted curves overlaid on experimental spectra reproduce each multiplet at every time point
- Overlapping ADP and ATP multiplets are correctly captured by the assigned models in the 1024-spectrum 1H series
- Fitted parameters (intensity, area, chemical shift, linewidth, coupling constants) converge and are reported with standard-deviation uncertainties of reasonable magnitude
- In the E. coli acetate labeling application, total acetate remains constant at 8.5 ± 0.1 mM across the series while 2-13C1-acetate declines from 8.5 to 6.5 mM
- Fitting the full 1024-spectrum series completes within the reported ~3-hour throughput

## Limitations

- The tool prescribes visual inspection of fitted curves as the acceptance criterion; no fully automated validation is provided
- The discussion section contains no text, so limitations, missing artifacts, or reproducibility gaps could not be extracted from the article
- Overlap handling is limited to peak overlaps with zero-order baseline correction within the built-in multiplet signal models

## Evidence

- [readme] semi-automated analysis: "**semi-automated analysis** for **peak picking** and **definition of multiplicity** for each signal"
- [abstract] abstract scope: "We demonstrate the robustness and flexibility of MultiNMRFit on 1H, 13C, and 31P NMR datasets collected in metabolomics and isotope labeling studies."
- [abstract] parameter extraction: "MultiNMRFit extracts signal parameters (e.g. intensity, area, chemical shift, and coupling constants) by fitting the experimental spectra using built-in or user-defined signal models"
- [results] validation criterion: [evidence span withheld — anchor status: not_found]
- [readme] overlap and baseline handling: "account for **overlaps** between peaks and **zero-order baseline correction**"
