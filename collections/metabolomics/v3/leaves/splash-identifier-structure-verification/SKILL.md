---
name: splash-identifier-structure-verification
description: Use when a mass spectrum in single-line 'ion:intensity' representation
  (e.g. MassBank PR100026 caffeine) has a published SPLASH and you need to verify
  the identifier's blocks are reproducible from the spectrum.
when_to_use_negative:
- Input is a processed feature table or peak list in another format rather than the
  single-line 'ion:intensity' spectrum string
- The goal is spectral similarity search or library matching rather than verifying
  identifier structure — use the SPLASH second block only as a search-space reducer,
  not as a similarity score
- The spectrum is not a mass spectrum of type MS (the validation tool defaults to
  '-t ms')
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Python
  role: Parse the caffeine PR100026 spectrum string, bin the reduced spectrum into
    5 Da bins, scale to base-3 digits, and emit the base-36 second block
  repo: https://github.com/berlinguyinca/spectra-hash
- name: Java SPLASH reference implementation
  role: Generate the full SPLASH via SplashFactory or SplashUtil.splash('138.0641:71.59
    195.0815:261.7', SpectraType.MS) to cross-validate the hand-computed blocks
  repo: https://github.com/berlinguyinca/spectra-hash
- name: Validation tool (validation-1.8.jar)
  role: Batch-verify generated SPLASHes against the reference implementation using
    flags k (SPLASH column), s (spectrum column), t (spectra type)
  repo: https://github.com/berlinguyinca/spectra-hash
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/nbt.3689
    title: SPLASH, a hashed identifier for mass spectra
  source_content_hash: sha256:d0cad8c3b09327cd3aeb0f9e5e35feb3daa02d4812192f5ab0202a2535e94f9b
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/berlinguyinca/spectra-hash
  tools:
  - Java SPLASH reference implementation
  - Python
  - Validation tool (validation-1.8.jar)
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# splash-identifier-structure-verification

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Recomputes the summary blocks of a SPLASH (SPectraL hASH) identifier from a raw mass spectrum and checks them against the published identifier, verifying that block generation (top-10 ion filtering, 5 Da binning, base-3 scaling, base-36 encoding, SHA256 hash block) is reproduced correctly.

## When to use

Use when a mass spectrum in single-line 'ion:intensity' representation (e.g. MassBank PR100026 caffeine) has a published SPLASH and you need to verify the identifier's blocks are reproducible from the spectrum.

## When NOT to use

- Input is a processed feature table or peak list in another format rather than the single-line 'ion:intensity' spectrum string
- The goal is spectral similarity search or library matching rather than verifying identifier structure — use the SPLASH second block only as a search-space reducer, not as a similarity score
- The spectrum is not a mass spectrum of type MS (the validation tool defaults to '-t ms')

## Inputs

- Mass spectrum in single-line string representation 'ion:intensity ion:intensity ...' (e.g. MassBank record PR100026)
- Published SPLASH identifier to verify against (e.g. splash10-0002-0900000000-b112e4e059e1ecf98c5f)
- Spectra type flag (default: MS)

## Outputs

- Computed 4-digit base-36 second summary block (e.g. '0002' for caffeine)
- Full regenerated SPLASH identifier (four dash-separated blocks: identifier, two summary blocks, SHA256-derived hash block)
- Verification artifact (e.g. caffeine_second_block.txt)

## How to apply

Parse the spectrum from its single-line 'ion:intensity ion:intensity ...' format (for caffeine PR100026, e.g. '138.0641:71.59 195.0815:261.7'). Build the reduced spectrum by keeping at most the 10 most intense ions whose intensities exceed 10% of the base peak. Sum intensities into 5 Da bins along the m/z axis, scale each bin relative to the base peak to a base-3 digit (0–2), and convert the length-10 histogram into the 4-digit base-36 second summary block. Validate by regenerating the full SPLASH with the Java reference implementation (SplashFactory/SplashUtil) or the Python implementation and comparing block-by-block against the published identifier — for caffeine this must yield 'splash10-0002-0900000000-b112e4e059e1ecf98c5f', whose second block '0002' should be written to caffeine_second_block.txt.

## Related tools

- **Python** (Parse the caffeine PR100026 spectrum string, bin the reduced spectrum into 5 Da bins, scale to base-3 digits, and emit the base-36 second block) — https://github.com/berlinguyinca/spectra-hash
- **Java SPLASH reference implementation** (Generate the full SPLASH via SplashFactory or SplashUtil.splash('138.0641:71.59 195.0815:261.7', SpectraType.MS) to cross-validate the hand-computed blocks) — https://github.com/berlinguyinca/spectra-hash
- **Validation tool (validation-1.8.jar)** (Batch-verify generated SPLASHes against the reference implementation using flags k (SPLASH column), s (spectrum column), t (spectra type)) — https://github.com/berlinguyinca/spectra-hash

## Examples

```
String splash = SplashUtil.splash("138.0641:71.59 195.0815:261.7", SpectraType.MS);
```

## Evaluation signals

- The computed second block equals '0002', matching the second block of the published caffeine SPLASH 'splash10-0002-0900000000-b112e4e059e1ecf98c5f'
- The full regenerated SPLASH matches the published identifier block-for-block, including the SHA256-derived fourth hash block
- The reduced spectrum contains no more than 10 ions, each above 10% of the base peak intensity
- The bin histogram has exactly length 10 with digits restricted to 0, 1, 2 (base-3)
- Java validation tool run (java -jar validation/target/validation-1.8.jar -c -s 2 -t ms) reports no mismatches against the reference implementation

## Limitations

- The second block compresses the spectrum so heavily that it is only a search-space reducer (94% reduction while returning 87% of spectra within similarity score 700); it is not a unique fingerprint
- Identical SPLASHes can arise for spectra containing a single ion of the same mass; theoretical collision probability is ~10^-31 for a database of 10^9 spectra
- The identical caffeine spectrum is represented differently across MassBank (PR100026), HMDB (5464), and GNPS (CCMSLIB00000222858), so raw spectrum strings from different libraries may not match byte-for-byte before parsing

## Evidence

- [methods] The reduced spectrum is summed over 5 Da bins; each bin is scaled to a single-digit integral value in base 3 (0–2): [evidence span withheld — anchor status: not_found]
