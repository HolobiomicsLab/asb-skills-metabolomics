---
name: spectral-intensity-binning-at-5-da
description: Use when you need to reduce a mass spectrum (ion:intensity pairs) to
  a coarse, search-compatible summary — e.g. to build SPLASH's second block or pre-filter
  spectra before cosine-similarity ranking.
when_to_use_negative:
- You need an exact, high-resolution spectral representation — binning to 5 Da and
  base-3 digits intentionally discards detail
- Input is not a mass spectrum (e.g. MS/MS precursor list, chromatogram) or lacks
  intensity information
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: SPLASH reference implementation
  role: Generates the SPLASH identifier; the SplashListener lets you inspect individual
    blocks (including the 5 Da-binned histogram) before they are hashed
  repo: https://github.com/berlinguyinca/spectra-hash
- name: Python API
  role: One of the language-specific implementations that applies the 5 Da binning
    and base-3/base-36 encoding to derive the second block
  repo: https://github.com/berlinguyinca/spectra-hash
- name: Java API
  role: 'Reference implementation: SplashUtil.splash(...) applies the full SPLASHing
    algorithm including the 5 Da-binned second block'
  repo: https://github.com/berlinguyinca/spectra-hash
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/nbt.3689
    title: SPLASH, a hashed identifier for mass spectra
  source_content_hash: sha256:b5fe1b0eee4aeca6bb345190805a00013b6de4fd633467c01e609a63c4afdf00
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/berlinguyinca/spectra-hash
  tools:
  - Java API
  - Python
  - Python API
  - SPLASH reference implementation
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# spectral-intensity-binning-at-5-da

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

A dimensionality-reduction step that bins a mass spectrum's ion intensities into 5 Da bins along the m/z axis and scales each bin to a base-3 digit, forming a compact 10-digit histogram used to derive the SPLASH identifier's second summary block.

## When to use

Use when you need to reduce a mass spectrum (ion:intensity pairs) to a coarse, search-compatible summary — e.g. to build SPLASH's second block or pre-filter spectra before cosine-similarity ranking.

## When NOT to use

- You need an exact, high-resolution spectral representation — binning to 5 Da and base-3 digits intentionally discards detail
- Input is not a mass spectrum (e.g. MS/MS precursor list, chromatogram) or lacks intensity information

## Inputs

- Mass spectrum in single-line string representation 'ion:intensity ion:intensity ...' (e.g. '138.0641:71.59 195.0815:261.7' for caffeine PR100026)

## Outputs

- Reduced spectrum (top-10 ions >10% base peak intensity)
- Binned intensity histogram over 5 Da bins
- Length-10 base-3 digit histogram (digits 0–2)
- 4-digit base-36 second summary block (e.g. '0002')

## How to apply

Parse the spectrum as 'ion:intensity' pairs (e.g. caffeine PR100026: '138.0641:71.59 195.0815:261.7'). First build the reduced spectrum by keeping at most the 10 most intense ions whose intensities exceed 10% of the base peak. Then partition the m/z axis into 5 Da bins and sum ion intensities within each bin to form a binned histogram. Scale each bin's summed intensity relative to the base peak to a single-digit base-3 value (0, 1, or 2), yielding a length-10 histogram. This base-3 histogram is then converted to a base-36 number to produce the 4-digit second summary block of the SPLASH (for the caffeine example this yields '0002'). The method was chosen from 136 candidate block formats as the best short, web search-compatible way to reduce the mass spectral search space.

## Related tools

- **SPLASH reference implementation** (Generates the SPLASH identifier; the SplashListener lets you inspect individual blocks (including the 5 Da-binned histogram) before they are hashed) — https://github.com/berlinguyinca/spectra-hash
- **Python API** (One of the language-specific implementations that applies the 5 Da binning and base-3/base-36 encoding to derive the second block) — https://github.com/berlinguyinca/spectra-hash
- **Java API** (Reference implementation: SplashUtil.splash(...) applies the full SPLASHing algorithm including the 5 Da-binned second block) — https://github.com/berlinguyinca/spectra-hash

## Examples

```
String splash = SplashUtil.splash("138.0641:71.59 195.0815:261.7", SpectraType.MS);
```

## Evaluation signals

- For the caffeine PR100026 example, the resulting second block must equal '0002', matching the published SPLASH splash10-0002-0900000000-b112e4e059e1ecf98c5f
- The binned histogram must contain exactly 10 base-3 digits (values 0, 1, or 2)
- Only ions with intensity >10% of the base peak, capped at the 10 most intense, should contribute to the reduced spectrum
- The 4-digit second block should be a valid base-36 encoding of the base-3 histogram
- When run across a large spectral database (e.g. 53.25 million spectra), the second block should reduce the search space by 94% or more while returning 87% of spectra within a similarity score of 700

## Limitations

- The 5 Da bin and base-3 digitization lose spectral detail by design — identical blocks can arise from different spectra
- Only spectra with ion intensities above 10% of the base peak (max 10 ions) contribute, so low-intensity features are ignored
- The second block alone is not a unique identifier; it is a coarse pre-filter and must be combined with the other SPLASH blocks (including the SHA256 hash block)
- Hash-collision testing on the full SPLASH showed identical SPLASHes only arose from mass spectra containing a single ion of the same mass — implying the binning step alone is not collision-free

## Evidence

- [other] The reduced spectrum is summed over 5 Da bins; each bin is scaled to a single-digit integral value in base 3 (0–2), and the resulting length-10 histogram is converted to a base 36 number, giving a 4-digit block.: [evidence span withheld — anchor status: not_found]
- [readme] For example, the full SPLASH of a caffeine mass spectrum above is splash10-0002-0900000000-b112e4e059e1ecf98c5f.: [evidence span withheld — anchor status: not_found]
