---
name: wiff-scan-block-binary-parsing
description: Use when raw AEMS data are in Sciex .wiff/.wiff.scan format and you need
  full-scan (precursor ion or neutral loss) spectra per acoustic ejection rather than
  vendor MRM output.
when_to_use_negative:
- The acquisition is MRM (targeted transitions) rather than full-scan PI/NL mode —
  rtmsEcho processes MRM runs through its own targeted-transition pipeline instead
  of full-scan block reconstruction
- The data are already in an open interchange format (e.g. mzML) and do not require
  parsing proprietary .wiff.scan binary blocks
- The goal is chromatographic peak processing on conventional LC-MS data rather than
  AEMS shot-to-ejection spectra
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: rtmsEcho
  role: Reader that opens the .wiff file, locates .wiff.scan blocks per detection
    time point, and decompresses the binary m/z/intensity streams into full-scan (PI/NL)
    spectra
  repo: github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Framework whose data structure defines the target format for the assembled
    per-ejection spectra objects
- name: R
  role: Environment in which the rtmsEcho reader and rtms data structures are used
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:83b2018ff854f06fada5dff5eb95f6e7ee7f37547c789bf98ba48c7857bd10da
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# wiff-scan-block-binary-parsing

## Summary

Parses Sciex .wiff/.wiff.scan binary blocks from full-scan (PI or NL mode) AEMS acquisitions by locating per-detection-time-point scan blocks, decompressing their binary m/z and intensity arrays, and reconstructing per-ejection spectra compatible with the rtms R package data structure.

## When to use

Use when raw AEMS data are in Sciex .wiff/.wiff.scan format and you need full-scan (precursor ion or neutral loss) spectra per acoustic ejection rather than vendor MRM output.

## When NOT to use

- The acquisition is MRM (targeted transitions) rather than full-scan PI/NL mode — rtmsEcho processes MRM runs through its own targeted-transition pipeline instead of full-scan block reconstruction
- The data are already in an open interchange format (e.g. mzML) and do not require parsing proprietary .wiff.scan binary blocks
- The goal is chromatographic peak processing on conventional LC-MS data rather than AEMS shot-to-ejection spectra

## Inputs

- Sciex .wiff file with associated .wiff.scan blocks containing compressed binary m/z and intensity arrays per detection time point
- Full-scan (precursor ion or neutral loss mode) AEMS acquisition, e.g. a 384-well plate run with up to a 60 Da full-scan window

## Outputs

- Decompressed m/z-intensity arrays, one per detection time point
- One integrated spectrum per acoustic ejection
- rtms-compatible spectrum objects serialized to ejection_spectra.rds

## How to apply

Open the bundled full-scan (PI or NL mode) .wiff file with the rtmsEcho reader in R and locate the .wiff.scan blocks associated with each detection time point. Decompress the binary m/z and intensity arrays stored within each detection time point's block, then reconstruct the full m/z-intensity array for each time point from the decompressed byte streams. Integrate the reconstructed intensities over each ejection's estimated time window so that the per-time-point arrays collapse into one spectrum per acoustic ejection. Assemble the resulting per-ejection spectra into objects compatible with the rtms data structure and serialize them (e.g. to ejection_spectra.rds). The integration-over-window step is the key decision point: it is what converts rtmsEcho's time-resolved scan blocks into the per-ejection quantification unit used downstream.

## Related tools

- **rtmsEcho** (Reader that opens the .wiff file, locates .wiff.scan blocks per detection time point, and decompresses the binary m/z/intensity streams into full-scan (PI/NL) spectra) — github.com/nathanieltwarog/rtmsEcho
- **rtms** (Framework whose data structure defines the target format for the assembled per-ejection spectra objects)
- **R** (Environment in which the rtmsEcho reader and rtms data structures are used)

## Evaluation signals

- The serialized ejection_spectra.rds loads successfully in R without error
- Each per-ejection spectrum contains a finite, non-empty m/z-intensity pair covering the full-scan acquisition window (the source reports no quantitative acceptance threshold, so acceptance is this qualitative structural check)
- The number of reconstructed spectra equals the number of acoustic ejections (one spectrum per ejection, not per detection time point)
- Reconstructed per-time-point arrays are complete after decompression — no truncated or empty byte-stream outputs

## Limitations

- No quantitative acceptance threshold is reported in the source material; correctness is judged only by a qualitative structural check of the spectra
- No software version number is stated for rtmsEcho, and no changelog is available
- Full-scan processing performance is bounded by window size: a full 384-well plate takes less than 30 s only for full-scan mode runs up to a 60 Da window

## Evidence

- [other] full-scan block parsing and ejection-window integration: [evidence span withheld — anchor status: not_found]
- [abstract] scope of full-scan support: "enabling customizable processing of both MRM and full-scan acquisitions (precursor ion and neutral loss modes)"
- [intro] full-scan window and throughput bound: "Processing speed for a full 384-well plate takes less than 2 s for MRM runs (<5 transitions) and less than 30 s for full-scan mode runs (up to a 60 Da window)."
- [discussion] scan-mode coverage: "rtmsEcho supports multiple scan modes, including MRM, PI, and NL modes"
