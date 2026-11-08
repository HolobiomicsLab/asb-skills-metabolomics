---
name: spectral-intensity-normalization
description: Use when a raw mass spectrum given as ion (m/z, intensity) pairs must
  be serialized into the canonical 'ion:intensity' string format that the SPLASH reference
  implementation hashes, e.g. MassBank/MoNA/GNPS records like PR100026.
when_to_use_negative:
- Input already carries a SPLASH identifier or is already in canonical 6-decimal/0-100
  integer form — re-canonicalization is unnecessary
- Task is chemical-structure identification rather than spectral identification —
  use InChIKey-style structure identifiers, not this spectrum text encoding
- Downstream analysis requires raw (unscaled) absolute intensities, since this filter
  replaces them with normalized 0-100 integers
tools:
- name: Java
  role: Reference implementation language hosting SpectrumImpl, Ion, SplashFactory
    and SplashUtil for building the canonical Spectrum and hashing it
  repo: https://github.com/berlinguyinca/spectra-hash
- name: Python
  role: One of the additional language implementations available for applying the
    same canonicalization/splashing scheme
  repo: https://github.com/berlinguyinca/spectra-hash
- name: SplashUtil
  role: Utility to directly SPLASH a spectrum already in the canonical 'm/z:intensity'
    string representation without building a Spectrum object
  repo: https://github.com/berlinguyinca/spectra-hash
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/nbt.3689
    title: SPLASH, a hashed identifier for mass spectra
  source_content_hash: sha256:2902a6db66b08d6144ce9ac6c2569179a88ab5e362f3b3102bd91871d0cc7ab0
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/berlinguyinca/spectra-hash
  tools:
  - Java
  - Python
  - SplashUtil
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# spectral-intensity-normalization

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Canonicalizes a mass spectrum into the single-line 'm/z:intensity m/z:intensity ...' text representation required before SPLASH block computation: integer intensities 0-100, 6-decimal m/z, retained zero-intensity ions, dropped N/A entries, and ascending m/z ordering.

## When to use

Use when a raw mass spectrum given as ion (m/z, intensity) pairs must be serialized into the canonical 'ion:intensity' string format that the SPLASH reference implementation hashes, e.g. MassBank/MoNA/GNPS records like PR100026.

## When NOT to use

- Input already carries a SPLASH identifier or is already in canonical 6-decimal/0-100 integer form — re-canonicalization is unnecessary
- Task is chemical-structure identification rather than spectral identification — use InChIKey-style structure identifiers, not this spectrum text encoding
- Downstream analysis requires raw (unscaled) absolute intensities, since this filter replaces them with normalized 0-100 integers

## Inputs

- Mass spectrum as list of ion (m/z, intensity) pairs
- Single-line 'm/z:intensity ion:intensity ...' spectrum string (e.g. '138.0641:71.59 195.0815:261.7')
- CSV file of spectra in single-line string representation (e.g. test-set-v1.csv with a spectrum column passed via -s)

## Outputs

- Canonical spectrum text file with space-separated 'm/z:intensity' ion pairs, 6-decimal m/z, integer 0-100 intensities, ascending m/z order
- Canonicalized Spectrum object consumable by SplashFactory/SplashUtil for SPLASH generation

## How to apply

Read the input as a list of ion (m/z, intensity) pairs from the single-line 'm/z:intensity' string representation (e.g. '138.0641:71.59 195.0815:261.7' for the caffeine spectrum PR100026). Normalize every intensity to an integer in the 0-100 range, deliberately keeping zero-intensity ions while dropping empty (N/A) values. Render each m/z to exactly 6 decimal places. Sort ions by ascending m/z, breaking ties by descending intensity, then re-serialize as 'm/z:intensity m/z:intensity ...' joined by single spaces. This canonical text form is the input the Java SpectrumImpl/SplashFactory pipeline (or SplashUtil on the raw string) consumes to generate the SPLASH hash.

## Related tools

- **Java** (Reference implementation language hosting SpectrumImpl, Ion, SplashFactory and SplashUtil for building the canonical Spectrum and hashing it) — https://github.com/berlinguyinca/spectra-hash
- **Python** (One of the additional language implementations available for applying the same canonicalization/splashing scheme) — https://github.com/berlinguyinca/spectra-hash
- **SplashUtil** (Utility to directly SPLASH a spectrum already in the canonical 'm/z:intensity' string representation without building a Spectrum object) — https://github.com/berlinguyinca/spectra-hash

## Examples

```
String splash = SplashUtil.splash("138.0641:71.59 195.0815:261.7", SpectraType.MS);
```

## Evaluation signals

- Output file is parseable in the 'ion:intensity ion:intensity ...' format
- Every m/z field is rendered with exactly 6 decimal places
- All intensities are integers within the range 0-100
- No empty (N/A) entries remain in the serialized spectrum
- Ions appear in ascending m/z order (ties broken by descending intensity), and the canonical string yields a SPLASH identical to the reference implementation via the validation tool

## Limitations

- Zero-intensity entries are retained rather than filtered, which changes the hashed input relative to spectra that silently omit them
- Hash collision testing on 53.25 million spectra showed identical SPLASHes only arise from single-ion spectra of the same mass, so canonicalization cannot distinguish spectra containing a lone ion of identical mass
- Consistency across the many language implementations (Java, Python, C++, C#, R, Ruby, Scala) is not guaranteed without running the provided validation tool

## Evidence
