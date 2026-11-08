---
name: mass-spectra-reduced-spectrum-extraction
description: Use when a peak list (ion:intensity pairs) must be converted into the
  SPLASH second summary block, or when a 4-digit base-36 histogram block must be reproduced
  or verified against a known SPLASH identifier.
when_to_use_negative:
- Input is an unprocessed vendor raw file (mzML/raw) rather than a peak list in 'ion:intensity'
  string representation — peak picking/deconvolution must happen first
- You need the third or fourth SPLASH blocks (full histogram summary or SHA256-derived
  unique hash), not the reduced second block
- The spectrum should be compared structurally to other compounds — SPLASH is a hashed
  identifier, not a similarity score
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Python
  role: Reference implementation language used to parse the caffeine spectrum, build
    the reduced spectrum, and emit the second block for validation
  repo: https://github.com/berlinguyinca/spectra-hash
- name: spectra-hash (Java reference implementation)
  role: Authoritative SPLASHer (SplashFactory/SplashUtil) against which the Python-computed
    second block is validated
  repo: https://github.com/berlinguyinca/spectra-hash
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/nbt.3689
    title: SPLASH, a hashed identifier for mass spectra
  source_content_hash: sha256:2c2c99cfb70cdbf29f97fdd52ceae23774c1939241fcd7c768ff4c41b34f001a
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/berlinguyinca/spectra-hash
  tools:
  - Python
  - spectra-hash (Java reference implementation)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# mass-spectra-reduced-spectrum-extraction

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Computes the SPLASH second summary block by reducing a mass spectrum to its top-10 ions above 10% of the base peak, binning intensities into 5 Da m/z bins, scaling each bin to a base-3 digit (0–2), and encoding the resulting histogram as a 4-digit base-36 block.

## When to use

Use when a peak list (ion:intensity pairs) must be converted into the SPLASH second summary block, or when a 4-digit base-36 histogram block must be reproduced or verified against a known SPLASH identifier.

## When NOT to use

- Input is an unprocessed vendor raw file (mzML/raw) rather than a peak list in 'ion:intensity' string representation — peak picking/deconvolution must happen first
- You need the third or fourth SPLASH blocks (full histogram summary or SHA256-derived unique hash), not the reduced second block
- The spectrum should be compared structurally to other compounds — SPLASH is a hashed identifier, not a similarity score

## Inputs

- Mass spectrum as a single-line string of ion:intensity pairs (e.g. '138.0641:71.59 195.0815:261.7')
- Caffeine reference spectrum MassBank record PR100026 (identical to HMDB 5464 and GNPS CCMSLIB00000222858)

## Outputs

- 4-character base-36 second summary block string (e.g. '0002' for caffeine PR100026)
- Intermediate length-10 base-3 histogram of 5 Da bin intensities

## How to apply

Parse the spectrum from its single-line 'ion:intensity ion:intensity ...' representation (e.g. '138.0641:71.59 195.0815:261.7' for caffeine PR100026). Build the reduced spectrum by retaining at most the 10 most intense ions whose intensities exceed 10% of the base peak; this cutoff keeps only peaks contributing meaningful signal while bounding the block length. Sum the retained ion intensities within 5 Da bins along the m/z axis to form a binned histogram. Scale each bin's summed intensity relative to the base peak to a single base-3 digit (0, 1, or 2), then convert the resulting base-3 histogram into the 4-digit base-36 second summary block. The rationale is that this format, chosen from 136 candidate block formats, reduces the spectral search space by ≥94% while remaining short and web-search compatible. For caffeine, the block must be '0002', matching the published SPLASH 'splash10-0002-0900000000-b112e4e059e1ecf98c5f'.

## Related tools

- **Python** (Reference implementation language used to parse the caffeine spectrum, build the reduced spectrum, and emit the second block for validation) — https://github.com/berlinguyinca/spectra-hash
- **spectra-hash (Java reference implementation)** (Authoritative SPLASHer (SplashFactory/SplashUtil) against which the Python-computed second block is validated) — https://github.com/berlinguyinca/spectra-hash

## Examples

```
String splash = SplashUtil.splash("138.0641:71.59 195.0815:261.7", SpectraType.MS); // second block expected: 0002
```

## Evaluation signals

- The generated second block must equal '0002' for the caffeine PR100026 spectrum, matching the published SPLASH 'splash10-0002-0900000000-b112e4e059e1ecf98c5f'
- The reduced spectrum must contain at most 10 ions, all with intensities > 10% of the base peak
- Each histogram bin digit must be a valid base-3 value (0, 1, or 2), and the block must be exactly 4 base-36 characters
- The computed block should match the output of the Java reference implementation when fed the same 'ion:intensity ion:intensity' string via SplashUtil
- Writing the result to caffeine_second_block.txt and diffing against the expected value '0002'

## Limitations

- Identical SPLASHes only arose from mass spectra containing a single ion of the same mass; collisions are otherwise ~10^-31 for 10^9 spectra, but single-ion edge cases exist
- The reduced spectrum discards ions below 10% of base peak and beyond the top 10, so the second block is a lossy summary and cannot be inverted back to the full spectrum
- Consistency across implementations requires the validation tool; implementations must be checked against the reference (java -jar validation-1.8.jar with -c -s 2 -t ms flags)

## Evidence

- [other] The reduced spectrum is summed over 5 Da bins; each bin is scaled to a single-digit integral value in base 3 (0–2), and the resulting length-10 histogram is converted to a base 36 number, giving a 4-digit block.: [evidence span withheld — anchor status: not_found]
