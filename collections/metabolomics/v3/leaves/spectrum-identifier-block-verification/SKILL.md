---
name: spectrum-identifier-block-verification
description: Use when you have a known peak list (ion:intensity pairs) and a published
  SPLASH, and need to verify, reproduce, or cross-check its fourth hash block or the
  full identifier.
when_to_use_negative:
- Input is raw instrument data (e.g. mzML) rather than a peak list — the canonical
  string must be built from ion:intensity pairs first
- You need to verify the second or third summary blocks — this skill covers only the
  fourth (SHA256 hash) block
- Spectra consisting of a single ion of the same mass, where identical SPLASHes can
  legitimately arise regardless of other content
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Python (hashlib)
  role: Compute the SHA256 hexadecimal digest of the canonical spectrum string and
    truncate it to 20 characters
  repo: https://github.com/berlinguyinca/spectra-hash
- name: SplashUtil (Java reference implementation)
  role: Directly SPLASH a single-line spectrum string to obtain the reference identifier
    for comparison
  repo: https://github.com/berlinguyinca/spectra-hash
- name: validation-1.8.jar
  role: Validate a file of generated SPLASHes against the Java reference implementation
    in batch
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/nbt.3689
    title: SPLASH, a hashed identifier for mass spectra
  source_content_hash: sha256:03b2f8bba38d9461b254ff97c8eff2722638c9a7528bd69c35adbea642e24420
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/berlinguyinca/spectra-hash
  tools:
  - Python
  - Python (hashlib)
  - SplashUtil (Java reference implementation)
  - validation-1.8.jar
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# spectrum-identifier-block-verification

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Verifies the fourth (hash) block of a SPLASH identifier by recomputing the SHA256 digest (hexadecimal, truncated to 20 characters) of the canonical single-line spectrum string and comparing it to the published block, e.g. 'b112e4e059e1ecf98c5f' for caffeine PR100026.

## When to use

Use when you have a known peak list (ion:intensity pairs) and a published SPLASH, and need to verify, reproduce, or cross-check its fourth hash block or the full identifier.

## When NOT to use

- Input is raw instrument data (e.g. mzML) rather than a peak list — the canonical string must be built from ion:intensity pairs first
- You need to verify the second or third summary blocks — this skill covers only the fourth (SHA256 hash) block
- Spectra consisting of a single ion of the same mass, where identical SPLASHes can legitimately arise regardless of other content

## Inputs

- Peak list of m/z-intensity pairs for a mass spectrum (e.g. 138.0641 71.59 and 195.0815 261.7 from MassBank record PR100026)
- Published SPLASH identifier to compare against (e.g. splash10-0002-0900000000-b112e4e059e1ecf98c5f)
- Single-line spectrum string in 'ion:intensity ion:intensity ...' format

## Outputs

- 20-character hexadecimal SHA256-derived hash block (e.g. b112e4e059e1ecf98c5f)
- Verification result: recomputed hash block equals (or differs from) the published fourth block
- Optionally the full SPLASH identifier string with four dash-separated blocks

## How to apply

Construct the canonical single-line spectrum string in the format 'ion:intensity ion:intensity ...' with m/z values scaled by 10^6 as 64-bit integers and ion:intensity pairs space-delimited, as used by SplashUtil.splash in the reference implementation. Compute the SHA256 hash of that canonical string in hexadecimal notation (e.g. with Python's hashlib). Truncate the hexadecimal digest to the first 20 characters to obtain the fourth block. For the caffeine MassBank record PR100026 (138.0641:71.59 195.0815:261.7), the result must be 'b112e4e059e1ecf98c5f', giving the full SPLASH 'splash10-0002-0900000000-b112e4e059e1ecf98c5f'. Cross-validate against the Java reference implementation's validation tool (validation-1.8.jar) when processing batches of spectra.

## Related tools

- **Python (hashlib)** (Compute the SHA256 hexadecimal digest of the canonical spectrum string and truncate it to 20 characters) — https://github.com/berlinguyinca/spectra-hash
- **SplashUtil (Java reference implementation)** (Directly SPLASH a single-line spectrum string to obtain the reference identifier for comparison) — https://github.com/berlinguyinca/spectra-hash
- **validation-1.8.jar** (Validate a file of generated SPLASHes against the Java reference implementation in batch)

## Examples

```
import hashlib; block = hashlib.sha256("138.0641:71.59 195.0815:261.7".encode()).hexdigest()[:20]; assert block == 'b112e4e059e1ecf98c5f'
```

## Evaluation signals

- For caffeine PR100026 (138.0641:71.59 195.0815:261.7), the truncated hash block must exactly equal 'b112e4e059e1ecf98c5f'
- The recomputed block must be exactly 20 hexadecimal characters
- The full identifier must contain four dash-separated blocks with the first block equal to 'splash10'
- Batch outputs must match the reference implementation when run through 'java -jar validation/target/validation-1.8.jar -c -s 2 -t ms'
- Same canonical string input must deterministically reproduce the same hash block across Python and Java implementations

## Limitations

- Hash collision testing on 53.25 million spectra found identical SPLASHes only for single-ion spectra of the same mass; theoretical collision probability is approximately 10^-31 for a database of 10^9 spectra
- The hash is only reproducible if the canonical string follows the exact single-line 'ion:intensity' format with correctly scaled m/z integers; any deviation changes the block
- The block verifies content only for the specific peak list hashed; it is not a database-independent check of spectrum provenance beyond that

## Evidence

- [other] The fourth block of the SPLASH is a hash of the full spectrum computed with SHA256 in hexadecimal notation and truncated to 20 characters: [evidence span withheld — anchor status: not_found]
