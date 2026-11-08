---
name: spectrum-string-parsing
description: Use when the input is a mass spectrum given as a single-line 'm/z:intensity
  ion:intensity ...' string (e.g. MassBank PR100026-style records) that must be canonicalized
  before SPLASH generation or validation.
when_to_use_negative:
- The input spectrum is already in a structured format (e.g. mzML or a list of Ion
  objects in the Java API) and does not need text canonicalization
- The goal is only to hash an already-canonical string — call SplashUtil.splash directly
  on the string rather than re-parsing it
- The spectrum contains only N/A/empty intensity values for all ions, leaving nothing
  to normalize
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Java (SplashUtil / SplashFactory)
  role: 'Reference implementation: SplashUtil.splash parses the single-line spectrum
    string directly and emits the SPLASH; SpectrumImpl/Ion objects build a Spectrum
    from parsed ion-intensity pairs'
  repo: https://github.com/berlinguyinca/spectra-hash
- name: Python
  role: Alternative reference implementation used to parse and canonicalize the ion:intensity
    string and compute SPLASH blocks
  repo: https://github.com/berlinguyinca/spectra-hash
- name: validation-1.8.jar
  role: Validation tool that SPLASHes spectra from a CSV column in single-line string
    representation and checks consistency against the reference implementation
  repo: https://github.com/berlinguyinca/spectra-hash
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/nbt.3689
    title: SPLASH, a hashed identifier for mass spectra
  source_content_hash: sha256:7d2ddcda09714658f877f4179f744b1888a3b5c93441c9409df3c6d6fcf96bb1
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/berlinguyinca/spectra-hash
  tools:
  - Java
  - Java (SplashUtil / SplashFactory)
  - Python
  - SplashUtil
  - validation-1.8.jar
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# spectrum-string-parsing

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Parses and canonicalizes a mass spectrum encoded in the SPLASH single-line 'ion:intensity ion:intensity ...' text representation — normalizing intensities to integers 0-100, rendering m/z to exactly 6 decimal places, and sorting ions — so the string can be fed to SPLASH block computation.

## When to use

Use when the input is a mass spectrum given as a single-line 'm/z:intensity ion:intensity ...' string (e.g. MassBank PR100026-style records) that must be canonicalized before SPLASH generation or validation.

## When NOT to use

- The input spectrum is already in a structured format (e.g. mzML or a list of Ion objects in the Java API) and does not need text canonicalization
- The goal is only to hash an already-canonical string — call SplashUtil.splash directly on the string rather than re-parsing it
- The spectrum contains only N/A/empty intensity values for all ions, leaving nothing to normalize

## Inputs

- Mass spectrum as a single-line 'm/z:intensity ion:intensity ...' string (e.g. '138.0641:71.59 195.0815:261.7' for caffeine PR100026)
- Spectrum type identifier (e.g. SpectraType.MS)

## Outputs

- Canonical spectrum text file with ion pairs joined by single spaces: 6-decimal m/z fields, integer intensities 0-100, zero-intensity ions retained, no N/A entries, ions sorted by ascending m/z then descending intensity
- Parsed list of (m/z, intensity) ion pairs suitable for downstream SPLASH block computation

## How to apply

Read the input as a list of ion (m/z, intensity) pairs from the single-line 'm/z:intensity ion:intensity ...' string. Normalize every intensity to the integer range 0-100, keeping zero-intensity ions and dropping empty (N/A) values. Render each m/z to exactly 6 decimal places, then sort ions by ascending m/z, breaking ties by descending intensity. Re-serialize the canonical ions back into the single-line format, joining pairs with single spaces, and write the canonical spectrum text file. This canonical form is the required pre-processing step before computing SPLASH summary blocks (e.g. the 100-Da-binned third block) and the SHA256-derived hash block.

## Related tools

- **Java (SplashUtil / SplashFactory)** (Reference implementation: SplashUtil.splash parses the single-line spectrum string directly and emits the SPLASH; SpectrumImpl/Ion objects build a Spectrum from parsed ion-intensity pairs) — https://github.com/berlinguyinca/spectra-hash
- **Python** (Alternative reference implementation used to parse and canonicalize the ion:intensity string and compute SPLASH blocks) — https://github.com/berlinguyinca/spectra-hash
- **validation-1.8.jar** (Validation tool that SPLASHes spectra from a CSV column in single-line string representation and checks consistency against the reference implementation) — https://github.com/berlinguyinca/spectra-hash

## Examples

```
String splash = SplashUtil.splash("138.0641:71.59 195.0815:261.7", SpectraType.MS);
```

## Evaluation signals

- The output file exists and is parseable in the 'm/z:intensity ion:intensity ...' format
- Every m/z field is rendered to exactly 6 decimal places
- All intensities are integers within the 0-100 range with no N/A/empty entries retained
- Ions appear in ascending m/z order, with ties broken by descending intensity
- Feeding the canonical string through SplashUtil or SplashFactory reproduces the published SPLASH for the caffeine spectrum PR100026 (e.g. splash10-0002-0900000000-b112e4e059e1ecf98c5f)

## Limitations

- Identical SPLASHes arose from mass spectra containing a single ion of the same mass, so single-ion spectra are inherently ambiguous after parsing and hashing
- The format for single-line string spectral representation is 'ion:intensity ion:intensity ...' — spectra not in this exact text form must be converted first
- The reference Java implementation requires Java 1.8

## Evidence

- [methods] Canonical representation rules: [evidence span withheld — anchor status: not_found]
