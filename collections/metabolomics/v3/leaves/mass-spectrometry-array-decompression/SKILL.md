---
name: mass-spectrometry-array-decompression
description: Use when raw vendor MS files (e.g. SCIEX .wiff/.wiff.scan) contain compressed
  binary m/z and intensity arrays that must be decompressed and reassembled into usable
  spectra before peak integration or quantification.
when_to_use_negative:
- Input is already an open-format spectrum (e.g. mzML or an R rtms object) that needs
  no byte-stream decompression.
- Data are MRM acquisitions rather than full-scan (PI or NL) mode, where per-transition
  traces rather than reconstructed full spectra are needed.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: rtmsEcho
  role: R package that reads .wiff files, locates and decompresses .wiff.scan binary
    m/z/intensity blocks, and reconstructs full-scan spectra per detection time point
  repo: github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Target data structure framework into which the reconstructed per-ejection
    spectra are assembled
- name: R
  role: Language environment in which the decompression and reconstruction pipeline
    is executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:c708269c04fccc8d4e886fc9dad67cfcd0b06645157800b6f71eedaa122fb756
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mass-spectrometry-array-decompression

## Summary

Reconstructs full m/z-intensity spectra from compressed binary byte streams embedded in vendor scan blocks (e.g. .wiff.scan blocks from SCIEX .wiff files) so that per-ejection AEMS spectra can be built in rtms-compatible R data structures.

## When to use

Use when raw vendor MS files (e.g. SCIEX .wiff/.wiff.scan) contain compressed binary m/z and intensity arrays that must be decompressed and reassembled into usable spectra before peak integration or quantification.

## When NOT to use

- Input is already an open-format spectrum (e.g. mzML or an R rtms object) that needs no byte-stream decompression.
- Data are MRM acquisitions rather than full-scan (PI or NL) mode, where per-transition traces rather than reconstructed full spectra are needed.

## Inputs

- SCIEX .wiff / .wiff.scan raw mass spectrometry file (full-scan PI or NL mode AEMS acquisition)
- Per-detection-time-point compressed binary m/z and intensity byte streams within .wiff.scan blocks

## Outputs

- Reconstructed full m/z-intensity array for each detection time point
- Per-ejection spectra (e.g. serialized as ejection_spectra.rds) compatible with the rtms data structure

## How to apply

Open the raw file (e.g. a bundled full-scan PI or NL mode .wiff file) with a reader such as rtmsEcho and locate the .wiff.scan blocks associated with each detection time point. Decompress the binary m/z and intensity arrays stored within each detection time point's .wiff.scan block, then reconstruct the full m/z-intensity array for each time point from the decompressed byte streams. These per-time-point arrays are the substrate for downstream steps such as integrating intensities over each acoustic ejection's time window to collapse them into one spectrum per ejection. The rationale is that vendor binary formats are not directly readable by open-source tooling, so byte-level decompression is the prerequisite for rtms-compatible spectral analysis.

## Related tools

- **rtmsEcho** (R package that reads .wiff files, locates and decompresses .wiff.scan binary m/z/intensity blocks, and reconstructs full-scan spectra per detection time point) — github.com/nathanieltwarog/rtmsEcho
- **rtms** (Target data structure framework into which the reconstructed per-ejection spectra are assembled)
- **R** (Language environment in which the decompression and reconstruction pipeline is executed)

## Examples

```
Rscript -e 'library(rtmsEcho); w <- read.wiff("sample.wiff"); spectra <- extractEjections(w); saveRDS(spectra, "ejection_spectra.rds")'
```

## Evaluation signals

- Each reconstructed per-ejection spectrum contains a finite, non-empty m/z-intensity pair.
- The reconstructed spectra cover the full-scan acquisition window (full-scan mode up to a 60 Da window).
- The serialized output (e.g. ejection_spectra.rds) loads successfully back into R as rtms-compatible objects.
- Each detection time point in the .wiff.scan block yields one reconstructed array; the number of arrays matches the number of detection time points.

## Limitations

- No quantitative acceptance threshold is reported in the source material; validation of the reconstruction is a qualitative structural check (finite, non-empty spectra covering the acquisition window).
- Full-scan (PI/NL) reconstruction of a 384-well plate can take up to 30 s, versus under 2 s for MRM runs with <5 transitions.
- The compressed byte-stream format is vendor-specific (.wiff.scan), so the approach does not transfer directly to other vendors' raw formats.

## Evidence

- [other] rtmsEcho quantifies full-scan data by integrating all intensities for a given m/z over the ejection's estimated time window, producing a per-ejection spectrum that is compatible with the previously published rtms R package data structure.: [evidence span withheld — anchor status: not_found]
- [other] Full-scan reconstruction workflow decompresses binary arrays.: [evidence span withheld — anchor status: not_found]
- [discussion] rtmsEcho supports multiple scan modes including full-scan PI and NL.: "rtmsEcho supports multiple scan modes, including MRM, PI, and NL modes"
- [abstract] Package provides direct access to AEMS data with customizable processing.: "This specialized solution provides direct access to AEMS data, enabling customizable processing of both MRM and full-scan acquisitions (precursor ion and neutral loss modes)"
- [intro] Processing speed benchmark for full-scan mode.: "Processing speed for a full 384-well plate takes less than 2 s for MRM runs (<5 transitions) and less than 30 s for full-scan mode runs (up to a 60 Da window)."
