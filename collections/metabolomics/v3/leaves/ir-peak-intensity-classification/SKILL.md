---
name: ir-peak-intensity-classification
description: Use when processing IR spectra in JCAMP-DX (or converted-to-JCAMP-DX)
  form and a peak table with wavenumber plus bracketed intensity labels, e.g. '1234
  [w]', must be generated for reporting or transfer to an ELN.
when_to_use_negative:
- The input is a non-IR spectrum (NMR multiplicity/integration or MS m/z annotation
  is needed instead — those are separate type-specific functions)
- Only 2D NMR data or spectra comparison is required, which the paper lists as missing
  functionality rather than supported peak annotation
- The spectrum still needs baseline/phase correction of raw FID data — the paper flags
  advanced correction as an unimplemented future improvement
edam_operation: http://edamontology.org/operation_3357
edam_topics:
- http://edamontology.org/topic_3273
tools:
- name: ChemSpectra
  role: Performs automatic peak detection on the IR curve per a default or manually
    defined threshold and writes the annotated signal workflow output
  repo: github.com/ComPlat/chem-spectra-app
- name: react-spectra-editor
  role: Client-side editor whose jcampconverter library extracts the JCAMP-DX spectroscopic
    data for visualization and editing
  repo: https://github.com/ComPlat/react-spectra-editor
- name: Chemotion ELN
  role: Receives the detected or manually chosen signals via the automated transfer
    procedure
  repo: https://github.com/ComPlat/chemotion_ELN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-020-00481-0
    title: 'ChemSpectra: a web-based spectra editor for analytical data'
  source_content_hash: sha256:ca65b4a4b719533e5a769f957db84edd79b7695ead4d54417a04eb55299b8cfa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemSpectra
  - Chemotion ELN
  - JavaScript
  - react-spectra-editor
  edam_operation: http://edamontology.org/operation_3357
  edam_topics:
  - http://edamontology.org/topic_3273
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# ir-peak-intensity-classification

## Summary

Classify automatically detected IR peaks into the five intensity categories very weak (vw), weak (w), medium (m), strong (s) and very strong (vs) based on relative intensity, and annotate each wavenumber with the bracketed label following IR reporting conventions as implemented in the ChemSpectra IR editor.

## When to use

Use when processing IR spectra in JCAMP-DX (or converted-to-JCAMP-DX) form and a peak table with wavenumber plus bracketed intensity labels, e.g. '1234 [w]', must be generated for reporting or transfer to an ELN.

## When NOT to use

- The input is a non-IR spectrum (NMR multiplicity/integration or MS m/z annotation is needed instead — those are separate type-specific functions)
- Only 2D NMR data or spectra comparison is required, which the paper lists as missing functionality rather than supported peak annotation
- The spectrum still needs baseline/phase correction of raw FID data — the paper flags advanced correction as an unimplemented future improvement

## Inputs

- IR spectrum file in JCAMP-DX format (or non-JCAMP-DX input already converted to JCAMP-DX as a first processing step)
- x/y data points extracted by a JCAMP parser (jcampconverter / JCAMP read blocks)
- peak-detection threshold (ChemSpectra default or a user-defined value)

## Outputs

- List of detected IR peak wavenumbers with bracketed intensity labels, e.g. 1685 [s], 1450 [w]
- IR peak-label output file (e.g. IR_peak_labels.txt) containing the annotated peak table
- Signals packaged for automated transfer to the Chemotion ELN or repository

## How to apply

Load the IR file via a JCAMP parser (e.g. jcampconverter inside react-spectra-editor, which extracts spectroscopic data of the composed JCAMP-DX file) and pull out the x/y data points. Run automatic peak detection using ChemSpectra's threshold logic — a default or manually defined threshold decides which signals are reported. For each detected peak, judge its relative intensity within the spectrum and assign one of the five categories: very weak, weak, medium, strong, very strong. Write the annotated peaks as wavenumber followed by the bracketed intensity abbreviation, e.g. '[w]', reproducing the peak-table format ChemSpectra writes, then output the labels (e.g. to IR_peak_labels.txt) for transfer to the ELN or repository via the automated signal-transfer workflow.

## Related tools

- **ChemSpectra** (Performs automatic peak detection on the IR curve per a default or manually defined threshold and writes the annotated signal workflow output) — github.com/ComPlat/chem-spectra-app
- **react-spectra-editor** (Client-side editor whose jcampconverter library extracts the JCAMP-DX spectroscopic data for visualization and editing) — https://github.com/ComPlat/react-spectra-editor
- **Chemotion ELN** (Receives the detected or manually chosen signals via the automated transfer procedure) — https://github.com/ComPlat/chemotion_ELN

## Examples

```
import nmrglue as ng; dic, data = ng.read_jcamp(open('spectrum.jdx')); peaks = detect_peaks(data, threshold=default); print([(x, intensity_label(y)) for x, y in peaks])
```

## Evaluation signals

- Every detected IR peak in the output carries exactly one of the five bracketed labels: [vw], [w], [m], [s], [vs]
- Each label sits immediately after its corresponding wavenumber, matching the annotation format written into the JCAMP peak table
- The set of annotated wavenumbers matches the peak list produced by the threshold-based automatic peak detection (default or user-defined threshold)
- The peak labels file (e.g. IR_peak_labels.txt) is complete and loadable for transfer into the Chemotion ELN via the automated signal-transfer procedure

## Limitations

- The intensity categories are relative to the specific spectrum, so labels are not comparable across independently recorded spectra without normalization
- Peak detection quality depends on the chosen threshold; a poorly set manual threshold will miss weak peaks or flood the table with noise peaks
- The paper notes ChemSpectra should be extended in the future with further data type-specific analysis functions, so IR annotation is limited to the implemented peak/intensity workflow

## Evidence

- [methods] jcampconverter extracts JCAMP-DX data: "The jcampconverter library [17] inside the react-spectra-editor extracts spectroscopic data of the composed JCAMP-DX file"
- [discussion] automatic peak detection threshold: "the automatic peak detection according to a default or manually defined threshold"
- [discussion] signal transfer workflow: "a workflow to write the automatically detected or manually chosen signals was implemented. This allows the direct transfer of information to e.g. the ELN or repository."
- [readme] backend processes IR from JCAMP-DX: "This backend web service processes spectra -- NMR, IR, UV/VIS, MS, LC/MS, XRD, CV and more -- from JCAMP-DX, Bruker, BagIt, RAW and mzML files."
