---
name: nmr-coupling-constant-calculation
description: Use when a one-dimensional 1H NMR spectrum (JCAMP-DX) has been loaded
  and peak-picked, and per-peak coupling constants and multiplicity labels must be
  derived automatically from the splitting pattern.
when_to_use_negative:
- The input spectrum is 13C NMR (typically decoupled, no interpretable splitting),
  IR, or MS data rather than 1H NMR — ChemSpectra treats these types separately and
  coupling/multiplicity logic is 1H-specific.
- The data are 2D NMR spectra — the paper explicitly lists 2D viewing and analysis
  as a missing feature for future improvement.
- Multiplicities have already been manually assigned by the user and only display/integration
  is needed (react-spectra-editor supports manual editing without recalculation).
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: ChemSpectra
  role: Backend web service that performs the NMR-specific coupling constant calculation
    and multiplicity assignment on 1H spectra
  repo: github.com/ComPlat/chem-spectra-app
- name: nmrglue
  role: Python library used to read JCAMP-DX blocks and select the recognized 1H spectrum
    block in the analysis workflow
- name: react-spectra-editor
  role: Frontend editor that displays the 1H spectrum, applies/visualizes peak-picking
    and the resulting multiplicity assignments
  repo: https://github.com/ComPlat/react-spectra-editor
- name: SciPy
  role: Python scientific library used for peak detection/analysis steps in the workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-020-00481-0
    title: 'ChemSpectra: a web-based spectra editor for analytical data'
  source_content_hash: sha256:4f84217afb0c414235bdcc203f59685f9ee6de472bb0bd8ba84b2edcae31a721
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemSpectra
  - Python
  - SciPy
  - nmrglue
  - react-spectra-editor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# nmr-coupling-constant-calculation

## Summary

Automatic calculation of 1H NMR coupling constants and assignment of multiplicity labels (s, d, t, q, quint, m) from picked peak splitting patterns, as implemented in ChemSpectra's NMR analysis workflow, where multiplicities are inferred from known libraries and validated by additional rules.

## When to use

Use when a one-dimensional 1H NMR spectrum (JCAMP-DX) has been loaded and peak-picked, and per-peak coupling constants and multiplicity labels must be derived automatically from the splitting pattern.

## When NOT to use

- The input spectrum is 13C NMR (typically decoupled, no interpretable splitting), IR, or MS data rather than 1H NMR — ChemSpectra treats these types separately and coupling/multiplicity logic is 1H-specific.
- The data are 2D NMR spectra — the paper explicitly lists 2D viewing and analysis as a missing feature for future improvement.
- Multiplicities have already been manually assigned by the user and only display/integration is needed (react-spectra-editor supports manual editing without recalculation).

## Inputs

- One-dimensional 1H NMR spectrum in JCAMP-DX format (produced/normalized by chem-spectra-app, which converts non-JCAMP-DX inputs to JCAMP-DX first)
- Peak list from automatic peak detection (default or manually defined threshold) with x/y peak positions

## Outputs

- Peak-to-multiplicity assignment table with per-peak position (x), intensity (y), coupling constant, and multiplicity label (e.g. multiplicity_assignments.csv)
- Per-peak coupling constant values (J) derived from the splitting pattern

## How to apply

Load the 1H NMR spectrum from its JCAMP-DX file (e.g. with nmrglue, selecting the recognized 1H spectrum block via the data_type mapping) and apply automatic peak detection using the default or a manually defined threshold (thres) to pick peaks with x/y positions. For each picked peak region, compute the coupling constant(s) from the spacing of the neighbouring lines of the splitting pattern. Assign the multiplicity label from the inferred number of coupling partners, using ChemSpectra's approach in which multiplicities are inferred from known libraries and then checked by additional rules to ensure correctness. Assemble peak position (x), intensity (y), coupling constant and multiplicity label into a peak-to-multiplicity assignment table. ChemSpectra's NMR-specific functions are designed to complement reference signal definition and signal integration, so run this after peak-picking alongside those steps.

## Related tools

- **ChemSpectra** (Backend web service that performs the NMR-specific coupling constant calculation and multiplicity assignment on 1H spectra) — github.com/ComPlat/chem-spectra-app
- **nmrglue** (Python library used to read JCAMP-DX blocks and select the recognized 1H spectrum block in the analysis workflow)
- **react-spectra-editor** (Frontend editor that displays the 1H spectrum, applies/visualizes peak-picking and the resulting multiplicity assignments) — https://github.com/ComPlat/react-spectra-editor
- **SciPy** (Python scientific library used for peak detection/analysis steps in the workflow)

## Examples

```
import nmrglue as ng; dic, data = ng.jcampdx.read('1H_spectrum.jdx')  # then peak-pick with threshold and compute J from line splitting
```

## Evaluation signals

- The output assignment table (e.g. multiplicity_assignments.csv) is valid CSV with exactly one row per picked peak, each row carrying x, y, coupling constant and multiplicity fields.
- All multiplicity labels fall within the standard set {s, d, t, q, quint, m}.
- Each computed coupling constant is consistent with the measured spacing of the neighbouring lines in the peak's splitting pattern.
- Inferred multiplicity assignments survive ChemSpectra's additional rule checks (library-inferred labels validated against splitting rules).
- Peak list used for assignment matches the peaks obtained at the defined detection threshold (default or manual thres).

## Limitations

- Multiplicity inference depends on known libraries plus rule checks; unusual or second-order splitting patterns may not be captured correctly.
- Restricted to one-dimensional 1H (and 13C) NMR; functions for viewing and analysis of 2D data are not yet implemented.
- Processing of FID files lacks advanced phase correction and baseline correction methods, which can degrade upstream peak picking.
- Non-JCAMP-DX inputs must first be converted to JCAMP-DX for standardized treatment before analysis.

## Evidence

- [abstract] NMR-specific functions include reference signal definition, signal integration, coupling constant calculation and multiplicity assignment: "NMR specific functions include the definition of a reference signal, the integration of signals, coupling constant calculation and multiplicity assignment"
- [discussion] Automatic peak detection threshold: "the automatic peak detection according to a default or manually defined threshold"
- [intro] Objective: advanced editors with coupling constant calculation: "advanced spectra editors for the visualization but also for the analysis of spectroscopic data such as NMR, MS and IR data with peak-picking, NMR signal integration, coupling"
- [readme] chem-spectra-app handles JCAMP-DX NMR spectra: "This backend web service processes spectra -- NMR, IR, UV/VIS, MS, LC/MS, XRD, CV and more -- from JCAMP-DX, Bruker, BagIt, RAW and mzML files."
