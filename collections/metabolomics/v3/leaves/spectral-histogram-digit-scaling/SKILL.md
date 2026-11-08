---
name: spectral-histogram-digit-scaling
description: Use when generating or verifying the third block of a SPLASH (SPectraL
  hASH) identifier from a mass spectrum given as ion:intensity pairs, e.g. to compare
  spectra across MassBank, MoNA, GNPS or HMDB entries.
when_to_use_negative:
- You need the second SPLASH summary block (chosen from 136 candidate block formats
  for search-space reduction), not the 100 Da binned histogram block
- You need the fourth block, which is the SHA256-derived unique hash, not a spectral
  histogram
- The spectrum is not an MS spectrum (the validation tool defaults to spectrum type
  MS)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: spectra-hash (SPLASH)
  role: Reference implementations (Java, Python, C++, C#, R) whose SplashFactory/SplashUtil
    compute SPLASH blocks including the binned third block; used to validate a hand-computed
    block
  repo: https://github.com/berlinguyinca/spectra-hash
- name: SPLASH validation tool
  role: Java validation-1.8.jar cross-checks computed SPLASHes against the reference
    implementation for consistent third-block generation across implementations
  repo: https://github.com/berlinguyinca/spectra-hash
- name: MassBank PR100026 (caffeine)
  role: Reference caffeine spectrum used as the ground-truth test case for reproducing
    third block '0900000000'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/nbt.3689
    title: SPLASH, a hashed identifier for mass spectra
  source_content_hash: sha256:3e949313d630412be75094230b22409243556994e3662b2e00dd8cff304a6003
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/berlinguyinca/spectra-hash
  tools:
  - MassBank PR100026 (caffeine)
  - Python
  - SPLASH validation tool
  - spectra-hash (SPLASH)
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# spectral-histogram-digit-scaling

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Computes the third summary block of a SPLASH identifier by summing ion intensities into ten 100 Da m/z bins and scaling each bin's summed intensity to a single base-10 digit (0–9). This yields a length-10 digit string such as caffeine's '0900000000' in splash10-0002-0900000000-b112e4e059e1ecf98c5f.

## When to use

Use when generating or verifying the third block of a SPLASH (SPectraL hASH) identifier from a mass spectrum given as ion:intensity pairs, e.g. to compare spectra across MassBank, MoNA, GNPS or HMDB entries.

## When NOT to use

- You need the second SPLASH summary block (chosen from 136 candidate block formats for search-space reduction), not the 100 Da binned histogram block
- You need the fourth block, which is the SHA256-derived unique hash, not a spectral histogram
- The spectrum is not an MS spectrum (the validation tool defaults to spectrum type MS)

## Inputs

- Mass spectrum as single-line string of ion:intensity pairs (e.g. '138.0641:71.59 195.0815:261.7' from MassBank PR100026, caffeine)
- List of (m/z, intensity) ion pairs parsed from MassBank, MoNA, GNPS or HMDB library entries

## Outputs

- Ten-character third SPLASH summary block (one base-10 digit 0–9 per 100 Da bin, concatenated in m/z order, e.g. '0900000000')

## How to apply

Parse the spectrum from its single-line 'ion:intensity ion:intensity ...' string representation (e.g. MassBank record PR100026 for caffeine). Bin the ion intensities by m/z into ten 100 Da windows covering the m/z range and sum the intensities within each window. Scale each bin's summed intensity to a single integral base-10 digit (0–9) and concatenate the ten digits in m/z order to form the third block. Validate the result against a published reference, e.g. the caffeine SPLASH 'splash10-0002-0900000000-b112e4e059e1ecf98c5f' whose third block must be '0900000000'; the Java reference implementation can be exercised via SplashFactory/SplashUtil or the bundled validation tool to cross-check outputs.

## Related tools

- **spectra-hash (SPLASH)** (Reference implementations (Java, Python, C++, C#, R) whose SplashFactory/SplashUtil compute SPLASH blocks including the binned third block; used to validate a hand-computed block) — https://github.com/berlinguyinca/spectra-hash
- **SPLASH validation tool** (Java validation-1.8.jar cross-checks computed SPLASHes against the reference implementation for consistent third-block generation across implementations) — https://github.com/berlinguyinca/spectra-hash
- **MassBank PR100026 (caffeine)** (Reference caffeine spectrum used as the ground-truth test case for reproducing third block '0900000000')

## Examples

```
String splash = SplashUtil.splash("138.0641:71.59 195.0815:261.7", SpectraType.MS);
```

## Evaluation signals

- The concatenated ten digits exactly match the third block of the published caffeine SPLASH 'splash10-0002-0900000000-b112e4e059e1ecf98c5f', i.e. '0900000000'
- Each of the ten characters is an integral digit in the range 0–9 (base-10 scaling)
- The block has exactly ten characters, one per 100 Da m/z bin, in m/z order
- Validation tool run (java -jar validation/target/validation-1.8.jar -c -s 2 -t ms ...) reports consistency with the reference implementation
- Identical spectra from different databases (PR100026 MassBank, 5464 HMDB, CCMSLIB00000222858 GNPS — all the same caffeine spectrum) yield the identical third block

## Limitations

- The third block is a coarse summary: identical blocks do not imply identical spectra; SPLASH uniqueness rests on the fourth SHA256-derived hash block
- Hash collision testing on 53.25 million spectra found identical SPLASHes only for single-ion spectra of the same mass, so very sparse spectra give weak discrimination
- Only ten 100 Da bins cover the m/z range, so fine spectral detail is lost in this block by design

## Evidence

- [methods] The third block of a SPLASH is computed by summing intensities over 100 Da bin sizes and scaling each bin to a single-digit integral base-10 digit (0–9), yielding a length-10 block such as caffeine's '0900000000'.: [evidence span withheld — anchor status: not_found]
- [readme] The README example caffeine SPLASH and its block structure.: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract statement of block structure including the hash block.: [evidence span withheld — anchor status: not_found]
