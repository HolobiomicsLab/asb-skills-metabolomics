---
name: splash-block-format-verification
description: Use when a mass spectrum in single-line 'ion:intensity' format has an
  associated SPLASH identifier and you must confirm a specific block (e.g., the 100
  Da-binned third block) matches what the reference implementation would emit.
when_to_use_negative:
- The goal is to generate a new SPLASH identifier rather than verify an existing one
  against a reference block
- The spectrum is not representable in the single-line 'ion:intensity' format and
  cannot be converted to ion-intensity pairs
- The task is chemical structure identification or spectral similarity scoring — SPLASH
  blocks are summaries, not similarity measures
tools:
- name: spectra-hash (SPLASH reference implementations)
  role: Reference Java implementation (SplashFactory/SplashUtil) and Python/C++/C#/R
    implementations used to recompute and verify SPLASH blocks
  repo: https://github.com/berlinguyinca/spectra-hash
- name: validation-1.8.jar (SPLASH validation tool)
  role: Validates a file of spectra against the reference implementation, checking
    generated SPLASHes column-by-column
  repo: https://github.com/berlinguyinca/spectra-hash
- name: Python
  role: One of the reference implementation languages used to parse the spectrum and
    recompute the 100 Da-binned third block
  repo: https://github.com/berlinguyinca/spectra-hash
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/nbt.3689
    title: SPLASH, a hashed identifier for mass spectra
  source_content_hash: sha256:403d4e394df3e8505e2d41014260f21f6a9ef2bf86acb057be79172184ff7018
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/berlinguyinca/spectra-hash
  tools:
  - Python
  - spectra-hash (SPLASH reference implementations)
  - validation-1.8.jar (SPLASH validation tool)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# splash-block-format-verification

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Verifies that individual blocks of a SPLASH identifier (e.g., the third 100 Da-binned summary block such as '0900000000' for caffeine) are correctly computed from a mass spectrum, using the reference implementations and validation tool from the spectra-hash project.

## When to use

Use when a mass spectrum in single-line 'ion:intensity' format has an associated SPLASH identifier and you must confirm a specific block (e.g., the 100 Da-binned third block) matches what the reference implementation would emit.

## When NOT to use

- The goal is to generate a new SPLASH identifier rather than verify an existing one against a reference block
- The spectrum is not representable in the single-line 'ion:intensity' format and cannot be converted to ion-intensity pairs
- The task is chemical structure identification or spectral similarity scoring — SPLASH blocks are summaries, not similarity measures

## Inputs

- Mass spectrum in single-line string representation 'ion:intensity ion:intensity ...' (e.g., '138.0641:71.59 195.0815:261.7' for caffeine, MassBank PR100026)
- Published SPLASH identifier to verify against (e.g., 'splash10-0002-0900000000-b112e4e059e1ecf98c5f')
- For bulk validation: a CSV of unsplashed spectra plus a CSV of reference SPLASHes (e.g., test-set-v1.csv and test-set-with-splash-v1.csv)

## Outputs

- Recomputed SPLASH block string (e.g., the 10-digit third block '0900000000' for caffeine)
- Pass/fail comparison of computed vs. reference block(s)
- Validation-tool report of consistent vs. inconsistent SPLASHes across implementations

## How to apply

Parse the spectrum from its single-line 'ion:intensity ion:intensity ...' representation (e.g., MassBank record PR100026 for caffeine). Recompute the block of interest: for the third block, bin ion intensities by m/z into ten 100 Da windows, scale each bin's summed intensity to a single base-10 digit (0-9), and concatenate the ten digits in m/z order. Cross-check the recomputed block against the published identifier — for caffeine the target is the '0900000000' block inside 'splash10-0002-0900000000-b112e4e059e1ecf98c5f'. For bulk verification, use the project's Java validation tool to compare generated SPLASHes column-by-column against a reference-implementation output file, using flags such as -c, -s (spectrum column), and -t ms. The rationale is that the SPLASH is designed as an unambiguous, database-independent identifier, so any deviation from the reference block signals a broken or non-conformant implementation.

## Related tools

- **spectra-hash (SPLASH reference implementations)** (Reference Java implementation (SplashFactory/SplashUtil) and Python/C++/C#/R implementations used to recompute and verify SPLASH blocks) — https://github.com/berlinguyinca/spectra-hash
- **validation-1.8.jar (SPLASH validation tool)** (Validates a file of spectra against the reference implementation, checking generated SPLASHes column-by-column) — https://github.com/berlinguyinca/spectra-hash
- **Python** (One of the reference implementation languages used to parse the spectrum and recompute the 100 Da-binned third block) — https://github.com/berlinguyinca/spectra-hash

## Examples

```
java -jar validation/target/validation-1.8.jar -c -s 2 -t ms ./base-dataset/spectra/notsplashed/test-set-v1.csv base-dataset/spectra/test-set-with-splash-v1.csv
```

## Evaluation signals

- For caffeine (MassBank PR100026), the recomputed third block must exactly equal '0900000000' and the full identifier must match 'splash10-0002-0900000000-b112e4e059e1ecf98c5f'
- Each summary block digit must be a single base-10 digit (0-9); the third block must contain exactly ten digits in m/z order over 100 Da bins
- The validation tool run with flags -c -s 2 -t ms must report agreement between generated and reference SPLASH columns for the test set
- Identical spectra recorded in different libraries (MassBank PR100026, HMDB 5464, GNPS CCMSLIB00000222858) must yield identical SPLASH blocks

## Limitations

- Hash-collision testing on 53.25 million spectra found identical SPLASHes arising only from mass spectra containing a single ion of the same mass, so block agreement alone does not guarantee unique spectrum identity
- The validation tool's flags (k = SPLASH column, o = spectrum ID column, s = spectrum column, t = spectrum type, T = delimiter, X = debug) must be set correctly for the input CSV layout or verification will fail spuriously
- Verification depends on the spectrum being the same experimental spectrum as the reference record; the caffeine example relies on PR100026 (MassBank/MoNA), 5464 (HMDB), and CCMSLIB00000222858 (GNPS) representing exactly the same spectrum

## Evidence
