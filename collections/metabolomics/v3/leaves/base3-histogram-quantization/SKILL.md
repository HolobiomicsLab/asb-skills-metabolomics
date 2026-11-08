---
name: base3-histogram-quantization
description: Use when a mass spectrum given as a single-line 'ion:intensity ion:intensity
  ...' string (e.g. MassBank record PR100026) must be reduced to its 4-digit SPLASH
  second summary block via 5 Da binning, base-3 digit scaling, and base-36 conversion.
when_to_use_negative:
- When an unambiguous unique identifier is needed — the second block is a lossy search-space-reducing
  summary; use the fourth (SHA256-derived) hash block for identity
- When the input is not a mass spectrum in 'ion:intensity' single-line format (e.g.
  retention-time data or already-binned feature tables)
- When a SPLASH identifier already exists and only needs parsing or validation rather
  than regeneration
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: spectra-hash (Java reference implementation, SplashUtil/SplashFactory)
  role: Generates the full SPLASH including the second block via SplashUtil.splash
    on a spectrum string; validation jar cross-checks block output
  repo: https://github.com/berlinguyinca/spectra-hash
- name: Python
  role: Used to parse the 'ion:intensity' spectrum string, bin 5 Da, scale to base-3
    digits, and convert to the base-36 second block
  repo: https://github.com/berlinguyinca/spectra-hash
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/nbt.3689
    title: SPLASH, a hashed identifier for mass spectra
  source_content_hash: sha256:7b8b63d08592742fb77bd0ce019b93a6b9a6e2376877c6e20c31a303248c9e35
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/berlinguyinca/spectra-hash
  tools:
  - Python
  - spectra-hash (Java reference implementation, SplashUtil/SplashFactory)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# base3-histogram-quantization

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Computes the second block of a SPLASH (SPectraL hASH) identifier by reducing a mass spectrum to its top-10 ions (>10% base peak), binning intensities into 5 Da bins, scaling each bin to a base-3 digit, and encoding the length-10 histogram as a 4-character base-36 string.

## When to use

Use when a mass spectrum given as a single-line 'ion:intensity ion:intensity ...' string (e.g. MassBank record PR100026) must be reduced to its 4-digit SPLASH second summary block via 5 Da binning, base-3 digit scaling, and base-36 conversion.

## When NOT to use

- When an unambiguous unique identifier is needed — the second block is a lossy search-space-reducing summary; use the fourth (SHA256-derived) hash block for identity
- When the input is not a mass spectrum in 'ion:intensity' single-line format (e.g. retention-time data or already-binned feature tables)
- When a SPLASH identifier already exists and only needs parsing or validation rather than regeneration

## Inputs

- Mass spectrum as a single-line 'ion:intensity ion:intensity ...' string (e.g. '138.0641:71.59 195.0815:261.7')
- Spectra type (default MS)

## Outputs

- 4-character base-36 second block string (e.g. '0002' for caffeine PR100026)
- Intermediate length-10 histogram of base-3 digits (values 0, 1, 2)

## How to apply

Parse the spectrum from its 'ion:intensity' single-line representation (e.g. '138.0641:71.59 195.0815:261.7' for caffeine PR100026). Build the reduced spectrum by keeping at most the 10 most intense ions whose intensities exceed 10% of the base peak. Sum ion intensities within 5 Da bins along the m/z axis to form a binned histogram, then scale each bin's summed intensity relative to the base peak to a single base-3 digit (0, 1, or 2). Convert the resulting length-10 base-3 histogram into one 4-digit base-36 number — the second block. This block format was selected from 136 candidate formats as the best short, web-search-compatible way to reduce the mass spectral search space; the hash block, not this coarse block, carries uniqueness (collision probability ~10^-31 for 10^9 spectra).

## Related tools

- **spectra-hash (Java reference implementation, SplashUtil/SplashFactory)** (Generates the full SPLASH including the second block via SplashUtil.splash on a spectrum string; validation jar cross-checks block output) — https://github.com/berlinguyinca/spectra-hash
- **Python** (Used to parse the 'ion:intensity' spectrum string, bin 5 Da, scale to base-3 digits, and convert to the base-36 second block) — https://github.com/berlinguyinca/spectra-hash

## Examples

```
String splash = SplashUtil.splash("138.0641:71.59 195.0815:261.7", SpectraType.MS);
```

## Evaluation signals

- For caffeine PR100026 the generated second block must equal '0002', matching the published SPLASH 'splash10-0002-0900000000-b112e4e059e1ecf98c5f'
- The block is exactly 4 characters, each a valid base-36 digit
- Every histogram bin digit is an integer in {0, 1, 2} and the histogram has length 10 (one digit per 5 Da bin)
- No more than 10 ions are retained, and each retained ion's intensity exceeds 10% of the base peak
- Output is consistent with the Java reference implementation when validated with the validation tool (java -jar validation-1.8.jar with -k/-o/-s/-t flags)

## Limitations

- The second block is deliberately coarse: it reduced the search space by 94% or above but returned only 87% of all spectra within a similarity score of 700, so it is a filter, not an identity key
- Identical SPLASHes arose only from mass spectra containing a single ion of the same mass — single-ion spectra can collide on the identifier
- The block format was chosen empirically from 136 candidate formats on a 563,902-spectrum dataset; it is not guaranteed optimal for arbitrary spectra

## Evidence

- [abstract] We designed the SPLASH (SPectraL hASH) as an unambiguous, database-independent spectrum identifier: "We designed the SPLASH (SPectraL hASH) as an unambiguous, database-independent spectrum identifier"
- [readme] The first block is the SPLASH identifier, the second and third are summary blocks, and the fourth is the unique hash block.: "The first block is the SPLASH identifier, the second and third are summary blocks, and the fourth is the unique hash block."
- [abstract] The second block was chosen from 136 different potential block formats as the best short, web search-compatible way to reduce the mass spectral search space: "The second block was chosen from 136 different potential block formats as the best short, web search-compatible way to reduce the mass spectral search space"
- [readme] build a `Spectrum` object from a list of ion-intnsity pairs and use the `SplashFactory` to utilize the default SPLASHer: "build a `Spectrum` object from a list of ion-intnsity pairs and use the `SplashFactory` to utilize the default SPLASHer"
