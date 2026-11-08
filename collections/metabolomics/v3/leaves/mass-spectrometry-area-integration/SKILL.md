---
name: mass-spectrometry-area-integration
description: Use when processing full-scan AEMS spectra (precursor ion or neutral
  loss modes) from 384- or 1536-well plates where persistent high-intensity baseline
  signals obscure weaker, less well-defined analyte peaks that need quantification.
when_to_use_negative:
- Data are MRM acquisitions (<5 transitions) rather than full-scan spectra — rtmsEcho
  handles MRM with a separate, faster path and reference-peak alignment is unnecessary
- Spectra lack well-defined high-mass reference peaks above the persistent baseline,
  so no anchor frames can be established
- Data are conventional chromatography-coupled LC-MS runs, not acoustic ejection plate-based
  acquisitions
edam_operation: http://edamontology.org/operation_3628
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: rtmsEcho
  role: Automates shot-to-peak association, peak alignment, baseline correction, and
    area quantification for full-scan AEMS data
  repo: github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Underlying R framework that rtmsEcho extends to provide access to and processing
    of mass-spectrometry traces
  repo: github.com/nathanieltwarog/rtmsEcho
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:e537aaaecf4a15124fe9a3286a7c61aec47dc392d5f190958e05213ba02ffc19
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: http://edamontology.org/operation_3628
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# mass-spectrometry-area-integration

## Summary

Quantifies analyte abundance in acoustic ejection mass spectrometry (AEMS) data by baseline-correcting signal traces and integrating peak areas within time frames anchored to well-defined high-mass reference peaks, as implemented in the rtmsEcho R package.

## When to use

Use when processing full-scan AEMS spectra (precursor ion or neutral loss modes) from 384- or 1536-well plates where persistent high-intensity baseline signals obscure weaker, less well-defined analyte peaks that need quantification.

## When NOT to use

- Data are MRM acquisitions (<5 transitions) rather than full-scan spectra — rtmsEcho handles MRM with a separate, faster path and reference-peak alignment is unnecessary
- Spectra lack well-defined high-mass reference peaks above the persistent baseline, so no anchor frames can be established
- Data are conventional chromatography-coupled LC-MS runs, not acoustic ejection plate-based acquisitions

## Inputs

- Extracted full-scan mass spectra per well of a 384- or 1536-well plate (precursor ion or neutral loss acquisition mode)
- Plate and compound metadata (well identifiers, m/z targets)

## Outputs

- Structured peak table keyed by well and m/z with frame boundaries and integrated area per entry
- Flags for ambiguous frame assignments requiring expert review

## How to apply

Load the extracted full-scan spectra for each well using rtmsEcho in R. Detect well-defined, high-intensity high-mass peaks that stand clearly above the persistent baseline signal in each well's spectrum, and use those high-mass reference peaks as anchor points to define time frames bounding the acoustic shot events. Apply the defined frames to the weaker, less well-defined signals at other m/z values, performing baseline correction and area integration within the frames to quantify them. Assemble the frame-based quantifications into a structured peak table keyed by well and m/z, recording frame boundaries and integrated area per entry.

## Related tools

- **rtmsEcho** (Automates shot-to-peak association, peak alignment, baseline correction, and area quantification for full-scan AEMS data) — github.com/nathanieltwarog/rtmsEcho
- **rtms** (Underlying R framework that rtmsEcho extends to provide access to and processing of mass-spectrometry traces) — github.com/nathanieltwarog/rtmsEcho

## Examples

```
library(rtmsEcho); plate <- read_plate("raw_data/"); peak_table <- process_full_scan(plate, mode = "precursor ion")
```

## Evaluation signals

- Confirm the output peak table exists with required fields: well, m/z, frame boundaries, and integrated area
- Check that frame assignments are consistent with the detected high-mass reference peaks in each well's spectrum
- Verify that ambiguous frame assignments are flagged for expert review rather than silently integrated
- Confirm processing time for a full 384-well plate stays under ~30 s for full-scan mode runs (up to a 60 Da window)

## Limitations

- The method relies on well-defined high-mass peaks standing above the persistent baseline; spectra dominated by baseline signals without such reference peaks cannot be aligned this way
- Ambiguous frame assignments must be flagged for expert review, indicating the automated alignment is not always decisive
- The reported 10- to 100-fold speedup over conventional methods is not backed by named benchmark datasets in the article

## Evidence

- [methods] rtmsEcho can selectively use well-defined high-mass peaks as reference points for peak alignment: [evidence span withheld — anchor status: not_found]
- [methods] Loading and frame definition workflow: [evidence span withheld — anchor status: not_found]
- [methods] Baseline correction and area integration: [evidence span withheld — anchor status: not_found]
- [intro] Full-scan mode processing speed: "Processing speed for a full 384-well plate takes less than 2 s for MRM runs (<5 transitions) and less than 30 s for full-scan mode runs (up to a 60 Da window)."
- [discussion] Automation of critical steps: "By automating critical steps such as peak alignment, baseline correction, and area quantification"
