---
name: high-mass-reference-peak-frame-definition
description: Reference for full-scan AEMS spectra where a persistent high-intensity
  baseline obscures weak analyte peaks, and you need to define shot-event frames from
  high-mass anchor peaks.
when_to_use_negative:
- Data are MRM or other targeted transitions rather than full-scan acquisitions, where
  shot-to-peak association is handled directly by rtmsEcho's MRM pipeline
- The persistent baseline is negligible and weak peaks are already well-defined, so
  direct peak detection/integration without frame anchoring would be simpler
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: rtmsEcho
  role: R package used to load extracted full-scan spectra and detect the high-mass
    anchor peaks that define the time frames for weak-signal quantification
  repo: github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Underlying framework extended by rtmsEcho, providing the spectral handling
    used for the frame definition and quantification steps
  repo: github.com/nathanieltwarog/rtmsEcho
- name: R
  role: Language in which rtmsEcho is implemented and in which the frame-definition
    workflow is executed
  repo: github.com/nathanieltwarog/rtmsEcho
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:f042c955ced7f4f3846b698fe0a44b1b78c77a528a81181f3f84ed2cc129e5c1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# high-mass-reference-peak-frame-definition

## Summary

Defines quantification frames for weak AEMS signals by anchoring on well-defined, high-mass peaks that stand clearly above a persistent baseline, using those time frames to bound acoustic shot events and quantify less well-defined lower signals.

## When to use

Reference for full-scan AEMS spectra where a persistent high-intensity baseline obscures weak analyte peaks, and you need to define shot-event frames from high-mass anchor peaks.

## When NOT to use

- Data are MRM or other targeted transitions rather than full-scan acquisitions, where shot-to-peak association is handled directly by rtmsEcho's MRM pipeline
- The persistent baseline is negligible and weak peaks are already well-defined, so direct peak detection/integration without frame anchoring would be simpler

## Inputs

- Extracted full-scan spectra for each well of a 384-well or 1536-well microplate (AEMS acoustic ejection mass spectrometry data)

## Outputs

- Structured peak table keyed by well and m/z with frame boundaries and integrated area per signal
- Flagged list of ambiguous frame assignments for expert review

## How to apply

In R, load extracted full-scan spectra for each well of the plate using rtmsEcho (an extension of the rtms framework). Detect well-defined, high-intensity high-mass peaks that stand clearly above the persistent baseline in each well's spectrum, and use them as anchor points to define time frames bounding the acoustic shot events for that well. Apply those frames to weaker, less well-defined signals at other m/z values: within each frame, perform baseline correction and area integration to quantify the target signal. Assemble the frame-based quantifications into a structured peak table keyed by well and m/z, flagging any ambiguous frame assignments for expert review.

## Related tools

- **rtmsEcho** (R package used to load extracted full-scan spectra and detect the high-mass anchor peaks that define the time frames for weak-signal quantification) — github.com/nathanieltwarog/rtmsEcho
- **rtms** (Underlying framework extended by rtmsEcho, providing the spectral handling used for the frame definition and quantification steps) — github.com/nathanieltwarog/rtmsEcho
- **R** (Language in which rtmsEcho is implemented and in which the frame-definition workflow is executed) — github.com/nathanieltwarog/rtmsEcho

## Examples

```
R> library(rtmsEcho); well.spectra <- loadFullScanSpectra(plate) ; frames <- defineFramesFromHighMassPeaks(well.spectra) ; peaks <- quantifyWithinFrames(well.spectra, frames)
```

## Evaluation signals

- The output peak table exists with required fields: well, m/z, frame boundaries, and integrated area per signal
- Frame assignments are consistent with the detected high-mass reference peaks; any ambiguous assignments are flagged for expert review
- Weak signals at other m/z values are quantified only within the frames defined by the anchor peaks (no out-of-frame integration)
- Baseline correction is applied within each frame before area integration, so reported areas are not inflated by the persistent baseline
- A full 384-well plate processes in the expected runtime (<30 s for full-scan mode runs with up to a 60 Da window)

## Limitations

- Applicable to full-scan AEMS spectra whose persistent high-intensity baseline obscures analyte peaks; not described for MRM runs
- Frame assignments can be ambiguous when no clear high-mass reference peak stands out, requiring expert review
- No DOI or publication venue is given for the rtmsEcho package itself; no benchmark datasets or accessioned data are named to support the reported 10- to 100-fold speedup

## Evidence

- [abstract] High-mass reference peaks anchor frame definition: [evidence span withheld — anchor status: not_found]
- [intro] rtmsEcho extends the rtms R framework: "we developed rtmsEcho, an open-source R package extending our previously published rtms framework"
- [intro] Persistent baseline limitation in AEMS software: "yet the vendor-specific software solutions remain inadequate for large-scale studies due to four fundamental constraints"
- [discussion] Frame-based quantification avoids baseline inflation: "By automating critical steps such as peak alignment, baseline correction, and area quantification"
