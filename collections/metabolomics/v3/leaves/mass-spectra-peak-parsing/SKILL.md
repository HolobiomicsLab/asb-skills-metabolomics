---
name: mass-spectra-peak-parsing
description: Use when a mass spectrum's peaks must be converted from a tabular or
  database record form (e.g. a MassBank record like PR100026) into the single-line
  'ion:intensity' string format that SPLASH tools expect.
when_to_use_negative:
- The input is already a single-line 'ion:intensity' spectrum string — feed it directly
  to SplashUtil.splash() instead of re-parsing
- The input is a chemical structure (InChI/SMILES) or raw instrument data (mzML) for
  which SPLASH only defines identifier generation after spectrum extraction, not structure
  hashing
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: spectra-hash (Java reference implementation)
  role: Consumes the parsed 'ion:intensity' string via SplashUtil.splash("138.0641:71.59
    195.0815:261.7", SpectraType.MS) to generate the SPLASH
  repo: https://github.com/berlinguyinca/spectra-hash
- name: SPLASH validation tool
  role: Validates that spectra parsed and splashed by an external tool match the reference
    implementation, using column flags -k, -o, -s, -t
  repo: https://github.com/berlinguyinca/spectra-hash
- name: Python API
  role: Alternative implementation for parsing peaks and generating SPLASH in Python,
    part of the spectra-hash repository
  repo: https://github.com/berlinguyinca/spectra-hash
- name: MassBank
  role: Source database of the example caffeine spectrum record PR100026 whose peak
    list is parsed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/nbt.3689
    title: SPLASH, a hashed identifier for mass spectra
  source_content_hash: sha256:16a5e6753e358483578b313839b8add9069dc81f42d74356e2f69356bfeba48b
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/berlinguyinca/spectra-hash
  tools:
  - Java
  - MassBank
  - Python
  - Python API
  - SPLASH validation tool
  - spectra-hash (Java reference implementation)
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# mass-spectra-peak-parsing

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Parses a mass spectrum's peak list into the single-line 'ion:intensity' representation required by the SPLASH (SPectraL hASH) algorithm, e.g. '138.0641:71.59 195.0815:261.7' for the caffeine spectrum MassBank PR100026. This canonical textual form is the input that all SPLASH reference implementations consume before block construction and SHA-256 hashing.

## When to use

Use when a mass spectrum's peaks must be converted from a tabular or database record form (e.g. a MassBank record like PR100026) into the single-line 'ion:intensity' string format that SPLASH tools expect.

## When NOT to use

- The input is already a single-line 'ion:intensity' spectrum string — feed it directly to SplashUtil.splash() instead of re-parsing
- The input is a chemical structure (InChI/SMILES) or raw instrument data (mzML) for which SPLASH only defines identifier generation after spectrum extraction, not structure hashing

## Inputs

- MassBank spectrum record (e.g. PR100026) or any peak list of m/z/intensity pairs
- Peak table with columns m/z and intensity (e.g. 138.0641 / 71.59, 195.0815 / 261.7)
- CSV file of spectra in tabular form for batch conversion (test-set-v1.csv style, with column positions given via -k/-o/-s/-t flags)

## Outputs

- Single-line spectrum string in 'ion:intensity ion:intensity ...' format (e.g. '138.0641:71.59 195.0815:261.7')
- SPLASH identifier derived from the parsed string (e.g. splash10-0002-0900000000-b112e4e059e1ecf98c5f for caffeine PR100026)

## How to apply

Retrieve the peak list from the spectral record (e.g. the MassBank caffeine spectrum PR100026 with m/z 138.0641 at intensity 71.59 and m/z 195.0815 at intensity 261.7) and flatten it into space-separated 'ion:intensity' pairs on a single line. Preserve the m/z values exactly as published, since the SPLASH summary blocks and the fourth SHA-256 hash block depend on the precise canonicalized peak representation; any rounding or reordering error changes the final identifier. Feed the resulting string directly to a SPLASH implementation — in Java via SplashUtil.splash() or by constructing a Spectrum object from Ion pairs and calling SplashFactory.splashIt(); in Python via the repository's Python API. Use the provided validation tool (java -jar validation/target/validation-1.8.jar with flags -k, -o, -s, -t) to check that spectra serialized this way produce identical SPLASHes to the reference implementation.

## Related tools

- **spectra-hash (Java reference implementation)** (Consumes the parsed 'ion:intensity' string via SplashUtil.splash("138.0641:71.59 195.0815:261.7", SpectraType.MS) to generate the SPLASH) — https://github.com/berlinguyinca/spectra-hash
- **SPLASH validation tool** (Validates that spectra parsed and splashed by an external tool match the reference implementation, using column flags -k, -o, -s, -t) — https://github.com/berlinguyinca/spectra-hash
- **Python API** (Alternative implementation for parsing peaks and generating SPLASH in Python, part of the spectra-hash repository) — https://github.com/berlinguyinca/spectra-hash
- **MassBank** (Source database of the example caffeine spectrum record PR100026 whose peak list is parsed)

## Examples

```
String splash = SplashUtil.splash("138.0641:71.59 195.0815:261.7", SpectraType.MS);
```

## Evaluation signals

- Parsing the PR100026 caffeine peaks (138.0641:71.59 195.0815:261.7) and splashing them must yield splash10-0002-0900000000-b112e4e059e1ecf98c5f byte-for-byte
- The single-line representation must contain exactly one space-separated 'ion:intensity' pair per peak with no loss of m/z decimal precision
- Running the validation tool (java -jar validation/target/validation-1.8.jar -c -s 2 -t ms) over a converted test-set CSV must report zero mismatches against the reference SPLASHes
- Identical spectra from different databases (MassBank PR100026, HMDB 5464, GNPS CCMSLIB00000222858) must parse to strings yielding the same SPLASH

## Limitations

- Identical SPLASHes can arise legitimately from mass spectra containing a single ion of the same mass, so single-peak spectra are not uniquely resolved by the hash
- Serialization must match the reference exactly — the README specifies the format as 'ion:intensity ion:intensity ...' and deviations in ordering or formatting break SPLASH agreement

## Evidence
