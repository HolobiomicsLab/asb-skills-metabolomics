---
name: ion-list-sorting-and-canonicalization
description: Use when a raw spectrum (e.g. a MassBank PR100026-style ion list such
  as '138.0641:71.59 195.0815:261.7') must be converted into the deterministic canonical
  ion:intensity string that feeds SPLASH hashing, similarity search, or cross-database
  comparison.
when_to_use_negative:
- The input is already a canonicalized spectrum file validated against the reference
  implementation — re-canonicalizing is unnecessary and may mask upstream defects
- The target is a structure or metabolite identifier (InChIKey-style); this skill
  only canonicalizes spectral ion lists, not chemical structures
- The spectrum contains annotations or metadata that must be preserved; canonicalization
  drops empty (N/A) entries and keeps only ion-intensity content
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SplashUtil
  role: Provides the direct 'm/z:intensity ion:intensity ...' string interface that
    consumes the canonicalized spectrum and produces the SPLASH
  repo: https://github.com/berlinguyinca/spectra-hash
- name: SplashFactory
  role: Creates the default SPLASHer; used on a Spectrum object built from the canonicalized
    ion-intensity pairs
  repo: https://github.com/berlinguyinca/spectra-hash
- name: validation-1.8.jar
  role: Validation tool that checks a canonical spectrum column against the reference
    implementation to ensure consistent SPLASH generation
  repo: https://github.com/berlinguyinca/spectra-hash
- name: Java (spectra-hash reference implementation)
  role: Reference implementation (core-1.8.jar) containing the Spectrum, Ion, and
    canonicalization logic
  repo: https://github.com/berlinguyinca/spectra-hash
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/nbt.3689
    title: SPLASH, a hashed identifier for mass spectra
  source_content_hash: sha256:99f66ea81d37642b510e3f2ee8a4a66ce50acfbb0e45bd6d8d6a886d5eb62467
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/berlinguyinca/spectra-hash
  tools:
  - Java
  - Java (spectra-hash reference implementation)
  - Python
  - SplashFactory
  - SplashUtil
  - validation-1.8.jar
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# ion-list-sorting-and-canonicalization

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Canonicalizes a mass spectrum into the single-line 'm/z:intensity ion:intensity ...' text form required before SPLASH block computation: intensities are normalized to integers 0-100, m/z values are rendered to exactly 6 decimal places, zero-intensity ions are retained, N/A values are dropped, and ions are ordered by ascending m/z (ties broken by descending intensity).

## When to use

Use when a raw spectrum (e.g. a MassBank PR100026-style ion list such as '138.0641:71.59 195.0815:261.7') must be converted into the deterministic canonical ion:intensity string that feeds SPLASH hashing, similarity search, or cross-database comparison.

## When NOT to use

- The input is already a canonicalized spectrum file validated against the reference implementation — re-canonicalizing is unnecessary and may mask upstream defects
- The target is a structure or metabolite identifier (InChIKey-style); this skill only canonicalizes spectral ion lists, not chemical structures
- The spectrum contains annotations or metadata that must be preserved; canonicalization drops empty (N/A) entries and keeps only ion-intensity content

## Inputs

- Mass spectrum as a single-line string in the 'ion:intensity ion:intensity ...' format (e.g. '138.0641:71.59 195.0815:261.7' for the caffeine MassBank record PR100026)
- List of ion (m/z, intensity) pairs from a spectral library entry (MassBank, MoNA, GNPS, HMDB) or raw spectrum object

## Outputs

- Canonical spectrum text file containing the normalized, 6-decimal m/z, integer 0-100 intensity, ascending-m/z ion:intensity string
- Canonical spectrum object parseable by the SPLASH reference implementation (Spectrum/SpectrumImpl) for hashing

## How to apply

Read the input spectrum as ion (m/z, intensity) pairs from the single-line string representation used by the spectra-hash tooling. Normalize every intensity to the integer range 0-100, keeping zero-intensity ions but eliminating empty (N/A) values, because the hash and similarity blocks must be insensitive to instrument-specific intensity scaling. Render each m/z to exactly 6 decimal places and sort ions by ascending m/z, breaking ties by descending intensity, so that the same spectrum always yields one canonical ordering regardless of input order. Serialize the ions back into the 'm/z:intensity ion:intensity ...' format with single-space separators and write it to the canonical spectrum text file; this canonical file is the input consumed by SplashUtil.splash or SplashFactory for SPLASH generation and by the validation tool (-s column) for cross-implementation checking.

## Related tools

- **SplashUtil** (Provides the direct 'm/z:intensity ion:intensity ...' string interface that consumes the canonicalized spectrum and produces the SPLASH) — https://github.com/berlinguyinca/spectra-hash
- **SplashFactory** (Creates the default SPLASHer; used on a Spectrum object built from the canonicalized ion-intensity pairs) — https://github.com/berlinguyinca/spectra-hash
- **validation-1.8.jar** (Validation tool that checks a canonical spectrum column against the reference implementation to ensure consistent SPLASH generation) — https://github.com/berlinguyinca/spectra-hash
- **Java (spectra-hash reference implementation)** (Reference implementation (core-1.8.jar) containing the Spectrum, Ion, and canonicalization logic) — https://github.com/berlinguyinca/spectra-hash

## Examples

```
java -jar validation/target/validation-1.8.jar -c -s 2 -t ms ./base-dataset/spectra/notsplashed/test-set-v1.csv base-dataset/spectra/test-set-with-splash-v1.csv
```

## Evaluation signals

- The output file exists and every entry parses in the 'm/z:intensity' format with exactly 6-decimal m/z fields
- All intensities in the canonical output are integers within the 0-100 range and no N/A or empty values remain
- Ions appear in strictly ascending m/z order, with ties broken by descending intensity
- SPLASHing the canonical string via SplashUtil.splash yields a SPLASH identical to the reference implementation's output for the same input (e.g. the caffeine PR100026 spectrum hashes to splash10-0002-0900000000-b112e4e059e1ecf98c5f)
- Running the validation tool (java -jar validation/target/validation-1.8.jar -c -s 2 -t ms ...) reports consistency between the canonical file and the SPLASHed test set

## Limitations

- The canonicalization is defined for MS spectra (validation default SpectraType.MS); other spectrum types may need type-specific handling
- Identical SPLASHes arise for spectra containing a single ion of the same mass, so canonicalization plus hashing cannot distinguish such degenerate spectra
- Zero-intensity ions are intentionally retained, which can inflate block content for sparse spectra but is required for deterministic hashing

## Evidence

- [methods] Canonical representation normalizes intensities to integers 0-100, keeps zero-intensity ions, drops empty values, and sorts ions by ascending m/z then descending intensity: [evidence span withheld — anchor status: not_found]
