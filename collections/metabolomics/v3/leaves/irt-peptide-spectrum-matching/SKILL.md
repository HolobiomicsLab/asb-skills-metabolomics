---
name: irt-peptide-spectrum-matching
description: Use when a reported PSM (e.g., a USI pointing to a specific scan of a
  Thermo .raw file) must be independently reproduced from the raw spectra with matching
  search scores and fragment-ion annotation.
when_to_use_negative:
- The spectra are already in an open exchange format like mzML and no direct Thermo
  raw-file access is needed
- The analysis goal is statistical postprocessing of existing MaxQuant or Skyline
  outputs rather than verifying a raw-file PSM
- A different peptide or scan is targeted without its own documented MS-GF+ metrics
  to reproduce
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: rawrr
  role: Installs the rawrr.exe runtime and reads scan 9594 into a rawrrSpectrum object
    for fragment-ion annotation of LGGNEQVTR
  repo: https://github.com/fgcz/rawrr
- name: rawR
  role: Provides OS-independent direct access to the spectral data logged in the Thermo
    raw file, closing the gap for end-to-end analysis in R
  repo: https://github.com/fgcz/rawR
- name: RawFileReader
  role: Vendor-provided .NET API wrapped by rawrr/rawrr.exe to access spectral data
    in the proprietary Thermo raw file
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: R
  role: Statistical environment in which the spectrum is read, annotated against the
    y/b-ion series, and results written to CSV
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2020.10.30.362533
    title: rawR - Direct access to raw mass spectrometry data in R
  source_content_hash: sha256:32420b047337f1ed042f6aa1a843a63b3b7df3ac71a24337a4f68915f76691dd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fgcz/rawrr
  tools:
  - R
  - RawFileReader
  - rawR
  - rawrr
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# irt-peptide-spectrum-matching

## Summary

Reproduce and verify a peptide-spectrum match (PSM) for the doubly-charged iRT peptide LGGNEQVTR at scan 9594 of 20181113_010_autoQC01.raw by reading the vendor raw file directly in R with rawrr and recording the MS-GF+ scores (Score 144, SpecProb 1.9e-12, DB E-Value 4.4e-4).

## When to use

Use when a reported PSM (e.g., a USI pointing to a specific scan of a Thermo .raw file) must be independently reproduced from the raw spectra with matching search scores and fragment-ion annotation.

## When NOT to use

- The spectra are already in an open exchange format like mzML and no direct Thermo raw-file access is needed
- The analysis goal is statistical postprocessing of existing MaxQuant or Skyline outputs rather than verifying a raw-file PSM
- A different peptide or scan is targeted without its own documented MS-GF+ metrics to reproduce

## Inputs

- Thermo Fisher Scientific raw file 20181113_010_autoQC01.raw (MD5 a1f5df9627cf9e0d51ec1906776957ab) from MassIVE MSV000086542 or ExperimentHub EH4547
- Target scan number (9594) and peptide sequence LGGNEQVTR with charge state 2
- Reported MS-GF+ search metrics (Score 144, SpecProb 1.9e-12, DB E-Value 4.4e-4) from MassIVE RMSV000000336.1

## Outputs

- rawrrSpectrum object for scan 9594 with annotated y-ion/b-ion fragment assignments
- msgf_scan9594_LGGNEQVTR_assignment.csv containing the MS-GF+ scores, USI, and precursor filter string
- Verified precursor filter string 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]' and charge state 2

## How to apply

Obtain the raw file 20181113_010_autoQC01.raw from MassIVE dataset MSV000086542 (or ExperimentHub entry EH4547 from the tartare package) and verify its MD5 checksum a1f5df9627cf9e0d51ec1906776957ab. Install the rawrr executable runtime with rawrr::installRawrrExe(), then read the target scan into R via rawrr::readSpectrum(rawfile, scan = 9594) to create a rawrrSpectrum object. Confirm the precursor filter string 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]' and charge state 2, then annotate the y-ion (and b-ion) series against LGGNEQVTR, checking that fragment signals sit far above local noise estimates (in the paper all y-ion signals were several ten or even hundred folds above the noise estimate). Record the MS-GF+ metrics — Score 144, SpecProb 1.9e-12, DB E-Value 4.4e-4 — alongside the USI mzspec:MSV000086542:20181113_010_autoQC01:scan:9594:LGGNEQVTR/2 and write all fields to msgf_scan9594_LGGNEQVTR_assignment.csv.

## Related tools

- **rawrr** (Installs the rawrr.exe runtime and reads scan 9594 into a rawrrSpectrum object for fragment-ion annotation of LGGNEQVTR) — https://github.com/fgcz/rawrr
- **rawR** (Provides OS-independent direct access to the spectral data logged in the Thermo raw file, closing the gap for end-to-end analysis in R) — https://github.com/fgcz/rawR
- **RawFileReader** (Vendor-provided .NET API wrapped by rawrr/rawrr.exe to access spectral data in the proprietary Thermo raw file) — https://github.com/thermofisherlsms/RawFileReader
- **R** (Statistical environment in which the spectrum is read, annotated against the y/b-ion series, and results written to CSV)

## Examples

```
rawrr::installRawrrExe(); S <- rawrr::readSpectrum(rawfile, scan = 9594); S$filter  # 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]'
```

## Evaluation signals

- The precursor filter string equals 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]' and the scan charge state is 2
- Annotated y-ion fragment signals for LGGNEQVTR sit several ten to hundred folds above the local noise estimate
- Recorded MS-GF+ metrics match Score 144, SpecProb 1.9e-12, DB E-Value 4.4e-4 as documented in MassIVE RMSV000000336.1
- The raw file MD5 checksum equals a1f5df9627cf9e0d51ec1906776957ab before reading
- Output CSV contains all fields tied to the USI mzspec:MSV000086542:20181113_010_autoQC01:scan:9594:LGGNEQVTR/2

## Limitations

- rawrrSpectrum content partially depends on the instrument model and installed instrument control software version
- On Windows, the decimal symbol must be configured as '.' for mZ and intensity vectors to have correct lengths
- rawrr requires installation of the rawrr.exe executable runtime (rawrr::installRawrrExe()) before raw files can be read

## Evidence
