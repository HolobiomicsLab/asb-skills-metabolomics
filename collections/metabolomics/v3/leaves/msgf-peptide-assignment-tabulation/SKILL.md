---
name: msgf-peptide-assignment-tabulation
description: Use when a published scan-to-peptide assignment (e.g. a USI referencing
  MassIVE MSV000086542) must be reproduced or verified by re-extracting the raw spectrum
  and recording the MS-GF+ scoring metrics alongside the precursor filter and charge
  state.
when_to_use_negative:
- The input is already a preprocessed mzML or feature table rather than a proprietary
  Thermo Fisher .raw file.
- No published MS-GF+ assignment (USI, Score, SpecProb, E-Value) exists to reproduce
  — use raw search/spectrum generation instead.
- The scan of interest is MS1 only; precursor filter and fragmentation series cannot
  be verified without an MS2 spectrum.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: rawrr
  role: Reads the target scan from the .raw file into a rawrrSpectrum object after
    installRawrrExe() runtime installation
  repo: https://github.com/fgcz/rawrr
- name: RawFileReader
  role: Vendor .NET API wrapped by rawrr that actually parses the Thermo Fisher raw
    file headers and spectral data
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: R
  role: Statistical environment in which the rawrr calls and downstream tabulation
    are scripted
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2020.10.30.362533
    title: rawR - Direct access to raw mass spectrometry data in R
  source_content_hash: sha256:866b7c079c3de89cf1f0730f95c927644ff3024b67c0f97ae97e4996706ad314
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fgcz/rawrr
  tools:
  - R
  - RawFileReader
  - rawR
  - rawrr
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# msgf-peptide-assignment-tabulation

## Summary

Tabulates an MS-GF+ peptide-spectrum match — Score, SpecProb, DB E-Value, USI and precursor filter string — for a specific scan in a Thermo Fisher .raw file, so assignments can be independently reproduced and audited (here: scan 9594 of 20181113_010_autoQC01.raw, iRT peptide LGGNEQVTR/2).

## When to use

Use when a published scan-to-peptide assignment (e.g. a USI referencing MassIVE MSV000086542) must be reproduced or verified by re-extracting the raw spectrum and recording the MS-GF+ scoring metrics alongside the precursor filter and charge state.

## When NOT to use

- The input is already a preprocessed mzML or feature table rather than a proprietary Thermo Fisher .raw file.
- No published MS-GF+ assignment (USI, Score, SpecProb, E-Value) exists to reproduce — use raw search/spectrum generation instead.
- The scan of interest is MS1 only; precursor filter and fragmentation series cannot be verified without an MS2 spectrum.

## Inputs

- Thermo Fisher Scientific .raw file (e.g. 20181113_010_autoQC01.raw from MassIVE MSV000086542 or ExperimentHub EH4547)
- Scan number of the assignment to verify (e.g. 9594)
- Reported peptide sequence and charge (e.g. LGGNEQVTR, charge 2)
- Published MS-GF+ metrics (Score, SpecProb, DB E-Value) and USI from the source publication

## Outputs

- rawrrSpectrum object for the target scan
- Annotated y-ion (and b-ion) series assignment for the peptide sequence
- CSV table msgf_scan9594_LGGNEQVTR_assignment.csv with filter string, charge state, MS-GF+ Score, SpecProb, DB E-Value, and USI
- Cross-reference to MassIVE RMSV000000336.1 reanalysis

## How to apply

Locate the .raw file (e.g. 20181113_010_autoQC01.raw from MassIVE MSV000086542 or ExperimentHub entry EH4547) and verify its MD5 checksum a1f5df9627cf9e0d51ec1906776957ab. Install the rawrr executable runtime with rawrr::installRawrrExe(), then read the target scan via rawrr::readSpectrum(rawfile, scan = 9594) to obtain a rawrrSpectrum object. Confirm the scan's precursor filter string ('FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]') and charge state 2, then annotate the spectrum's y-ion (and b-ion) series against the reported sequence LGGNEQVTR, checking that fragment signals sit far above the local noise estimates. Finally, record the reported MS-GF+ metrics — Score 144, SpecProb 1.9e-12, DB E-Value 4.4e-4 — together with the USI mzspec:MSV000086542:20181113_010_autoQC01:scan:9594:LGGNEQVTR/2 and the MassIVE RMSV000000336.1 reference, writing all fields to msgf_scan9594_LGGNEQVTR_assignment.csv. The rationale is that the rawrrSpectrum content depends on instrument model and instrument control software version, so the filter string and charge state must be captured as read from the file, not inferred.

## Related tools

- **rawrr** (Reads the target scan from the .raw file into a rawrrSpectrum object after installRawrrExe() runtime installation) — https://github.com/fgcz/rawrr
- **RawFileReader** (Vendor .NET API wrapped by rawrr that actually parses the Thermo Fisher raw file headers and spectral data) — https://github.com/thermofisherlsms/RawFileReader
- **R** (Statistical environment in which the rawrr calls and downstream tabulation are scripted)

## Examples

```
s <- rawrr::readSpectrum(rawfile = '20181113_010_autoQC01.raw', scan = 9594); cat(s$filterString, s$state, s$precursorCharge)
```

## Evaluation signals

- MD5 of the .raw file equals a1f5df9627cf9e0d51ec1906776957ab.
- Spectrum header shows filter string 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]' and charge state 2.
- All y-ion fragments of LGGNEQVTR fall several tens to hundreds of folds above the local noise estimate in the annotated spectrum.
- Recorded MS-GF+ values match the published Score 144, SpecProb 1.9e-12, DB E-Value 4.4e-4.
- CSV output contains the exact USI mzspec:MSV000086542:20181113_010_autoQC01:scan:9594:LGGNEQVTR/2.

## Limitations

- The rawrrSpectrum content partially depends on the instrument model and installed instrument control software version.
- On Windows, the decimal symbol must be configured as '.' or mZ and intensity vector lengths will be wrong.
- Requires the rawrr.exe runtime (installRawrrExe); no DOI-blessed parallel backend yet — exchangeable Spectra backends are planned but not implemented.

## Evidence
