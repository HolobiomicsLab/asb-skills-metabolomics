---
name: spectral-noise-relative-intensity-assessment
description: Use when a specific scan in a Thermo Fisher .raw file (e.g., scan 9594
  of 20181113_010_autoQC01.raw) must be validated by checking that annotated fragment
  ion intensities sit far above local noise, without converting to mzML first.
when_to_use_negative:
- Input is already an open exchange format (mzML/mgf) or a non-Thermo vendor raw file
  — rawrr only reads Thermo .raw via RawFileReader
- The goal is statistical postprocessing of aggregated peptide/protein estimates (e.g.,
  MSstats on MaxQuant output) rather than inspecting a single spectrum
- A systematic, throughput-oriented XIC or LC-MS method QC question — use rawDiag-style
  chromatogram/XIC workflows instead
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: rawrr
  role: Reads the target scan (rawrr::readSpectrum) directly from the Thermo raw file
    into a rawrrSpectrum object with noise estimates; runtime installed via rawrr::installRawrrExe()
  repo: https://github.com/fgcz/rawrr
- name: RawFileReader
  role: Vendor .NET API wrapped by rawrr to access spectral data in proprietary Thermo
    Fisher raw files
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: MsBackendRawFileReader
  role: Alternative Spectra backend exposing rawrr accessors, for integration into
    Bioconductor Spectra workflows
  repo: https://github.com/cpanse/MsBackendRawFileReader
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2020.10.30.362533
    title: rawR - Direct access to raw mass spectrometry data in R
  source_content_hash: sha256:ceb64a340676109154639405142e6c4444ead234fda0838956d644c5934d750a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fgcz/rawrr
  tools:
  - MsBackendRawFileReader
  - R
  - RawFileReader
  - rawR
  - rawrr
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# spectral-noise-relative-intensity-assessment

## Summary

Assess whether candidate fragment-ion signals in a vendor raw MS2 spectrum exceed local noise estimates by orders of magnitude, using rawrr to read Thermo raw files directly in R and annotate ion series (e.g., y-ions of LGGNEQVTR) against noise baselines.

## When to use

Use when a specific scan in a Thermo Fisher .raw file (e.g., scan 9594 of 20181113_010_autoQC01.raw) must be validated by checking that annotated fragment ion intensities sit far above local noise, without converting to mzML first.

## When NOT to use

- Input is already an open exchange format (mzML/mgf) or a non-Thermo vendor raw file — rawrr only reads Thermo .raw via RawFileReader
- The goal is statistical postprocessing of aggregated peptide/protein estimates (e.g., MSstats on MaxQuant output) rather than inspecting a single spectrum
- A systematic, throughput-oriented XIC or LC-MS method QC question — use rawDiag-style chromatogram/XIC workflows instead

## Inputs

- Thermo Fisher Scientific .raw file (e.g., 20181113_010_autoQC.raw from MassIVE MSV000086542, MD5 a1f5df9627cf9e0d51ec1906776957ab)
- Target scan number (e.g., 9594)
- Assigned peptide sequence and charge state (e.g., LGGNEQVTR/2)
- rawrrSpectrum object returned by rawrr::readSpectrum

## Outputs

- Per-fragment intensity-to-noise assessment for annotated b-/y-ion series
- Confirmation record of precursor filter string and charge state
- CSV of assignment metrics (e.g., msgf_scan9594_LGGNEQVTR_assignment.csv with Score 144, SpecProb 1.9e-12, DB E-Value 4.4e-4 and the USI)

## How to apply

Read the target scan directly from the raw file with rawrr::readSpectrum(rawfile, scan = N) after installing the runtime via rawrr::installRawrrExe(); this returns a rawrrSpectrum object containing m/z and intensity vectors plus noise estimates. Confirm the precursor filter string (e.g., 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]') and charge state to ensure you are inspecting the intended MS2 scan with centroided, lock-mass-corrected peaks. Annotate the theoretical b- and y-ion series of the assigned peptide sequence (here LGGNEQVTR, doubly charged) and, for each matched fragment, compare its intensity to the scan's local noise estimate. Judge success when all annotated signals (especially the y-ion series) are several ten or even hundred folds above the noise estimate; document the metrics alongside the USI (mzspec:MSV000086542:20181113_010_autoQC01:scan:9594:LGGNEQVTR/2).

## Related tools

- **rawrr** (Reads the target scan (rawrr::readSpectrum) directly from the Thermo raw file into a rawrrSpectrum object with noise estimates; runtime installed via rawrr::installRawrrExe()) — https://github.com/fgcz/rawrr
- **RawFileReader** (Vendor .NET API wrapped by rawrr to access spectral data in proprietary Thermo Fisher raw files) — https://github.com/thermofisherlsms/RawFileReader
- **MsBackendRawFileReader** (Alternative Spectra backend exposing rawrr accessors, for integration into Bioconductor Spectra workflows) — https://github.com/cpanse/MsBackendRawFileReader

## Examples

```
S <- rawrr::readSpectrum(rawfile, scan = 9594); S[[1]]$filter; S[[1]]$mZ; S[[1]]$intensity
```

## Evaluation signals

- All annotated y-ion signals are several ten or even hundred folds above the noise estimate (expected for scan 9594 of 20181113_010_autoQC01.raw)
- Scan header matches filter = 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]' with charge state 2
- Observed MS-GF+ metrics reproduce the reported values: Score 144, SpecProb 1.9e-12, DB E-Value 4.4e-4
- Raw file integrity confirmed by MD5 a1f5df9627cf9e0d51ec1906776957ab
- Output CSV records the USI mzspec:MSV000086542:20181113_010_autoQC01:scan:9594:LGGNEQVTR/2 alongside the scores

## Limitations

- rawrrSpectrum content partially depends on the instrument model and installed instrument control software version
- On Windows, the decimal symbol must be configured as '.' for mZ and intensity vectors to have correct lengths
- Exchangeable Spectra backends and parallel computation support were planned next steps, not yet implemented at time of writing

## Evidence
