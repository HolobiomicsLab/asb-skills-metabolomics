---
name: total-ion-chromatogram-signal-processing
description: Use when processing acoustic ejection mass spectrometry (AEMS) .wiff
  files where ejection times are not explicitly stored and shot-to-peak association
  must be recovered from the TIC signal over time.
when_to_use_negative:
- The input is conventional LC-MS chromatography data with explicit retention-time
  information rather than AEMS .wiff files lacking stored ejection times
- The vendor-specific software already provides reliable automated shot-to-peak alignment
  for the run, so TIC-based sinusoidal inference is unnecessary
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: rtmsEcho
  role: R package that loads AEMS .wiff data, extracts the TIC, and runs the sinusoidal
    shot-to-peak association algorithm to infer ejection times
  repo: https://github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Underlying R framework that rtmsEcho extends for raw mass spectrometry data
    handling
  repo: https://github.com/nathanieltwarog/rtmsEcho
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:dcd0113e4ed16c8c05723390507720a6c14e315f00076aaff385fb5e1fcd96d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nathanieltwarog/rtmsEcho
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# total-ion-chromatogram-signal-processing

## Summary

Infers acoustic ejection times from AEMS .wiff raw data by fitting a sinusoidal reference signal (nominal 1000 ms ejection period) to the total ion chromatogram (TIC), achieving ~0.01 ms precision in shot-to-peak timing.

## When to use

Use when processing acoustic ejection mass spectrometry (AEMS) .wiff files where ejection times are not explicitly stored and shot-to-peak association must be recovered from the TIC signal over time.

## When NOT to use

- The input is conventional LC-MS chromatography data with explicit retention-time information rather than AEMS .wiff files lacking stored ejection times
- The vendor-specific software already provides reliable automated shot-to-peak alignment for the run, so TIC-based sinusoidal inference is unnecessary

## Inputs

- AEMS .wiff raw data file from an acoustic ejection mass spectrometry run
- Start-time estimate for the first acoustic ejection
- Initial estimate of milliseconds between ejection peaks (nominal 1000 ms interval)

## Outputs

- Table of inferred, evenly spaced ejection times (e.g., ejection_times.csv) at ~0.01 ms precision
- Fitted sinusoidal signal aligned with the TIC total intensity over time

## How to apply

Load the AEMS .wiff raw data file with rtmsEcho (built on the rtms R framework) and extract the total ion chromatogram of total signal intensity versus time. Construct a sinusoidal reference signal with a nominal period equal to the 1000 ms acoustic ejection interval. Optimize the phase/offset of the sinusoid to maximize agreement between the observed TIC total intensity and the sinusoidal model, per the rtmsEcho automated shot-to-peak association algorithm. Derive the inferred evenly spaced ejection times from the fitted sinusoid at approximately 0.01 ms precision, then write the table of estimated ejection times (e.g., to ejection_times.csv). The rationale is that AEMS .wiff files do not explicitly store ejection times, so periodic structure in the TIC serves as the timing reference for associating each acoustic shot with its spectral peak.

## Related tools

- **rtmsEcho** (R package that loads AEMS .wiff data, extracts the TIC, and runs the sinusoidal shot-to-peak association algorithm to infer ejection times) — https://github.com/nathanieltwarog/rtmsEcho
- **rtms** (Underlying R framework that rtmsEcho extends for raw mass spectrometry data handling) — https://github.com/nathanieltwarog/rtmsEcho

## Examples

```
Rscript -e "library(rtmsEcho); data <- readAems('run.wiff'); times <- inferEjectionTimes(data); write.csv(times, 'ejection_times.csv', row.names=FALSE)"
```

## Evaluation signals

- Fitted ejection times are evenly spaced at the ~1000 ms nominal acoustic ejection interval
- Residuals between the observed TIC and the fitted sinusoid are consistent with the stated ~0.01 ms timing precision
- Each inferred ejection time maps to exactly one spectral peak in the automated shot-to-peak association
- Peak detection and subsequent quantification downstream proceed without unassigned or duplicated shots

## Limitations

- Timing inference assumes an evenly spaced ejection schedule; deviations from the nominal 1000 ms interval may degrade the sinusoidal fit
- Accuracy of the inferred times depends on a reasonable start-time estimate and initial ms-between-peaks estimate
- AEMS's full potential remains constrained by software limitations; the sinusoidal approach is specific to acoustic ejection data architectures and does not address conventional chromatographic peak alignment

## Evidence

- [other] rtmsEcho infers ejection times as an evenly spaced set: given a start-time estimate and an initial estimate of milliseconds between peaks, it optimizes agreement between the total intensity signal over time and a simple sinusoidal signal: [evidence span withheld — anchor status: not_found]
- [abstract] automated shot-to-peak association: "This specialized solution provides direct access to AEMS data, enabling customizable processing of both MRM and full-scan acquisitions (precursor ion and neutral loss modes) while automating"
- [discussion] shot-to-peak association workflow step: "By automating critical steps such as peak alignment, baseline correction, and area quantification"
