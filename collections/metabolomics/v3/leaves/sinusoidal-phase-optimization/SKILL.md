---
name: sinusoidal-phase-optimization
description: Use when AEMS .wiff raw data lacks explicit per-ejection timestamps and
  peaks from 384-well plate acoustic shots must be associated with ejection times
  before MRM or full-scan peak integration.
when_to_use_negative:
- Ejection timestamps are already stored explicitly in the acquisition file, so no
  timing inference is needed
- Data are conventional LC-MS with chromatographic separation rather than AEMS, since
  peaks are not periodic at a fixed acoustic ejection interval
- Input is already a processed peak table with shots associated; the sinusoidal fit
  would be redundant
edam_operation: http://edamontology.org/operation_3454
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: rtmsEcho
  role: Implements the automated shot-to-peak association algorithm that fits the
    sinusoidal model to the TIC and outputs estimated ejection times
  repo: github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Underlying R framework that rtmsEcho extends to load and process raw MS data
- name: R
  role: Language in which the sinusoidal phase-optimization workflow is executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:2e85cb0e77f2349b3c1963a4fc4168f0dbd2d56985c57f05da783ab9d22c7a14
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: http://edamontology.org/operation_3454
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# sinusoidal-phase-optimization

## Summary

Infers acoustic ejection times for AEMS runs when they are not explicitly stored in the .wiff file, by fitting a sinusoid to the total ion chromatogram (TIC) total-intensity signal and recovering each shot's timing to within 0.01 ms.

## When to use

Use when AEMS .wiff raw data lacks explicit per-ejection timestamps and peaks from 384-well plate acoustic shots must be associated with ejection times before MRM or full-scan peak integration.

## When NOT to use

- Ejection timestamps are already stored explicitly in the acquisition file, so no timing inference is needed
- Data are conventional LC-MS with chromatographic separation rather than AEMS, since peaks are not periodic at a fixed acoustic ejection interval
- Input is already a processed peak table with shots associated; the sinusoidal fit would be redundant

## Inputs

- AEMS .wiff raw data file (total signal intensity versus time)
- start-time estimate for the first ejection
- initial estimate of milliseconds between ejections (nominal period, e.g. 1000 ms)

## Outputs

- table of inferred, evenly spaced ejection times (ejection_times.csv) with ~0.01 ms precision
- shot-to-peak associations for downstream peak detection, integration, and area quantification

## How to apply

Load the AEMS .wiff raw data file with the rtmsEcho package (built on the rtms framework in R) and extract the total ion chromatogram of total signal intensity versus time. Construct a sinusoidal reference signal with a nominal period equal to the acoustic ejection interval (e.g. 1000 ms between ejections). Starting from a start-time estimate and an initial estimate of milliseconds between peaks, optimize the phase/offset of the sinusoid to maximize agreement between the observed TIC total intensity and the sinusoidal model, following rtmsEcho's automated shot-to-peak association algorithm. Derive the inferred evenly spaced ejection times from the fitted sinusoid at approximately 0.01 ms precision, and write the table of estimated ejection times to ejection_times.csv. Validate that the fitted ejection times are evenly spaced at the nominal interval and that TIC-vs-sinusoid residuals are consistent with the ~0.01 ms precision.

## Related tools

- **rtmsEcho** (Implements the automated shot-to-peak association algorithm that fits the sinusoidal model to the TIC and outputs estimated ejection times) — github.com/nathanieltwarog/rtmsEcho
- **rtms** (Underlying R framework that rtmsEcho extends to load and process raw MS data)
- **R** (Language in which the sinusoidal phase-optimization workflow is executed)

## Evaluation signals

- Fitted ejection times are evenly spaced at the nominal ~1000 ms acoustic ejection interval
- Residuals between the observed TIC total intensity and the fitted sinusoid are consistent with ~0.01 ms timing precision
- Every peak in the run associates to exactly one ejection time, with no unassigned or ambiguous shot-to-peak matches
- The resulting ejection_times.csv table supports subsequent peak detection, baseline correction, and area quantification across a full 384-well plate

## Limitations

- The sinusoidal model assumes ejections are an evenly spaced set, so runs with irregular or variable ejection intervals may not fit
- Requires a reasonable start-time estimate and an initial estimate of milliseconds between peaks; poor initial guesses may cause the phase optimization to fail
- Timing precision is bounded at approximately 0.01 ms; the article and provided context do not report behavior on anomalous wells or failed ejections

## Evidence

- [abstract] while automating shot-to-peak association and spectral analysis: "while automating shot-to-peak association and spectral analysis"
- [other] rtmsEcho infers ejection times as an evenly spaced set, optimizing sinusoid-TIC agreement: [evidence span withheld — anchor status: not_found]
- [other] TIC extraction from AEMS .wiff files: [evidence span withheld — anchor status: not_found]
