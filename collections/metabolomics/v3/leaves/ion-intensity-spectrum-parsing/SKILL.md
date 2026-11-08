---
name: ion-intensity-spectrum-parsing
description: Use when input records are raw mass spectra (m/z / intensity pairs) that
  must be converted into the ion:intensity single-line format or Spectrum objects
  before SPLASH generation or similarity work.
when_to_use_negative:
- Input is already a SPLASH string or hash block — parsing is upstream of SPLASH generation,
  not applicable
- Input is raw instrument data (e.g. mzML/NetCDF profiles) requiring peak picking
  to centroided ion-intensity pairs before this representation applies
- Spectra that are not mass spectra (SpectraType other than MS) — the default SPLASHer
  flags here target MS spectra ('-t ms')
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Java reference implementation (edu.ucdavis.fiehnlab.splash)
  role: Build SpectrumImpl objects from Ion lists and generate SPLASHes via SplashFactory/SplashUtil
  repo: https://github.com/berlinguyinca/spectra-hash
- name: SPLASH validation tool
  role: Consumes parsed single-line spectrum columns (-s) and validates generated
    SPLASHes against the reference implementation
  repo: https://github.com/berlinguyinca/spectra-hash
- name: MoNA
  role: Source of the openly deposited spectra subset to parse (all but the NIST spectra)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/nbt.3689
    title: SPLASH, a hashed identifier for mass spectra
  source_content_hash: sha256:41406d106196ed16bc319b315d9df493e4b49e15631af90f654b4703fb83bfd7
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/berlinguyinca/spectra-hash
  tools:
  - Java
  - Java reference implementation (edu.ucdavis.fiehnlab.splash)
  - MoNA
  - Python
  - SPLASH validation tool
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# ion-intensity-spectrum-parsing

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Parse raw mass spectra records into ion-intensity pairs and build Spectrum objects (SpectraType.MS), in the SPLASH single-line 'ion:intensity ion:intensity ...' format, so that hashed identifiers (SPLASHes) can be generated downstream.

## When to use

Use when input records are raw mass spectra (m/z / intensity pairs) that must be converted into the ion:intensity single-line format or Spectrum objects before SPLASH generation or similarity work.

## When NOT to use

- Input is already a SPLASH string or hash block — parsing is upstream of SPLASH generation, not applicable
- Input is raw instrument data (e.g. mzML/NetCDF profiles) requiring peak picking to centroided ion-intensity pairs before this representation applies
- Spectra that are not mass spectra (SpectraType other than MS) — the default SPLASHer flags here target MS spectra ('-t ms')

## Inputs

- Mass spectra records containing m/z and intensity values (e.g. MoNA/MassBank/GNPS/HMDB/ReSpect/FiehnLib entries; NIST 14 subset excluded as non-public)
- Single-line spectrum strings in 'ion:intensity ion:intensity ...' format (e.g. CSV columns consumed by the SPLASH validation tool)

## Outputs

- One Spectrum object per record (SpectraType.MS) built from Ion intensity pairs
- Single-line spectrum strings keyed by spectrum ID, ready for SplashFactory.splashIt / SplashUtil.splash

## How to apply

Extract the ion (m/z) and intensity values from each record and represent every spectrum as a single-line string of 'ion:intensity ion:intensity ...' pairs, which is the format the SPLASH validation tool and SplashUtil accept. For the Java reference implementation, build a SpectrumImpl from a list of Ion objects with SpectraType.MS and pass it to SplashFactory.splashIt, or shortcut with SplashUtil.splash on the string representation; Python, R, C++, C#, Scala, Ruby and JavaScript implementations exist in the same repository if Java is unavailable. Apply this parsing to every record of the target spectral library (e.g. the 563,902-spectrum MoNA-accessible subset from MassBank, GNPS, HMDB, ReSpect, FiehnLib and NIST 14, excluding the non-public NIST spectra). Record each parsed spectrum with a stable spectrum ID so parsed objects can be joined back to SPLASHes and collision groups in later steps. The caffeine example (138.0641:71.59 195.0815:261.7, MassBank PR100026) serves as a canonical test input for the parser.

## Related tools

- **Java reference implementation (edu.ucdavis.fiehnlab.splash)** (Build SpectrumImpl objects from Ion lists and generate SPLASHes via SplashFactory/SplashUtil) — https://github.com/berlinguyinca/spectra-hash
- **SPLASH validation tool** (Consumes parsed single-line spectrum columns (-s) and validates generated SPLASHes against the reference implementation) — https://github.com/berlinguyinca/spectra-hash
- **MoNA** (Source of the openly deposited spectra subset to parse (all but the NIST spectra))

## Examples

```
String splash = SplashUtil.splash("138.0641:71.59 195.0815:261.7", SpectraType.MS);
```

## Evaluation signals

- The caffeine reference spectrum PR100026 (138.0641:71.59 195.0815:261.7) parses and SPLASHes to splash10-0002-0900000000-b112e4e059e1ecf98c5f
- Every parsed spectrum carries a spectrum ID that can be joined back to library entries (e.g. MassBank PR100026, HMDB 5464, GNPS CCMSLIB00000222858)
- Validation tool run (java -jar validation/target/validation-1.8.jar -c -s 2 -t ms) reports no mismatch between generated SPLASHes and the reference implementation
- Parsed record count matches the expected library subset (563,902 spectra in the paper's development dataset; 53.25 million in the collision test)

## Limitations

- The NIST 14 portion of the 563,902-spectrum dataset cannot be released publicly, so only MoNA-accessible spectra are parseable in an open reproducible run
- Not all spectra in the validation set are annotated with structures, limiting downstream structural interpretation
- The single-line 'ion:intensity' string format must be preserved exactly (spaces between pairs, colon separator); validation flags -s/-k/-o/-t/-T define column positions and spectrum type

## Evidence
