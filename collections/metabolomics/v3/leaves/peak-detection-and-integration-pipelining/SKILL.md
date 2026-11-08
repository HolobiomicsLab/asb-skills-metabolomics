---
name: peak-detection-and-integration-pipelining
description: Use when processing raw AEMS acquisitions from 384- or 1536-well plates
  in MRM or full-scan mode (PI/NL) where vendor software cannot automate shot-to-peak
  association, integration, and area quantification at high throughput.
when_to_use_negative:
- Input is conventional LC-MS chromatography data — existing open-source platforms
  (ASTS, MAVEN, MaxQuant, MRMAnalyzer, MRMPROBS, Skyline, SmartPeak, XCMS-MRM, METLIN-MRM)
  already address that setting, whereas rtmsEcho targets AEMS's unique data architecture
- The analysis requires interactive, manual per-peak curation rather than fully automated
  peak detection, integration, and quantification across a whole plate
- Vendor-specific AEMS software output is acceptable and no automated shot-to-peak
  alignment or multimode (MRM/PI/NL) integration is needed
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: rtmsEcho
  role: 'Primary implementation: automates peak detection, shot-to-peak association,
    peak alignment, baseline correction, and area quantification for MRM and full-scan
    AEMS data'
  repo: github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Underlying R framework extended by rtmsEcho that provides the core mass-spectrometry
    processing routines
- name: R
  role: Runtime environment for executing the rtmsEcho pipeline and wrapping calls
    in system.time for benchmark measurement
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:a0c3c11f144f575a150cb2aea5179fa97534fa84878b718efb942b42d2730076
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# peak-detection-and-integration-pipelining

## Summary

Automated peak detection, shot-to-peak association, alignment, baseline correction, and area quantification pipeline for acoustic ejection mass spectrometry (AEMS) data, implemented in the rtmsEcho R package on the rtms framework.

## When to use

Use when processing raw AEMS acquisitions from 384- or 1536-well plates in MRM or full-scan mode (PI/NL) where vendor software cannot automate shot-to-peak association, integration, and area quantification at high throughput.

## When NOT to use

- Input is conventional LC-MS chromatography data — existing open-source platforms (ASTS, MAVEN, MaxQuant, MRMAnalyzer, MRMPROBS, Skyline, SmartPeak, XCMS-MRM, METLIN-MRM) already address that setting, whereas rtmsEcho targets AEMS's unique data architecture
- The analysis requires interactive, manual per-peak curation rather than fully automated peak detection, integration, and quantification across a whole plate
- Vendor-specific AEMS software output is acceptable and no automated shot-to-peak alignment or multimode (MRM/PI/NL) integration is needed

## Inputs

- Raw AEMS mass spectrometry data files (MRM, precursor-ion, or neutral-loss acquisitions) from 384- or 1536-well plates
- Plate metadata mapping wells to samples
- Compound metadata defining targeted MRM transitions

## Outputs

- Detected and integrated peaks per well with automated shot-to-peak associations
- Area quantification per transition/well
- Standardized reports with interactive visualization
- Benchmark artifacts such as mrm_processing_time.json with elapsed wall-clock times

## How to apply

Load raw AEMS data files into rtmsEcho together with associated plate and compound metadata, then run the package's automated processing over the full plate: peak detection, shot-to-peak association (matching each acoustic ejection to its detected peak), peak alignment, baseline correction, and area quantification built on the rtms framework. For MRM runs with fewer than 5 transitions, a full 384-well plate should process in under 2 s; full-scan mode with up to a 60 Da window should finish in under 30 s — wrap the processing call in R's system.time to verify. The pipeline supports MRM, precursor-ion (PI), and neutral-loss (NL) scan modes and is intended to replace manual/vendor workflows that take hours, delivering 10- to 100-fold efficiency gains. Record measured timings and processing outputs (e.g. into a JSON artifact such as mrm_processing_time.json) so the benchmark and quantification can be validated against the paper's reported speeds.

## Related tools

- **rtmsEcho** (Primary implementation: automates peak detection, shot-to-peak association, peak alignment, baseline correction, and area quantification for MRM and full-scan AEMS data) — github.com/nathanieltwarog/rtmsEcho
- **rtms** (Underlying R framework extended by rtmsEcho that provides the core mass-spectrometry processing routines)
- **R** (Runtime environment for executing the rtmsEcho pipeline and wrapping calls in system.time for benchmark measurement)

## Evaluation signals

- A full 384-well MRM plate with fewer than 5 transitions processes in less than 2 s wall-clock time (verified via system.time)
- A full 384-well full-scan run with up to a 60 Da window completes in less than 30 s
- Every acoustic ejection shot is associated with a detected peak for the plate without manual alignment
- Peak areas are produced for all targeted transitions and match a standardized report with interactive visualization
- The pipeline runs end-to-end across MRM, PI, and NL scan modes without manual reconfiguration

## Limitations

- No software version number is stated for rtmsEcho (no changelog found)
- No benchmark datasets or accessioned data resources are provided to substantiate the reported 10- to 100-fold speedup, and baseline details of the conventional methods used for comparison are not given
- The <2 s benchmark applies specifically to MRM runs with fewer than 5 transitions; other transition counts or full-scan windows beyond 60 Da are not characterized

## Evidence

- [abstract] This specialized solution provides direct access to AEMS data, enabling customizable processing of both MRM and full-scan acquisitions (precursor ion and neutral loss modes): "This specialized solution provides direct access to AEMS data, enabling customizable processing of both MRM and full-scan acquisitions (precursor ion and neutral loss modes)"
- [intro] Processing speed for a full 384-well plate takes less than 2 s for MRM runs (<5 transitions) and less than 30 s for full-scan mode runs (up to a 60 Da window).: "Processing speed for a full 384-well plate takes less than 2 s for MRM runs (<5 transitions) and less than 30 s for full-scan mode runs (up to a 60 Da window)."
- [discussion] By automating critical steps such as peak alignment, baseline correction, and area quantification: "By automating critical steps such as peak alignment, baseline correction, and area quantification"
- [intro] none currently address the specialized requirements of AEMS platforms, particularly their unique data architectures and ultrahigh-throughput processing demands: "none currently address the specialized requirements of AEMS platforms, particularly their unique data architectures and ultrahigh-throughput processing demands"
