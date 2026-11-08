---
name: aems-full-scan-spectrum-reconstruction
description: Use when processing AEMS full-scan acquisitions (PI or NL mode) stored
  in .wiff/.wiff.scan files and you need a per-ejection spectrum compatible with the
  rtms R package data structure rather than vendor software output.
when_to_use_negative:
- Input is MRM (multiple reaction monitoring) AEMS data rather than full-scan PI/NL
  mode — rtmsEcho processes MRM runs through a different path that does not require
  reconstructing per-time-point spectra
- Data are conventional LC-MS chromatography acquisitions rather than chromatography-free
  acoustic ejection acquisitions from 384- or 1536-well plates
- Vendor-specific AEMS software output is already acceptable and no customizable processing
  of the full-scan data is needed
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: rtmsEcho
  role: R package used to open the .wiff file, locate and decompress .wiff.scan blocks,
    integrate intensities over ejection windows, and assemble rtms-compatible per-ejection
    spectra
  repo: github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Previously published R framework whose data structure the reconstructed per-ejection
    spectra must be compatible with
- name: R
  role: Language in which the reader, decompression, reconstruction, integration,
    and serialization (e.g. to .rds) are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:d5b9cb4dfcbf1f814367a1f1eabdb71c3d302e6b6f12ce2bb483cf53f65f5ecb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# aems-full-scan-spectrum-reconstruction

## Summary

Reconstructs full-scan (precursor ion or neutral loss mode) acoustic ejection mass spectrometry spectra from Sciex .wiff.scan binary blocks by decompressing m/z and intensity arrays per detection time point and integrating them over each acoustic ejection's time window, yielding one rtms-compatible spectrum per ejection.

## When to use

Use when processing AEMS full-scan acquisitions (PI or NL mode) stored in .wiff/.wiff.scan files and you need a per-ejection spectrum compatible with the rtms R package data structure rather than vendor software output.

## When NOT to use

- Input is MRM (multiple reaction monitoring) AEMS data rather than full-scan PI/NL mode — rtmsEcho processes MRM runs through a different path that does not require reconstructing per-time-point spectra
- Data are conventional LC-MS chromatography acquisitions rather than chromatography-free acoustic ejection acquisitions from 384- or 1536-well plates
- Vendor-specific AEMS software output is already acceptable and no customizable processing of the full-scan data is needed

## Inputs

- Full-scan AEMS acquisition in .wiff file with embedded .wiff.scan blocks (precursor ion or neutral loss scan mode)
- Detection time point association mapping (which .wiff.scan blocks belong to each detection time point)
- Ejection time windows (estimated window per acoustic ejection, used for integration)

## Outputs

- One m/z-intensity spectrum per acoustic ejection, as objects compatible with the rtms R package data structure
- Serialized per-ejection spectra (e.g. ejection_spectra.rds)

## How to apply

Open the bundled full-scan (PI or NL mode) example .wiff file with the rtmsEcho reader in R and locate the .wiff.scan blocks associated with each detection time point. Decompress the binary m/z and intensity arrays stored within each detection time point's .wiff.scan block, then reconstruct the full m/z-intensity array for each detection time point from the decompressed byte streams. Integrate the reconstructed intensities over each ejection's estimated time window to collapse per-time-point arrays into one spectrum per acoustic ejection. Finally assemble the per-ejection spectra into objects compatible with the rtms data structure and serialize the result (e.g. to ejection_spectra.rds). The integration-over-window step is the rationale-bearing core: full-scan AEMS acquisitions sample m/z arrays per detection time point, so quantification requires summing all intensities for a given m/z across the ejection's window to produce a single spectrum per ejection.

## Related tools

- **rtmsEcho** (R package used to open the .wiff file, locate and decompress .wiff.scan blocks, integrate intensities over ejection windows, and assemble rtms-compatible per-ejection spectra) — github.com/nathanieltwarog/rtmsEcho
- **rtms** (Previously published R framework whose data structure the reconstructed per-ejection spectra must be compatible with)
- **R** (Language in which the reader, decompression, reconstruction, integration, and serialization (e.g. to .rds) are executed)

## Examples

```
library(rtmsEcho); spectra <- rtmsEcho::readFullScan('example.wiff'); saveRDS(spectra, 'ejection_spectra.rds')
```

## Evaluation signals

- The serialized spectra (e.g. ejection_spectra.rds) load successfully in R via the rtms data structure
- Each per-ejection spectrum contains a finite, non-empty m/z-intensity pair covering the full-scan acquisition window
- One spectrum is produced per acoustic ejection (per-time-point arrays are collapsed by integration over the ejection's estimated time window)
- Reconstructed m/z-intensity arrays are complete after decompression of the .wiff.scan byte streams (no truncation or decoding artifacts)

## Limitations

- No quantitative acceptance threshold is reported in the source material; validation is judged by a qualitative structural check (finite, non-empty m/z-intensity pair per ejection spectrum)
- Full-scan mode processing is slower than MRM: a full 384-well plate takes under 30 s for full-scan runs (up to a 60 Da window) versus under 2 s for MRM runs with <5 transitions
- rtmsEcho is specific to the AEMS data architecture; it does not address conventional LC-MS chromatographic data

## Evidence

- [other] rtmsEcho quantifies full-scan data by integrating all intensities for a given m/z over the ejection's estimated time window: [evidence span withheld — anchor status: not_found]
- [abstract] Direct access to AEMS data with customizable full-scan processing: "This specialized solution provides direct access to AEMS data, enabling customizable processing of both MRM and full-scan acquisitions (precursor ion and neutral loss modes)"
- [intro] Processing speed for full-scan mode runs: "Processing speed for a full 384-well plate takes less than 2 s for MRM runs (<5 transitions) and less than 30 s for full-scan mode runs (up to a 60 Da window)."
- [intro] rtms framework extension: "we developed rtmsEcho, an open-source R package extending our previously published rtms framework"
- [discussion] Multiple scan modes supported: "rtmsEcho supports multiple scan modes, including MRM, PI, and NL modes"
