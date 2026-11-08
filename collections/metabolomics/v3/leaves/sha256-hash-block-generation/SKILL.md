---
name: sha256-hash-block-generation
description: Use when a mass spectrum in 'ion:intensity ion:intensity' single-line
  form must receive a stable, database-independent SPLASH identifier, or when verifying
  that records such as MassBank PR100026, HMDB 5464 and GNPS CCMSLIB00000222858 refer
  to the same caffeine spectrum.
when_to_use_negative:
- The spectrum already has a SPLASH and only the summary blocks need inspection —
  SPLASH was designed as a full four-block identifier, and isolating the hash block
  for non-spectral metadata would be out of scope.
- The input is a chemical structure rather than a mass spectrum — the paper positions
  SPLASH as the spectral analogue of the InChIKey, not as a structure hash.
- You need in-spectrum similarity search rather than identity — the SPLASH hash block
  guarantees collisions only for identical spectra, not for near neighbours.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: SplashUtil
  role: Java utility that directly SPLASHes a spectrum given as a single-line 'ion:intensity
    ion:intensity' string, producing the full four-block SPLASH including the SHA-256
    hash block
  repo: https://github.com/berlinguyinca/spectra-hash
- name: SplashFactory
  role: Java factory returning the default SPLASHer used to SPLASH a programmatically
    constructed Spectrum object of Ion pairs
  repo: https://github.com/berlinguyinca/spectra-hash
- name: validation tool
  role: Java validation JAR (validation-1.8.jar) used to verify that an implementation
    produces SPLASHes consistent with the reference implementation
  repo: https://github.com/berlinguyinca/spectra-hash
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/nbt.3689
    title: SPLASH, a hashed identifier for mass spectra
  source_content_hash: sha256:da518e235b7be541b278a971e1d63cf266d6b69823932c2845356229adc25a29
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/berlinguyinca/spectra-hash
  tools:
  - Java
  - Python
  - SplashFactory
  - SplashUtil
  - validation tool
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# sha256-hash-block-generation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Generates the fourth, unique hash block of a SPLASH identifier by applying the SHA-256 algorithm to a canonicalized mass spectrum so that identical spectra across MassBank, MoNA, GNPS, HMDB and mzCloud collapse to the same database-independent identifier.

## When to use

Use when a mass spectrum in 'ion:intensity ion:intensity' single-line form must receive a stable, database-independent SPLASH identifier, or when verifying that records such as MassBank PR100026, HMDB 5464 and GNPS CCMSLIB00000222858 refer to the same caffeine spectrum.

## When NOT to use

- The spectrum already has a SPLASH and only the summary blocks need inspection — SPLASH was designed as a full four-block identifier, and isolating the hash block for non-spectral metadata would be out of scope.
- The input is a chemical structure rather than a mass spectrum — the paper positions SPLASH as the spectral analogue of the InChIKey, not as a structure hash.
- You need in-spectrum similarity search rather than identity — the SPLASH hash block guarantees collisions only for identical spectra, not for near neighbours.

## Inputs

- Mass spectrum in single-line string representation 'ion:intensity ion:intensity ...' (e.g. 138.0641:71.59 195.0815:261.7 for MassBank caffeine record PR100026)
- Spectrum type flag (SpectraType.MS for the published mass-spectrometry use case)

## Outputs

- Full SPLASH identifier string with four dash-separated blocks, e.g. splash10-0002-0900000000-b112e4e059e1ecf98c5f for the PR100026 caffeine spectrum
- Fourth SHA-256-derived hash block (last block of the SPLASH string)

## How to apply

Build a Spectrum object from ion-intensity pairs (e.g. 138.0641:71.59 and 195.0815:261.7 for the caffeine spectrum PR100026) and pass it through SplashFactory or, more directly, SplashUtil.splash("138.0641:71.59 195.0815:261.7", SpectraType.MS). The full SPLASH string is four dash-separated blocks; the first block is the SPLASH10 identifier, the second and third are summary blocks derived from the spectrum's canonicalized peak list, and the fourth is the SHA-256-based hash block. Because the reference implementation (Java core 1.8) is the authority, validate any locally produced hash block against the published string splash10-0002-0900000000-b112e4e059e1ecf98c5f for PR100026, or against the reference Java tool via the bundled validation JAR before trusting a non-reference implementation.

## Related tools

- **SplashUtil** (Java utility that directly SPLASHes a spectrum given as a single-line 'ion:intensity ion:intensity' string, producing the full four-block SPLASH including the SHA-256 hash block) — https://github.com/berlinguyinca/spectra-hash
- **SplashFactory** (Java factory returning the default SPLASHer used to SPLASH a programmatically constructed Spectrum object of Ion pairs) — https://github.com/berlinguyinca/spectra-hash
- **validation tool** (Java validation JAR (validation-1.8.jar) used to verify that an implementation produces SPLASHes consistent with the reference implementation) — https://github.com/berlinguyinca/spectra-hash

## Examples

```
String splash = SplashUtil.splash("138.0641:71.59 195.0815:261.7", SpectraType.MS);
```

## Evaluation signals

- For the MassBank caffeine spectrum PR100026 the computed full SPLASH must byte-for-byte match splash10-0002-0900000000-b112e4e059e1ecf98c5f.
- The output must contain exactly four dash-separated blocks, with the first block equal to splash10 (SPLASH10 identifier) and the last block being the SHA-256-derived hash block.
- When run on the reference implementation's own test set (e.g. base-dataset/spectra/notsplashed/test-set-v1.csv against test-set-with-splash-v1.csv via the validation JAR), no mismatches should be reported.
- Running the same non-reference implementation (Python, C++, C#, R, Ruby, Scala) and the reference Java core on identical ion:intensity input must yield identical SPLASH strings.
- Identical SPLASHes must only arise from spectra containing the same ions; per the paper's collision test on 53.25 million spectra, identical SPLASHes from single-ion spectra of the same mass are the only expected collisions (theoretical collision probability ≈ 10^-31 for a database of 10^9 spectra).

## Limitations

- The paper's collision testing on 53.25 million spectra found that identical SPLASHes only arose from mass spectra containing a single ion of the same mass — meaning single-peak spectra are inherently ambiguous.
- Theoretical collision probability is approximately 10^-31 for a database of 10^9 spectra, so hash collisions are practically negligible but not mathematically impossible.
- Not all spectra in the validation set are annotated with structures, so the hash block cannot be used to resolve structural identity — it is only a spectral identity hash.
- The reference implementation is Java (requires Java 1.8); other language ports (Python, C++, C#, R, Ruby, Scala) must be validated against the Java reference via the validation tool before being trusted.

## Evidence
