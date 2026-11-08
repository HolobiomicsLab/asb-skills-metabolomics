---
name: ejection-time-window-intensity-integration
description: Use when processing full-scan acoustic ejection mass spectrometry (AEMS)
  data in precursor ion (PI) or neutral loss (NL) mode from SCIEX .wiff/.wiff.scan
  files and a per-ejection spectrum or quantified signal is needed.
when_to_use_negative:
- The acquisition is MRM mode rather than full-scan — rtmsEcho processes MRM transitions
  with a different, faster path (<2 s for a 384-well plate), not per-ejection full-scan
  spectra
- The data are conventional LC-MS chromatography runs rather than AEMS acquisitions,
  where chromatographic separation (not ejection time windows) defines the analysis
  dimensions
- The vendor-specific AEMS software workflow is sufficient and the user does not need
  open-source, rtms-compatible per-ejection spectra
edam_operation: http://edamontology.org/operation_0336
tools:
- name: rtmsEcho
  role: Reads the full-scan .wiff file, decompresses .wiff.scan m/z/intensity arrays,
    and performs the per-ejection time-window intensity integration
  repo: github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Defines the data structure the assembled per-ejection spectra must be compatible
    with
- name: R
  role: Runtime environment in which rtmsEcho is executed and the ejection_spectra.rds
    output is serialized
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:80f18177b04bc843565bbcaa953f2ca05953e93fe6e07a1780fefdc5305ae070
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: http://edamontology.org/operation_0336
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# ejection-time-window-intensity-integration

## Summary

A full-scan AEMS quantification method in which rtmsEcho integrates all intensities for a given m/z over each acoustic ejection's estimated time window, collapsing reconstructed .wiff.scan spectra into one rtms-compatible spectrum per ejection.

## When to use

Use when processing full-scan acoustic ejection mass spectrometry (AEMS) data in precursor ion (PI) or neutral loss (NL) mode from SCIEX .wiff/.wiff.scan files and a per-ejection spectrum or quantified signal is needed.

## When NOT to use

- The acquisition is MRM mode rather than full-scan — rtmsEcho processes MRM transitions with a different, faster path (<2 s for a 384-well plate), not per-ejection full-scan spectra
- The data are conventional LC-MS chromatography runs rather than AEMS acquisitions, where chromatographic separation (not ejection time windows) defines the analysis dimensions
- The vendor-specific AEMS software workflow is sufficient and the user does not need open-source, rtms-compatible per-ejection spectra

## Inputs

- SCIEX .wiff file containing full-scan AEMS acquisitions in precursor ion (PI) or neutral loss (NL) mode
- .wiff.scan blocks holding compressed binary m/z and intensity arrays per detection time point
- Ejection time windows associated with each acoustic ejection

## Outputs

- One reconstructed m/z-intensity spectrum per acoustic ejection, in rtms-compatible data structures
- Serialized per-ejection spectra (e.g. ejection_spectra.rds) loadable in R via the rtms framework

## How to apply

Open the bundled full-scan (PI or NL mode) .wiff file with the rtmsEcho reader in R and locate the .wiff.scan blocks associated with each detection time point. Decompress the binary m/z and intensity arrays stored in each detection time point's .wiff.scan block and reconstruct the full m/z-intensity array for that time point. Then integrate the reconstructed intensities over each ejection's estimated time window to collapse the per-time-point arrays into one spectrum per acoustic ejection — this time-window integration is the core quantification step, since full-scan AEMS has no chromatographic retention dimension. Assemble the per-ejection spectra into objects compatible with the rtms package's data structure and serialize the result (e.g. to ejection_spectra.rds). Full-scan processing is sized to handle up to a 60 Da acquisition window and complete a full 384-well plate in under 30 s, so keep windows within that bound.

## Related tools

- **rtmsEcho** (Reads the full-scan .wiff file, decompresses .wiff.scan m/z/intensity arrays, and performs the per-ejection time-window intensity integration) — github.com/nathanieltwarog/rtmsEcho
- **rtms** (Defines the data structure the assembled per-ejection spectra must be compatible with)
- **R** (Runtime environment in which rtmsEcho is executed and the ejection_spectra.rds output is serialized)

## Evaluation signals

- Each serialized per-ejection spectrum contains a finite, non-empty m/z-intensity pair covering the full-scan acquisition window (qualitative structural check; no quantitative threshold is reported)
- The serialized ejection_spectra.rds loads successfully in R and its objects conform to the rtms data structure
- The number of reconstructed spectra equals the number of acoustic ejections represented by the detection time points in the .wiff.scan blocks
- Processing of a full 384-well plate in full-scan mode (up to a 60 Da window) completes in under 30 s
- Integration collapses per-time-point arrays rather than per-ejection raw traces: each output spectrum reflects intensities summed over one ejection's estimated time window

## Limitations

- No quantitative acceptance threshold is reported in the source material, so validation is a qualitative structural check on spectrum completeness
- The 30 s full-scan processing bound applies to windows up to 60 Da; larger windows are not characterized
- No benchmark datasets or accessioned data resources are named in the context to independently verify the integration step

## Evidence

- [other] rtmsEcho quantifies full-scan data by integrating all intensities for a given m/z over the ejection's estimated time window: [evidence span withheld — anchor status: not_found]
- [abstract] customizable full-scan processing: "This specialized solution provides direct access to AEMS data, enabling customizable processing of both MRM and full-scan acquisitions (precursor ion and neutral loss modes)"
- [discussion] scan modes supported: "rtmsEcho supports multiple scan modes, including MRM, PI, and NL modes"
- [intro] full-scan throughput bound: "Processing speed for a full 384-well plate takes less than 2 s for MRM runs (<5 transitions) and less than 30 s for full-scan mode runs (up to a 60 Da window)."
