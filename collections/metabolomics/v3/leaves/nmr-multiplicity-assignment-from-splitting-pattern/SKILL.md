---
name: nmr-multiplicity-assignment-from-splitting-pattern
description: Use when processing 1D 1H NMR spectra (e.g. JCAMP-DX from ChemSpectra/Chemotion
  ELN) and you need automatic peak picking plus coupling constant calculation and
  multiplicity assignment for the detected signals.
when_to_use_negative:
- The input is 2D NMR data — the paper explicitly lists viewing and analysis of 2D
  data as future work not yet implemented in ChemSpectra
- The input is an IR or MS spectrum (including mzML/RAW-converted files) — coupling
  constant calculation and multiplicity assignment are NMR-specific functions only
- The spectrum requires advanced phase or baseline correction before peak picking
  — the paper notes FID processing still lacks advanced phase/baseline correction
  methods
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: ChemSpectra (chem-spectra-app)
  role: Backend service that converts inputs to JCAMP-DX, performs threshold-based
    automatic peak detection, coupling constant calculation and library-based multiplicity
    assignment with rule-based checking
  repo: https://github.com/ComPlat/chem-spectra-app
- name: react-spectra-editor
  role: Front-end editor (jcampconverter library) that displays the 1H spectrum and
    allows manual correction of automatically inferred multiplicities
  repo: https://github.com/ComPlat/react-spectra-editor
- name: Chemotion ELN
  role: Main client that receives the automatically detected or manually chosen signals
    via the automated transfer procedure
  repo: https://github.com/ComPlat/chemotion_ELN
- name: nmrglue
  role: Python library used to read JCAMP-DX blocks when scripting the workflow outside
    the web app
  repo: https://github.com/ComPlat/chem-spectra-app
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-020-00481-0
    title: 'ChemSpectra: a web-based spectra editor for analytical data'
  source_content_hash: sha256:53607a07173b52bf1eb809077e6c429298469ee0e233742897af73ba62324c47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ComPlat/chem-spectra-app
  tools:
  - ChemSpectra
  - ChemSpectra (chem-spectra-app)
  - Chemotion ELN
  - Python
  - SciPy
  - nmrglue
  - react-spectra-editor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# nmr-multiplicity-assignment-from-splitting-pattern

## Summary

Automatic assignment of 1H NMR multiplicities and coupling constants: peak positions are picked from a JCAMP-DX spectrum with a threshold-based peak picker, the splitting pattern of neighbouring lines is used to calculate J-couplings, and multiplicity labels (s, d, t, q, quint, m) are inferred from known libraries and validated by additional rules.

## When to use

Use when processing 1D 1H NMR spectra (e.g. JCAMP-DX from ChemSpectra/Chemotion ELN) and you need automatic peak picking plus coupling constant calculation and multiplicity assignment for the detected signals.

## When NOT to use

- The input is 2D NMR data — the paper explicitly lists viewing and analysis of 2D data as future work not yet implemented in ChemSpectra
- The input is an IR or MS spectrum (including mzML/RAW-converted files) — coupling constant calculation and multiplicity assignment are NMR-specific functions only
- The spectrum requires advanced phase or baseline correction before peak picking — the paper notes FID processing still lacks advanced phase/baseline correction methods

## Inputs

- 1H NMR spectrum in JCAMP-DX format (or Bruker FID/other formats converted to JCAMP-DX by chem-spectra-app)
- Peak-detection threshold (default or user-defined thres value)

## Outputs

- Peak table with position (x), intensity (y), coupling constant (J) and multiplicity label (s, d, t, q, quint, m)
- Signals transferred to the Chemotion ELN / repository via the automated signal-transfer workflow

## How to apply

Load the 1H NMR spectrum (JCAMP-DX is ChemSpectra's standardized internal format; non-JCAMP-DX inputs such as Bruker FID are converted to JCAMP-DX first). Apply automatic peak detection with the default or a manually defined threshold (thres parameter) to obtain picked peaks with x/y positions. For each picked peak region, compute the coupling constant(s) from the spacing of the splitting pattern of neighbouring lines and infer the number of coupling partners; multiplicity labels are initially inferred from known libraries (refs 27, 28 in the paper) and then checked by additional rules to ensure correctness. Assemble peak position (x), intensity (y), J-coupling value(s) and multiplicity label into an assignment table, which can be exported (e.g. CSV) or transferred automatically to the Chemotion ELN via the built-in workflow that writes detected or manually chosen signals. If the rules flag an inferred assignment as incorrect, fall back to manual editing in the react-spectra-editor front end.

## Related tools

- **ChemSpectra (chem-spectra-app)** (Backend service that converts inputs to JCAMP-DX, performs threshold-based automatic peak detection, coupling constant calculation and library-based multiplicity assignment with rule-based checking) — https://github.com/ComPlat/chem-spectra-app
- **react-spectra-editor** (Front-end editor (jcampconverter library) that displays the 1H spectrum and allows manual correction of automatically inferred multiplicities) — https://github.com/ComPlat/react-spectra-editor
- **Chemotion ELN** (Main client that receives the automatically detected or manually chosen signals via the automated transfer procedure) — https://github.com/ComPlat/chemotion_ELN
- **nmrglue** (Python library used to read JCAMP-DX blocks when scripting the workflow outside the web app) — https://github.com/ComPlat/chem-spectra-app

## Examples

```
import nmrglue as ng; dic, data = ng.jcampdx.read('1h_nmr.jdx')  # read recognized 1H JCAMP-DX block before threshold-based peak picking
```

## Evaluation signals

- Every picked peak row carries a multiplicity label drawn from the standard set {s, d, t, q, quint, m}
- Reported J values equal the spacing between neighbouring lines of the splitting pattern within spectral resolution
- Inferred multiplicity labels survive the additional rule-based checks ChemSpectra applies after library inference (or are manually corrected in the editor)
- The assignment table contains one row per picked peak with position (x), intensity (y), coupling constant and multiplicity fields
- Transferred signals match the peaks visible in the spectrum preview image generated by the file management (png output)

## Limitations

- Multiplicity inference depends on known libraries plus rule checks; unusual splitting patterns (e.g. higher-order effects) may be mislabelled and require manual editing
- 2D NMR viewing/analysis is not supported — the function is limited to 1D 1H (and 13C) spectra
- Peak picking quality depends on the threshold (default or manually defined); a poorly chosen thres can miss or over-pick peaks feeding the multiplicity assignment
- Processing of FID files still needs improved phase and baseline correction, which can degrade downstream peak picking

## Evidence

- [abstract] NMR specific functions include the definition of a reference signal, the integration of signals, coupling constant calculation and multiplicity assignment: "NMR specific functions include the definition of a reference signal, the integration of signals, coupling constant calculation and multiplicity assignment"
- [discussion] threshold-based peak detection: "the automatic peak detection according to a default or manually defined threshold"
- [results] library-inferred multiplicities validated by rules (task card finding): [evidence span withheld — anchor status: not_found]
- [readme] editor scope covers NMR spectra: [evidence span withheld — anchor status: external_doc]
