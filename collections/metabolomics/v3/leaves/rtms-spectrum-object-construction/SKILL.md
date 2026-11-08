---
name: rtms-spectrum-object-construction
description: Use when wiff.scan full-scan (PI or NL mode) AEMS data must be turned
  into per-ejection m/z-intensity spectra compatible with the rtms R package data
  structure.
when_to_use_negative:
- The acquisition is MRM-mode rather than full-scan PI/NL — rtmsEcho handles MRM runs
  as a separate processing path (<5 transitions, under 2 s for a 384-well plate)
- The data are already per-ejection spectra or rtms-compatible objects (e.g. an existing
  ejection_spectra.rds) — no .wiff.scan reconstruction is needed
- The .wiff.scan blocks are not full-scan PI/NL acquisitions, since the decompress-then-integrate
  workflow is specific to full-scan mode data
tools:
- name: rtmsEcho
  role: Reads the .wiff file, locates and decompresses .wiff.scan blocks per detection
    time point, and supports extraction of full-scan PI/NL acquisitions
  repo: https://github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Defines the target data structure that the assembled per-ejection spectra
    must be compatible with before serialization to ejection_spectra.rds
- name: R
  role: Language in which the reader, decompression, integration, and serialization
    workflow is executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:70529b86378a3a058eb2edc283237d0f6e341c5c58790ab34d8861a80b421870
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nathanieltwarog/rtmsEcho
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# rtms-spectrum-object-construction

## Summary

Reconstructs full-scan (precursor ion or neutral loss mode) AEMS spectra from .wiff.scan blocks into rtms-compatible per-ejection spectrum objects by decompressing binary m/z-intensity arrays and integrating intensities over each ejection's estimated time window.

## When to use

Use when .wiff.scan full-scan (PI or NL mode) AEMS data must be turned into per-ejection m/z-intensity spectra compatible with the rtms R package data structure.

## When NOT to use

- The acquisition is MRM-mode rather than full-scan PI/NL — rtmsEcho handles MRM runs as a separate processing path (<5 transitions, under 2 s for a 384-well plate)
- The data are already per-ejection spectra or rtms-compatible objects (e.g. an existing ejection_spectra.rds) — no .wiff.scan reconstruction is needed
- The .wiff.scan blocks are not full-scan PI/NL acquisitions, since the decompress-then-integrate workflow is specific to full-scan mode data

## Inputs

- Full-scan (PI or NL mode) AEMS .wiff file containing .wiff.scan blocks
- .wiff.scan binary m/z and intensity arrays per detection time point
- Ejection time-window estimates (to define integration boundaries per acoustic ejection)

## Outputs

- One reconstructed m/z-intensity spectrum per acoustic ejection
- rtms-compatible spectrum objects serialized to ejection_spectra.rds

## How to apply

Open the full-scan (PI or NL mode) .wiff file with the rtmsEcho reader in R and locate the .wiff.scan blocks associated with each detection time point. Decompress the binary m/z and intensity arrays stored within each detection time point's .wiff.scan block, then reconstruct the full m/z-intensity array for each time point from the decompressed byte streams. Integrate the reconstructed intensities over each ejection's estimated time window to collapse per-time-point arrays into one spectrum per acoustic ejection. Assemble the per-ejection spectra into objects compatible with the rtms data structure and serialize the result to ejection_spectra.rds. The rationale is that rtmsEcho quantifies full-scan data by integrating all intensities for a given m/z over the ejection's time window, producing output that plugs into the previously published rtms framework.

## Related tools

- **rtmsEcho** (Reads the .wiff file, locates and decompresses .wiff.scan blocks per detection time point, and supports extraction of full-scan PI/NL acquisitions) — https://github.com/nathanieltwarog/rtmsEcho
- **rtms** (Defines the target data structure that the assembled per-ejection spectra must be compatible with before serialization to ejection_spectra.rds)
- **R** (Language in which the reader, decompression, integration, and serialization workflow is executed)

## Evaluation signals

- The serialized ejection_spectra.rds loads successfully in R
- Each per-ejection spectrum contains a finite, non-empty m/z-intensity pair
- Each per-ejection spectrum covers the full-scan acquisition window
- Exactly one spectrum is produced per acoustic ejection (per-time-point arrays collapse correctly onto the ejection's time window)
- The output objects are structurally compatible with the rtms data structure

## Limitations

- No quantitative acceptance threshold is reported in the source material; acceptance is judged by the qualitative structural check (finite, non-empty spectra covering the acquisition window)
- Full-scan mode processing scales with window size: a full 384-well plate takes less than 30 s only for full-scan runs up to a 60 Da window
- No software version number is stated for rtmsEcho, complicating reproducibility of the extraction behavior

## Evidence

- [abstract] rtmsEcho extends the rtms framework for full-scan extraction: "we developed rtmsEcho, an open-source R package that extends our previously published rtms framework"
- [abstract] rtmsEcho handles both MRM and full-scan PI/NL acquisitions: "provides direct access to AEMS data, enabling customizable processing of both MRM and full-scan acquisitions (precursor ion and neutral loss modes)"
- [discussion] Supported scan modes include MRM, PI, and NL: "rtmsEcho supports multiple scan modes, including MRM, PI, and NL modes"
- [intro] Pipeline components: raw data upload, automated processing, reporting: "raw data file upload with associated plate and compound metadata; automated processing through optimized algorithms"
