---
name: spectral-annotation-formatting
description: Use when detected peak positions from IR spectra (or shared peak lists
  from NMR/MS/IR) must be converted into bracketed annotation strings for a peak table
  or labels file, such as an IR_peak_labels.txt output.
when_to_use_negative:
- The task is 2D NMR data annotation — viewing and analysis of 2D data is explicitly
  listed as not yet supported by ChemSpectra
- The input is a raw proprietary file (RAW MS or FID NMR) that first requires conversion
  to mzML/JCAMP-DX via msConvert/pymzML before peaks can be detected
- The peaks are already annotated — the skill is for formatting detected signals,
  not for re-editing existing peak tables
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: ChemSpectra
  role: Web-based backend service that detects peaks and writes the annotated peak/signal
    information (IR intensity labels in brackets) for transfer to the ELN
  repo: https://github.com/ComPlat/chem-spectra-app
- name: react-spectra-editor
  role: JavaScript front-end in which the jcampconverter library extracts JCAMP-DX
    data and the editor displays/edits peak annotations
  repo: https://github.com/ComPlat/react-spectra-editor
- name: chem-spectra-client
  role: Standalone client page calling the ChemSpectra service; handles peaks output
    and file management
  repo: https://github.com/ComPlat/chem-spectra-client
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-020-00481-0
    title: 'ChemSpectra: a web-based spectra editor for analytical data'
  source_content_hash: sha256:1a166f8f55006c518b7ba11a4fc959b0ae3c6068d60169d33f8c0cb983bcee03
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ComPlat/chem-spectra-app
  tools:
  - ChemSpectra
  - JavaScript
  - chem-spectra-client
  - react-spectra-editor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# spectral-annotation-formatting

## Summary

Formats detected spectral peaks into standardized annotation strings — for IR, appending intensity labels (vw, w, m, s, vs) in square brackets after the wavenumber, e.g. [w] — following the annotation conventions implemented in the ChemSpectra editor.

## When to use

Use when detected peak positions from IR spectra (or shared peak lists from NMR/MS/IR) must be converted into bracketed annotation strings for a peak table or labels file, such as an IR_peak_labels.txt output.

## When NOT to use

- The task is 2D NMR data annotation — viewing and analysis of 2D data is explicitly listed as not yet supported by ChemSpectra
- The input is a raw proprietary file (RAW MS or FID NMR) that first requires conversion to mzML/JCAMP-DX via msConvert/pymzML before peaks can be detected
- The peaks are already annotated — the skill is for formatting detected signals, not for re-editing existing peak tables

## Inputs

- JCAMP-DX IR spectrum file (x/y data points)
- Detected or manually chosen peak list (wavenumber positions) from the threshold-based peak picker
- Threshold setting (default or manually defined)

## Outputs

- IR_peak_labels.txt — peak table with each wavenumber annotated by a bracketed intensity label [vw]/[w]/[m]/[s]/[vs]
- Annotated peak set transferable to the Chemotion ELN or repository

## How to apply

Parse the input JCAMP-DX file (as ChemSpectra does via the jcampconverter library in react-spectra-editor) to obtain the x/y data points. Run automatic peak detection with the default or manually defined threshold to obtain the detected peak list, or use manually chosen signals. For IR spectra, classify each detected peak into one of the five intensity categories (very weak vw, weak w, medium m, strong s, very strong vs) based on its relative intensity within the spectrum. Format each annotation by placing the intensity abbreviation in square brackets after the corresponding wavenumber, per IR reporting standards, and write the annotated peak list to the labels output (e.g. IR_peak_labels.txt). Rationale: ChemSpectra implements this workflow as an automated transfer of detected or manually chosen signals to the ELN or repository.

## Related tools

- **ChemSpectra** (Web-based backend service that detects peaks and writes the annotated peak/signal information (IR intensity labels in brackets) for transfer to the ELN) — https://github.com/ComPlat/chem-spectra-app
- **react-spectra-editor** (JavaScript front-end in which the jcampconverter library extracts JCAMP-DX data and the editor displays/edits peak annotations) — https://github.com/ComPlat/react-spectra-editor
- **chem-spectra-client** (Standalone client page calling the ChemSpectra service; handles peaks output and file management) — https://github.com/ComPlat/chem-spectra-client

## Examples

```
yarn add @complat/react-spectra-editor
```

## Evaluation signals

- Every detected IR peak in the output carries exactly one of the five bracketed intensity labels: [vw], [w], [m], [s], or [vs]
- Each annotation appears directly after the corresponding wavenumber value, matching the format ChemSpectra writes into the JCAMP peak table
- The set of annotated wavenumbers equals the set returned by the threshold-based peak detection (no missing or extra peaks)
- The output file (e.g. IR_peak_labels.txt) parses as a plain-text peak table and the annotated signals transfer cleanly to the Chemotion ELN
- Re-running detection with the same threshold yields an identical annotation set (deterministic output)

## Limitations

- Intensity classification relies on relative intensity within the spectrum; the article does not define numeric cutoffs separating vw/w/m/s/vs, so thresholds may need manual adjustment
- Peak detection quality depends on the default or manually defined threshold; poor threshold choice propagates into annotations
- The paper reports no quantitative evaluation of the annotation output, and 2D data, multi-spectrum comparison, and advanced baseline/phase correction are stated as future work

## Evidence

- [full_text] The IR editor annotates detected IR peaks by adding the intensity of the identified signals (vw, w, m, s, vs) to the recorded wavenumber: [evidence span withheld — anchor status: not_found]
- [discussion] the automatic peak detection according to a default or manually defined threshold: "the automatic peak detection according to a default or manually defined threshold"
- [abstract] an automated procedure for the transfer of the detected or manually chosen signals to the ELN was implemented: "an automated procedure for the transfer of the detected or manually chosen signals to the ELN was implemented"
- [methods] The jcampconverter library [17] inside the react-spectra-editor extracts spectroscopic data of the composed JCAMP-DX file which is sent from the server-side: "The jcampconverter library [17] inside the react-spectra-editor extracts spectroscopic data of the composed JCAMP-DX file which is sent from the server-side"
- [readme] This backend web service processes spectra -- NMR, IR, UV/VIS, MS, LC/MS, XRD, CV and more -- from JCAMP-DX, Bruker, BagIt, RAW and mzML files.: "This backend web service processes spectra -- NMR, IR, UV/VIS, MS, LC/MS, XRD, CV and more -- from JCAMP-DX, Bruker, BagIt, RAW and mzML files."
