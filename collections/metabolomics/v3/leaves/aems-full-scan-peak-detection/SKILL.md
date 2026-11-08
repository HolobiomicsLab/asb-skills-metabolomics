---
name: aems-full-scan-peak-detection
description: Use when processing full-scan AEMS plate data (384- or 1536-well) in
  R where persistent high-intensity baseline signals obscure analyte peaks and shot-to-peak
  association plus baseline correction and area integration must be automated.
when_to_use_negative:
- Data are MRM transitions rather than full-scan acquisitions — rtmsEcho has a separate
  MRM path and MRM runs complete in under 2 s with <5 transitions
- Sample is conventional LC-MS with chromatographic separation, not AEMS acoustic
  ejection nanoliter injections
- Vendor-specific AEMS software already satisfies the lab's throughput and workflow
  needs — the skill targets the four vendor constraints (shot-to-peak alignment, heterogeneous
  data integration, throughput, workflow configuration)
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: rtmsEcho
  role: Primary R package that loads full-scan AEMS data, detects reference peaks,
    defines frames, and automates baseline correction and area integration
  repo: https://github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Underlying framework that rtmsEcho extends to provide mass-spectrometry data
    access and processing
  repo: https://github.com/nathanieltwarog/rtmsEcho
- name: R
  role: Execution environment in which rtmsEcho is installed and invoked
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:86cfc516579b64876b31ec6d0a69dbb6b582d4a993ec228bcd03d1dd9670102e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nathanieltwarog/rtmsEcho
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# aems-full-scan-peak-detection

## Summary

Detects and quantifies peaks in full-scan acoustic ejection mass spectrometry (AEMS) spectra by anchoring peak alignment to well-defined high-mass reference peaks so that weak, baseline-obscured analyte signals can be framed and integrated.

## When to use

Use when processing full-scan AEMS plate data (384- or 1536-well) in R where persistent high-intensity baseline signals obscure analyte peaks and shot-to-peak association plus baseline correction and area integration must be automated.

## When NOT to use

- Data are MRM transitions rather than full-scan acquisitions — rtmsEcho has a separate MRM path and MRM runs complete in under 2 s with <5 transitions
- Sample is conventional LC-MS with chromatographic separation, not AEMS acoustic ejection nanoliter injections
- Vendor-specific AEMS software already satisfies the lab's throughput and workflow needs — the skill targets the four vendor constraints (shot-to-peak alignment, heterogeneous data integration, throughput, workflow configuration)

## Inputs

- Full-scan AEMS spectra per well of a 384- or 1536-well plate (precursor ion or neutral loss mode)
- Plate and compound metadata associated with the raw data files
- R session with rtmsEcho installed (extends rtms)

## Outputs

- Structured peak table keyed by well and m/z
- Per-well integrated peak areas after baseline correction within defined time frames
- Time-frame boundaries bounding acoustic shot events per well

## How to apply

Load each well's full-scan spectra with rtmsEcho (the R package extending rtms) and detect well-defined high-intensity high-mass peaks that stand clearly above the persistent baseline. Use those reference peaks as anchor points to define time frames bounding the acoustic shot events for each well. Apply those frames to the weaker, less well-defined signals at other m/z values, performing baseline correction and area integration within the frames to quantify them, and assemble a peak table keyed by well and m/z. rtmsEcho processes a full 384-well plate in under 30 s for full-scan mode (up to a 60 Da window), and the same package automates shot-to-peak association, peak detection, integration, and multimodal analysis that vendor software lacks.

## Related tools

- **rtmsEcho** (Primary R package that loads full-scan AEMS data, detects reference peaks, defines frames, and automates baseline correction and area integration) — https://github.com/nathanieltwarog/rtmsEcho
- **rtms** (Underlying framework that rtmsEcho extends to provide mass-spectrometry data access and processing) — https://github.com/nathanieltwarog/rtmsEcho
- **R** (Execution environment in which rtmsEcho is installed and invoked)

## Examples

```
R> library(rtmsEcho); peaks <- rtmsEcho::processFullScan(well_spectra, plate_meta)  # framed peak table keyed by well and m/z
```

## Evaluation signals

- Output peak table exists with required fields: well, m/z, frame boundaries, and integrated area
- Frame assignments are consistent with the detected high-mass reference peaks, with ambiguous assignments flagged for expert review
- Processing time for a full 384-well plate is under 30 s in full-scan mode (up to a 60 Da window)
- Full-scan acquisitions are correctly identified as precursor ion or neutral loss mode and processed accordingly
- Baseline correction visibly removes the persistent high-intensity baseline so that weak signals are quantifiable within their frames

## Limitations

- Full-scan mode is limited to mass windows of up to 60 Da; wider windows exceed the reported processing envelope
- Peak detection, integration, and multimodal analysis are constrained by the quality of high-mass reference peaks — poorly defined anchors will propagate into ambiguous frame assignments requiring expert review
- Reported 10- to 100-fold efficiency gains lack named benchmark datasets or accessioned data resources in the article

## Evidence

- [other] rtmsEcho can selectively use well-defined high-mass peaks as reference points for peak alignment: [evidence span withheld — anchor status: not_found]
- [intro] rtmsEcho achieves processing of a full 384-well plate in under 2 s for MRM runs (<5 transitions) and under 30 s for full-scan mode runs (up to a 60 Da window): "Processing speed for a full 384-well plate takes less than 2 s for MRM runs (<5 transitions) and less than 30 s for full-scan mode runs (up to a 60 Da window)."
- [abstract] rtmsEcho provides direct access to AEMS data with customizable processing of MRM and full-scan acquisitions: "This specialized solution provides direct access to AEMS data, enabling customizable processing of both MRM and full-scan acquisitions (precursor ion and neutral loss modes) while automating"
- [abstract] AEMS's full potential remains constrained by software limitations─existing tools lack robust automated processing capabilities for critical tasks such as peak detection, integration, and multimodal: "AEMS's full potential remains constrained by software limitations─existing tools lack robust automated processing capabilities for critical tasks such as peak detection, integration, and multimodal"
- [discussion] By automating critical steps such as peak alignment, baseline correction, and area quantification: "By automating critical steps such as peak alignment, baseline correction, and area quantification"
