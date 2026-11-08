---
name: vendor-format-chromatogram-extraction
description: Use when raw AEMS data are in a vendor .wiff file and a total ion chromatogram
  (intensity vs. time) must be extracted from the vendor binary before ejection-time
  fitting or MRM/full-scan peak processing.
when_to_use_negative:
- Data are already in an open exchange format (e.g. mzML) or a processed feature table
  — the vendor-format reader is only needed for direct .wiff access
- The goal is conventional LC-MS chromatographic peak processing with retention-time
  alignment rather than chromatography-free AEMS shot-to-peak association
- The run is not an acoustic ejection MS acquisition; existing open-source MRM platforms
  (ASTS, MAVEN, MaxQuant, Skyline, XCMS-MRM, etc.) do not address AEMS's unique data
  architecture, and vice versa rtmsEcho targets AEMS specifically
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: rtmsEcho
  role: Open-source R package that loads the AEMS .wiff file and provides direct access
    to the vendor data for TIC extraction and automated shot-to-peak association
  repo: github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Underlying R framework extended by rtmsEcho that handles the raw mass-spectrometry
    data structures
- name: R
  role: Runtime language in which the rtms/rtmsEcho extraction pipeline is executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:aafbd6d82b26bb0daf6d19ca31619d6ce9a7bfbb4e799f8658c5d678630ce635
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# vendor-format-chromatogram-extraction

## Summary

Extracts total ion chromatograms (TIC) of signal intensity versus time directly from acoustic ejection mass spectrometry (AEMS) vendor .wiff raw files using the rtmsEcho/rtms R framework, providing the time-domain signal needed for downstream ejection-time inference and shot-to-peak association.

## When to use

Use when raw AEMS data are in a vendor .wiff file and a total ion chromatogram (intensity vs. time) must be extracted from the vendor binary before ejection-time fitting or MRM/full-scan peak processing.

## When NOT to use

- Data are already in an open exchange format (e.g. mzML) or a processed feature table — the vendor-format reader is only needed for direct .wiff access
- The goal is conventional LC-MS chromatographic peak processing with retention-time alignment rather than chromatography-free AEMS shot-to-peak association
- The run is not an acoustic ejection MS acquisition; existing open-source MRM platforms (ASTS, MAVEN, MaxQuant, Skyline, XCMS-MRM, etc.) do not address AEMS's unique data architecture, and vice versa rtmsEcho targets AEMS specifically

## Inputs

- AEMS .wiff raw data file (vendor binary from an acoustic ejection mass spectrometry run)
- Associated plate and compound metadata (384-well or 1536-well microplate)

## Outputs

- Total ion chromatogram (TIC): total signal intensity versus time
- Table of estimated, evenly spaced acoustic ejection times (e.g. ejection_times.csv, ~0.01 ms precision)

## How to apply

Load the AEMS .wiff raw data file with the rtmsEcho package (which extends the rtms framework in R) to gain direct access to the vendor-encoded acquisition. Extract the total ion chromatogram (TIC) of total signal intensity versus time, which serves as the primary trace for the automated shot-to-peak association algorithm. The extraction supports both MRM acquisitions and full-scan modes (precursor ion and neutral loss), so confirm the scan mode to select the right downstream processing path (MRM runs with <5 transitions process in under 2 s per 384-well plate; full-scan runs with up to a 60 Da window in under 30 s). Pass the extracted TIC to the sinusoidal ejection-timing optimizer, which fits a reference signal with a nominal ~1000 ms acoustic ejection period to locate evenly spaced ejection times to within ~0.01 ms.

## Related tools

- **rtmsEcho** (Open-source R package that loads the AEMS .wiff file and provides direct access to the vendor data for TIC extraction and automated shot-to-peak association) — github.com/nathanieltwarog/rtmsEcho
- **rtms** (Underlying R framework extended by rtmsEcho that handles the raw mass-spectrometry data structures)
- **R** (Runtime language in which the rtms/rtmsEcho extraction pipeline is executed)

## Evaluation signals

- The fitted ejection times are evenly spaced at the nominal ~1000 ms acoustic ejection interval
- Residuals between the observed TIC total intensity and the fitted sinusoid are consistent with the stated ~0.01 ms ejection-timing precision
- A full 384-well plate processes in under 2 s for MRM runs (<5 transitions) and under 30 s for full-scan runs (up to a 60 Da window); large deviations suggest a mis-specified scan mode or file-format problem
- The extracted TIC exhibits a periodic intensity profile (one feature per acoustic ejection) rather than a continuous chromatographic trace, confirming correct vendor-file decoding

## Limitations

- Direct access is specific to the AEMS .wiff vendor format; no other vendor formats are described
- No rtmsEcho version number or changelog is documented, and no benchmark datasets or accessioned data resources are named to support the reported 10- to 100-fold efficiency gains
- Baseline details of the conventional methods used for the efficiency comparison are not provided, so speed expectations may not transfer to other settings

## Evidence

- [abstract] This specialized solution provides direct access to AEMS data, enabling customizable processing of both MRM and full-scan acquisitions (precursor ion and neutral loss modes) while automating: "This specialized solution provides direct access to AEMS data, enabling customizable processing of both MRM and full-scan acquisitions (precursor ion and neutral loss modes) while automating"
- [intro] we developed rtmsEcho, an open-source R package extending our previously published rtms framework: "we developed rtmsEcho, an open-source R package extending our previously published rtms framework"
- [intro] Processing speed for a full 384-well plate takes less than 2 s for MRM runs (<5 transitions) and less than 30 s for full-scan mode runs (up to a 60 Da window).: "Processing speed for a full 384-well plate takes less than 2 s for MRM runs (<5 transitions) and less than 30 s for full-scan mode runs (up to a 60 Da window)."
