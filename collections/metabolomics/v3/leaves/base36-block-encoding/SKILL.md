---
name: base36-block-encoding
description: Use when a single-line 'ion:intensity ion:intensity ...' mass spectrum
  (e.g. MassBank PR100026 caffeine) must be reduced to its SPLASH second summary block
  for search-space reduction or identifier validation.
when_to_use_negative:
- The goal is the fourth SPLASH hash block (SHA256-derived), which is a separate hashing
  step, not the base-36 binned summary
- The input is already a SPLASH identifier or a binned/feature table rather than a
  raw 'ion:intensity' spectrum string
- The spectrum contains a single ion — identical single-ion spectra of the same mass
  produce identical SPLASHes, so the block provides no discrimination
tools:
- name: Python
  role: Implementation language for parsing the spectrum string, 5 Da binning, base-3
    digit scaling, and base-36 conversion of the second block
  repo: https://github.com/berlinguyinca/spectra-hash
- name: spectra-hash (Java reference implementation)
  role: Reference SPLASHer (SplashUtil/SplashFactory) against which the Python-computed
    second block is validated
  repo: https://github.com/berlinguyinca/spectra-hash
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/nbt.3689
    title: SPLASH, a hashed identifier for mass spectra
  source_content_hash: sha256:bb6284044d8032476103f9ecc29adc77689a1c659f2def3231a0d726c5eba9d5
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/berlinguyinca/spectra-hash
  tools:
  - Python
  - spectra-hash (Java reference implementation)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# base36-block-encoding

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Computes the second summary block of a SPLASH identifier by reducing a mass spectrum to at most its 10 most intense ions above 10% of the base peak, summing intensities into 5 Da bins, scaling each bin to a base-3 digit (0-2), and converting the length-10 histogram into a 4-digit base-36 block.

## When to use

Use when a single-line 'ion:intensity ion:intensity ...' mass spectrum (e.g. MassBank PR100026 caffeine) must be reduced to its SPLASH second summary block for search-space reduction or identifier validation.

## When NOT to use

- The goal is the fourth SPLASH hash block (SHA256-derived), which is a separate hashing step, not the base-36 binned summary
- The input is already a SPLASH identifier or a binned/feature table rather than a raw 'ion:intensity' spectrum string
- The spectrum contains a single ion — identical single-ion spectra of the same mass produce identical SPLASHes, so the block provides no discrimination

## Inputs

- Mass spectrum in single-line string representation 'ion:intensity ion:intensity ...' (e.g. '138.0641:71.59 195.0815:261.7' for caffeine PR100026)
- Spectra type (default MS)

## Outputs

- 4-character base-36 second summary block (e.g. '0002' for caffeine PR100026)
- caffeine_second_block.txt validation artifact containing the generated block

## How to apply

Parse the spectrum string into ion:intensity pairs (e.g. '138.0641:71.59 195.0815:261.7' for caffeine PR100026). Build the reduced spectrum by keeping at most the 10 most intense ions whose intensities exceed 10% of the base peak. Sum intensities into 5 Da bins along the m/z axis to form a binned histogram. Scale each bin's summed intensity relative to the base peak to a single base-3 integral digit (0, 1, or 2), yielding a length-10 histogram. Convert that base-3 histogram to a 4-digit base-36 number, which becomes the second dash-separated block of the SPLASH. This block format was selected from 136 candidate formats as the best short, web-search-compatible way to cut the mass spectral search space.

## Related tools

- **Python** (Implementation language for parsing the spectrum string, 5 Da binning, base-3 digit scaling, and base-36 conversion of the second block) — https://github.com/berlinguyinca/spectra-hash
- **spectra-hash (Java reference implementation)** (Reference SPLASHer (SplashUtil/SplashFactory) against which the Python-computed second block is validated) — https://github.com/berlinguyinca/spectra-hash

## Examples

```
String splash = SplashUtil.splash("138.0641:71.59 195.0815:261.7", SpectraType.MS); // second block must be 0002
```

## Evaluation signals

- Generated second block equals '0002', matching the published caffeine SPLASH 'splash10-0002-0900000000-b112e4e059e1ecf98c5f' for PR100026
- Block is exactly 4 characters long and encodable in base 36
- Each histogram digit is a valid base-3 value (0, 1, or 2) derived from a 5 Da bin scaled to the base peak
- Reduced spectrum contains at most 10 ions, all above 10% of the base peak intensity
- Consistency check against the Java validation tool / reference implementation yields the same block

## Limitations

- The second block is a coarse summary: it reduced the search space by 94% or above but returned 87% of all spectra within a similarity score of 700, so it is not a unique spectrum fingerprint
- Identical SPLASHes can arise from mass spectra containing a single ion of the same mass; theoretical collision probability is approximately 10^-31 for a database of 10^9 spectra (fourth hash block)
- The block format was chosen from 136 candidate formats on a 563,902-spectrum development dataset; behavior on spectra outside that style (e.g. NIST, which cannot be publicly released) is not covered by the public validation set

## Evidence

- [abstract] Second block from 136 candidate formats: "The second block was chosen from 136 different potential block formats as the best short, web search-compatible way to reduce the mass spectral search space"
- [abstract] Second block search-space performance: "reduced the search space by 94% or above (36,107 spectra or less) in all cases, while returning 87% of all spectra within a similarity score of 700"
- [readme] SPLASH block structure and caffeine example: "the full SPLASH of a [caffeine mass spectrum](https://massbank.eu/MassBank/RecordDisplay?id=PR100026) above is `splash10-0002-0900000000-b112e4e059e1ecf98c5f`"
- [readme] Second and third blocks are summaries: "The first block is the SPLASH identifier, the second and third are summary blocks, and the fourth is the unique hash block."
- [readme] Single-line spectrum format: "The format for single-line string spectral representation follows:

```
ion:intensity ion:intensity ...
```"
- [abstract] Hash collision testing on 53.25 million spectra: "Testing the fourth block for hash collisions on the full dataset of 53.25 million spectra revealed that identical SPLASHes only arose from mass spectra containing a single ion of the same mass"
