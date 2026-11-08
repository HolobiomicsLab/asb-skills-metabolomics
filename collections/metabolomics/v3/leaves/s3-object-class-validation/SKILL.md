---
name: s3-object-class-validation
description: Use when rawrr::readSpectrum or rawrr::readChromatogram has returned
  objects from a Thermo Fisher .raw file and you must confirm their class (rawrrSpectrum,
  rawrrSpectrumSet, rawrrChromatogram(Set)) and internal integrity before analysis
  or reporting.
when_to_use_negative:
- Input is not a proprietary Thermo Fisher .raw file (e.g. data already in mzML or
  an mzR-backed object) — open-format readers handle those directly without rawrr
- Objects are already Spectra-backed via MsBackendRawFileReader, where accessors are
  checked by the Spectra package's backend contract instead of rawrr S3 validators
- You only need vendor API access at the C#/.NET level (RawFileReader) without constructing
  or validating R objects
edam_operation: http://edamontology.org/operation_3454
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: rawrr/rawR
  role: Provides readSpectrum/readChromatogram to construct the S3 objects and is.rawrrSpectrum/is.rawrrChromatogram/validate_rawrrSpectrum
    to validate them
  repo: https://github.com/fgcz/rawR
- name: R
  role: Runtime environment in which the rawrr S3 objects are constructed and class-validated
- name: RawFileReader
  role: Vendor .NET API wrapped by rawrr that extracts spectral and chromatographic
    data from the .raw file before S3 object reconstruction in R
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: tartare
  role: Bioconductor data package supplying example raw files for demonstration and
    extended testing of object construction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2020.10.30.362533
    title: rawR - Direct access to raw mass spectrometry data in R
  source_content_hash: sha256:a08e156e58c59eb30e77327b6b4b5b6361ed993bf64e2edfe74a15c95916c9a9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fgcz/rawR
  tools:
  - R
  - RawFileReader
  - rawR
  - rawrr/rawR
  - tartare
  edam_operation: http://edamontology.org/operation_3454
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# s3-object-class-validation

## Summary

Validates that rawrr S3 objects (rawrrSpectrum, rawrrChromatogram) returned by readSpectrum()/readChromatogram() are correctly constructed before downstream use, using is.rawrrSpectrum/is.rawrrChromatogram class checks and validate_rawrrSpectrum integrity checks on data read from Thermo Fisher .raw files.

## When to use

Use when rawrr::readSpectrum or rawrr::readChromatogram has returned objects from a Thermo Fisher .raw file and you must confirm their class (rawrrSpectrum, rawrrSpectrumSet, rawrrChromatogram(Set)) and internal integrity before analysis or reporting.

## When NOT to use

- Input is not a proprietary Thermo Fisher .raw file (e.g. data already in mzML or an mzR-backed object) — open-format readers handle those directly without rawrr
- Objects are already Spectra-backed via MsBackendRawFileReader, where accessors are checked by the Spectra package's backend contract instead of rawrr S3 validators
- You only need vendor API access at the C#/.NET level (RawFileReader) without constructing or validating R objects

## Inputs

- Thermo Fisher Scientific .raw file (e.g. 20181113_010_autoQC01.raw from MassIVE MSV000086542 or ExperimentHub EH4547)
- Installed rawrr executable runtime (rawrr::installRawrrExe())
- Scan indices (e.g. scan 9594) and target m/z masses with 10 ppm tolerance for XIC extraction

## Outputs

- rawrrSpectrumSet / rawrrSpectrum objects from readSpectrum()
- rawrrChromatogramSet / rawrrChromatogram objects from readChromatogram(type='xic')
- Boolean results from is.rawrrSpectrum / is.rawrrChromatogram and validate_rawrrSpectrum integrity output, written to validation_summary.txt

## How to apply

After fetching the raw file (e.g. 20181113_010_autoQC01.raw from MassIVE MSV000086542 or via ExperimentHub EH4547, MD5 a1f5df9627cf9e0d51ec1906776957ab) and installing the runtime with rawrr::installRawrrExe(), call rawrr::readSpectrum on a chosen scan (e.g. scan 9594, the doubly-charged iRT peptide LGGNEQVTR/2) and rawrr::readChromatogram with 10 ppm tolerance around target masses restricted to MS1-level scans (filter = 'ms'). readSpectrum returns a rawrrSpectrumSet and readChromatogram(type='xic') returns a rawrrChromatogramSet; extract individual elements and verify class with is.rawrrSpectrum and is.rawrrChromatogram, and run validate_rawrrSpectrum to check object integrity (mass and intensity vectors must have equal, non-zero lengths). Record class-check results and object summaries to a validation summary file. Rationale: rawR exchanges data between R and the C# RawFileReader layer via file I/O, so validating the reconstructed S3 objects guards against silent corruption in the R/.NET interchange.

## Related tools

- **rawrr/rawR** (Provides readSpectrum/readChromatogram to construct the S3 objects and is.rawrrSpectrum/is.rawrrChromatogram/validate_rawrrSpectrum to validate them) — https://github.com/fgcz/rawR
- **R** (Runtime environment in which the rawrr S3 objects are constructed and class-validated)
- **RawFileReader** (Vendor .NET API wrapped by rawrr that extracts spectral and chromatographic data from the .raw file before S3 object reconstruction in R) — https://github.com/thermofisherlsms/RawFileReader
- **tartare** (Bioconductor data package supplying example raw files for demonstration and extended testing of object construction)

## Examples

```
S <- rawrr::readSpectrum(rawfile, 9594); stopifnot(is.rawrrSpectrum(S[[1]])); validate_rawrrSpectrum(S[[1]]); xic <- rawrr::readChromatogram(rawfile, masses, type='xic', tolppm=10, filter='ms')
```

## Evaluation signals

- is.rawrrSpectrum(S[[9594]]) and is.rawrrChromatogram(xic[[i]]) return TRUE for the extracted elements of the returned sets
- readSpectrum returns an object of class rawrrSpectrumSet and readChromatogram(type='xic') returns a rawrrChromatogramSet as the paper reports
- validate_rawrrSpectrum passes without error — mass and intensity vectors have matching, non-zero lengths (a failure mode seen on Windows with wrong decimal symbol settings)
- The source file MD5 matches a1f5df9627cf9e0d51ec1906776957ab before validation is attempted
- Scan 9594 summary matches the expected assignment (LGGNEQVTR/2, MS-GF+ Score 144, SpecProb 1.9e-12)

## Limitations

- rawrrSpectrum content partially depends on the instrument model and installed instrument control software version
- On Windows, the decimal symbol must be configured as '.' for mZ and intensity vectors to have correct lengths
- rawrr.exe must be present — install via rawrr::installRawrrExe(); data exchange between R and C# layers uses file I/O, which can fail silently if not validated

## Evidence

- [methods] Use rawrr::readSpectrum on the raw file to build a rawrrSpectrumSet, selecting the scan of interest: [evidence span withheld — anchor status: not_found]
