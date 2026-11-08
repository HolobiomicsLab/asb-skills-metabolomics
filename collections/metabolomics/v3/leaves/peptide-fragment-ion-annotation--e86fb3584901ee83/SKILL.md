---
name: peptide-fragment-ion-annotation
description: Use when a specific scan in a Thermo .raw file has an assigned PSM (peptide
  + charge) and you must verify it by annotating the y-/b-ion series against fragment
  m/z values and comparing signal intensity to noise estimates.
when_to_use_negative:
- The input data is already an open exchange format (mzML) or non-Thermo vendor raw
  data — rawR/rawrr wrap the vendor RawFileReader API specifically for proprietary
  Thermo Fisher Scientific raw files.
- No peptide sequence/charge assignment exists for the scan — this skill validates
  a given PSM by fragment annotation; it does not perform de novo sequencing or database
  search itself.
- The goal is chromatographic rather than spectral inspection (e.g. XIC extraction,
  which uses filter = 'ms' MS1-level scans and 10 ppm tolerance instead of MS2 fragment
  annotation).
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: rawrr
  role: R package used to install the runtime (rawrr::installRawrrExe()), read the
    target scan into a rawrrSpectrum object via rawrr::readSpectrum, and expose the
    filter string and mZ/intensity vectors needed for y-/b-ion annotation against
    LGGNEQVTR.
  repo: fgcz/rawrr
- name: RawFileReader
  role: Vendor-provided .NET API wrapped by rawrr to access spectral data logged in
    the proprietary Thermo Fisher raw file underpinning the annotated scan.
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: MsBackendRawFileReader
  role: Alternative MsBackend for the Bioconductor Spectra package built on rawrr
    accessors; optional route to the same spectral data for annotation within the
    Spectra framework.
  repo: fgcz/MsBackendRawFileReader
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2020.10.30.362533
    title: rawR - Direct access to raw mass spectrometry data in R
  source_content_hash: sha256:463ea6e921b7782f99c0bc0514aa3a2dd4ee3436607be3dbdfcaeac5b62489f5
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
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# peptide-fragment-ion-annotation

## Summary

Annotate an MS2 spectrum from a Thermo Fisher raw file against an assigned peptide sequence (here the doubly-charged iRT peptide LGGNEQVTR at scan 9594 of 20181113_010_autoQC01.raw) by mapping its b-/y-ion series and verifying fragment signals exceed the local noise estimate, thereby independently validating a peptide-spectrum match and its reported MS-GF+ scores.

## When to use

Use when a specific scan in a Thermo .raw file has an assigned PSM (peptide + charge) and you must verify it by annotating the y-/b-ion series against fragment m/z values and comparing signal intensity to noise estimates.

## When NOT to use

- The input data is already an open exchange format (mzML) or non-Thermo vendor raw data — rawR/rawrr wrap the vendor RawFileReader API specifically for proprietary Thermo Fisher Scientific raw files.
- No peptide sequence/charge assignment exists for the scan — this skill validates a given PSM by fragment annotation; it does not perform de novo sequencing or database search itself.
- The goal is chromatographic rather than spectral inspection (e.g. XIC extraction, which uses filter = 'ms' MS1-level scans and 10 ppm tolerance instead of MS2 fragment annotation).

## Inputs

- Thermo Fisher Scientific raw file (e.g. 20181113_010_autoQC01.raw from MassIVE MSV000086542 or ExperimentHub EH4547, MD5 a1f5df9627cf9e0d51ec1906776957ab)
- Assigned peptide sequence and precursor charge state (e.g. LGGNEQVTR, charge 2)
- Scan number of the target MS2 spectrum (e.g. 9594)
- Reported MS-GF+ search metrics to compare against (Score, SpecProb, DB E-Value)

## Outputs

- rawrrSpectrum object with scan metadata, filter string, and mZ/intensity vectors
- b-ion and y-ion annotation of the spectrum against the peptide sequence with signal-vs-noise assessment
- CSV of the assignment record, e.g. msgf_scan9594_LGGNEQVTR_assignment.csv containing the USI, filter string, and MS-GF+ Score/SpecProb/DB E-Value

## How to apply

Install the rawrr executable runtime with rawrr::installRawrrExe(), then read the target scan into R via rawrr::readSpectrum(rawfile, scan = 9594) to obtain a rawrrSpectrum object. Confirm the precursor filter string 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]' and charge state 2 for the scan before annotating. Compute the theoretical b- and y-ion m/z series for the assigned sequence LGGNEQVTR and match observed fragment signals to them, checking that the y-ion signals sit several ten or even hundred folds above the local noise estimate. Record the reported MS-GF+ metrics for the match — Score 144, SpecProb 1.9e-12, DB E-Value 4.4e-4 — alongside the USI mzspec:MSV000086542:20181113_010_autoQC01:scan:9594:LGGNEQVTR/2, and write all fields to a CSV such as msgf_scan9594_LGGNEQVTR_assignment.csv. The rationale is that a correct assignment must show the expected fragment series well above noise, not merely a favorable search score.

## Related tools

- **rawrr** (R package used to install the runtime (rawrr::installRawrrExe()), read the target scan into a rawrrSpectrum object via rawrr::readSpectrum, and expose the filter string and mZ/intensity vectors needed for y-/b-ion annotation against LGGNEQVTR.) — fgcz/rawrr
- **RawFileReader** (Vendor-provided .NET API wrapped by rawrr to access spectral data logged in the proprietary Thermo Fisher raw file underpinning the annotated scan.) — https://github.com/thermofisherlsms/RawFileReader
- **MsBackendRawFileReader** (Alternative MsBackend for the Bioconductor Spectra package built on rawrr accessors; optional route to the same spectral data for annotation within the Spectra framework.) — fgcz/MsBackendRawFileReader

## Examples

```
S <- rawrr::readSpectrum(rawfile, scan = 9594); S$scan[[1]]$filter
```

## Evaluation signals

- The scan's precursor filter string equals 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]' and the reported charge state is 2.
- Observed y-ion (and b-ion) fragment peaks align with the theoretical series computed for LGGNEQVTR at the scan's m/z values.
- All y-ion signals are several ten or even hundred folds above the local noise estimate, as shown for this scan in the article.
- Recorded MS-GF+ metrics match the reference values: Score 144, SpecProb 1.9e-12, DB E-Value 4.4e-4.
- The output CSV carries the full USI mzspec:MSV000086542:20181113_010_autoQC01:scan:9594:LGGNEQVTR/2 with all assignment fields populated.

## Limitations

- The rawrrSpectrum content partially depends on the instrument model and installed instrument control software version.
- On Windows, the decimal symbol must be configured as '.' for mZ and intensity vectors to have correct lengths.
- Exchangeable Spectra backends (e.g., MsBackendRawFileReader) were only planned, not yet implemented, and parallel computation support was absent at publication time.
- MS2 spectra in the example were acquired with HCD at NCE 27 and centroiding with lock mass correction; spectra generated under different fragmentation or profile-mode settings may not reproduce the same annotation quality.

## Evidence

- [results] assigned to the doubly-charged iRT peptide LGGNEQVTR by MS-GF+ (Score: 144, SpecProb: 1.9e-12, DB E-Value: 4.4e-4: [evidence span withheld — anchor status: not_found]
- [results] filter = "FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]": [evidence span withheld — anchor status: not_found]
