---
name: mz-decimal-precision-formatting
description: Use when a spectrum must be converted from a raw ion list or 'm/z:intensity'
  string into the canonical single-line form (integer intensities 0–100, 6-decimal
  m/z, zero-intensity ions kept) before hashing it to a SPLASH identifier.
when_to_use_negative:
- The spectrum is already a canonical SPLASH input string — re-normalizing could alter
  the hash if the source used different rounding
- You need the SPLASH identifier itself rather than the canonical spectrum text; use
  the SplashFactory/SplashUtil hashing step instead
- The goal is raw spectral data processing (peak picking, deconvolution) — this skill
  only standardizes the text encoding, not the underlying signal
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SplashUtil
  role: Java utility to directly SPLASH a spectrum in its single-line string representation,
    applying the canonical encoding rules before hashing
  repo: https://github.com/berlinguyinca/spectra-hash
- name: SplashFactory
  role: Creates the default SPLASHer used to splash a Spectrum object built from a
    list of Ion (m/z, intensity) pairs
  repo: https://github.com/berlinguyinca/spectra-hash
- name: spectra-hash validation tool
  role: Validation jar that checks canonical single-line spectral representations
    against the reference implementation across implementations
  repo: https://github.com/berlinguyinca/spectra-hash
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/nbt.3689
    title: SPLASH, a hashed identifier for mass spectra
  source_content_hash: sha256:7a81a513e84deb4a115c8259c11f20e3a0a748ddebbee5ee76f2b716936593cf
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/berlinguyinca/spectra-hash
  tools:
  - Java
  - Python
  - SplashFactory
  - SplashUtil
  - spectra-hash validation tool
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# mz-decimal-precision-formatting

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Canonicalizes a mass spectrum into the single-line 'm/z:intensity ion:intensity ...' text representation required before SPLASH block computation, fixing intensity normalization to integers 0–100, m/z rendering to exactly 6 decimal places, and ion ordering. Deterministic text encoding is what makes the subsequent SPLASH hash reproducible across MassBank, MoNA, GNPS, HMDB and other databases.

## When to use

Use when a spectrum must be converted from a raw ion list or 'm/z:intensity' string into the canonical single-line form (integer intensities 0–100, 6-decimal m/z, zero-intensity ions kept) before hashing it to a SPLASH identifier.

## When NOT to use

- The spectrum is already a canonical SPLASH input string — re-normalizing could alter the hash if the source used different rounding
- You need the SPLASH identifier itself rather than the canonical spectrum text; use the SplashFactory/SplashUtil hashing step instead
- The goal is raw spectral data processing (peak picking, deconvolution) — this skill only standardizes the text encoding, not the underlying signal

## Inputs

- Single-line spectrum string in 'ion:intensity ion:intensity ...' format (e.g. '138.0641:71.59 195.0815:261.7')
- List of ion (m/z, intensity) pairs from MassBank, MoNA, GNPS, HMDB or similar spectral library records

## Outputs

- Canonical spectrum text file with 6-decimal m/z fields, integer intensities 0–100, zero-intensity ions retained, N/A entries removed, and ions sorted ascending by m/z (ties broken by descending intensity), pairs joined by single spaces

## How to apply

Read the spectrum as ion (m/z, intensity) pairs from the single-line 'ion:intensity ion:intensity ...' string representation, as defined by the spectra-hash validation tool's spectral format. Normalize every intensity to an integer in the 0–100 range relative to the maximum peak, retaining zero-intensity entries but eliminating empty (N/A) values. Render each m/z value to exactly 6 decimal places (e.g. 138.0641 becomes '138.064100'). Sort the ions by ascending m/z, breaking ties by descending intensity, then join the pairs with single spaces into the output file. This canonical ordering and precision is essential because the SPLASH hash block is SHA256-derived from the canonical spectrum, so any difference in rounding or ordering changes the resulting identifier. The spectra-hash repo provides reference implementations in Java (SplashUtil/SplashFactory), C++, Python, C#, R, Ruby and Scala that encode these rules.

## Related tools

- **SplashUtil** (Java utility to directly SPLASH a spectrum in its single-line string representation, applying the canonical encoding rules before hashing) — https://github.com/berlinguyinca/spectra-hash
- **SplashFactory** (Creates the default SPLASHer used to splash a Spectrum object built from a list of Ion (m/z, intensity) pairs) — https://github.com/berlinguyinca/spectra-hash
- **spectra-hash validation tool** (Validation jar that checks canonical single-line spectral representations against the reference implementation across implementations) — https://github.com/berlinguyinca/spectra-hash

## Examples

```
java -jar validation/target/validation-1.8.jar -c -s 2 -t ms ./base-dataset/spectra/notsplashed/test-set-v1.csv base-dataset/spectra/test-set-with-splash-v1.csv
```

## Evaluation signals

- Output file exists and every ion field parses in the 'm/z:intensity' format
- All m/z values render with exactly 6 decimal places (e.g. '195.081500')
- All intensities are integers within the 0–100 range and no empty (N/A) entries remain
- Ions appear in ascending m/z order, with ties broken by descending intensity
- Hashing the canonical string with SplashUtil yields a SPLASH matching the reference implementation, verifiable via the validation jar (java -jar validation-1.8.jar -c -s 2 -t ms ...)

## Limitations

- Hash collisions observed in collision testing on 53.25 million spectra arose only for single-ion spectra of the same mass; theoretical collision probability is approximately 10^-31 for 10^9 spectra
- The canonical representation is defined for MS-type spectra ('t ms'); the validation tool's spectrum-type flag must match the data
- Zero-intensity entries must be retained (not silently dropped), otherwise the canonical spectrum — and hence the SPLASH — will not match the reference implementation

## Evidence

- [methods] The canonical representation normalizes intensities to integers 0-100, renders m/z to exactly 6 decimal places, retains zero-intensity entries, eliminates empty values, and sorts ions ascending by m/z then descending by intensity: [evidence span withheld — anchor status: not_found]
