---
name: sha256-hash-computation
description: Use when a mass spectrum in 'ion:intensity' single-line form needs a
  database-independent identifier, or when validating/reproducing the fourth block
  of an existing SPLASH.
when_to_use_negative:
- The goal is to compute the first three SPLASH blocks (identifier or spectral summary
  blocks) — those use different algorithms, not the SHA256 hash
- The input is a raw instrument file (mzML, profile data) rather than a peak list;
  peaks must first be extracted into the ion:intensity representation
- A different identifier scheme (e.g., InChIKey for chemical structures) is required
  — SPLASH identifiers are specific to mass spectra
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: hashlib-based SHA256 implementation used to hash the canonical spectrum string
    and truncate to the 20-character fourth block
  repo: https://github.com/berlinguyinca/spectra-hash
- name: spectra-hash
  role: reference implementations (Java via SplashFactory/SplashUtil, plus C++, Python,
    C#, R, Ruby, Scala wrappers) that generate the full SPLASH including the SHA256
    hash block
  repo: https://github.com/berlinguyinca/spectra-hash
- name: Java validation tool
  role: validation-1.8.jar cross-checks SPLASH generation against the reference implementation
    for consistent hash-block output
  repo: https://github.com/berlinguyinca/spectra-hash
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/nbt.3689
    title: SPLASH, a hashed identifier for mass spectra
  source_content_hash: sha256:6a27597935d778522bd975b31694e08318125579a2b38b87f7941b0a8ce9c94c
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/berlinguyinca/spectra-hash
  tools:
  - Java validation tool
  - Python
  - spectra-hash
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# sha256-hash-computation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Computes the fourth (hash) block of a SPLASH identifier by hashing the canonical single-line spectrum string with SHA256 in hexadecimal notation and truncating the digest to 20 characters.

## When to use

Use when a mass spectrum in 'ion:intensity' single-line form needs a database-independent identifier, or when validating/reproducing the fourth block of an existing SPLASH.

## When NOT to use

- The goal is to compute the first three SPLASH blocks (identifier or spectral summary blocks) — those use different algorithms, not the SHA256 hash
- The input is a raw instrument file (mzML, profile data) rather than a peak list; peaks must first be extracted into the ion:intensity representation
- A different identifier scheme (e.g., InChIKey for chemical structures) is required — SPLASH identifiers are specific to mass spectra

## Inputs

- Single-line mass spectrum string in 'ion:intensity ion:intensity ...' format (e.g., '138.0641:71.59 195.0815:261.7' for caffeine MassBank record PR100026)

## Outputs

- 20-character hexadecimal SHA256 hash block (e.g., 'b112e4e059e1ecf98c5f') suitable as the fourth block of a full SPLASH identifier

## How to apply

Build the canonical single-line spectrum string in the form 'ion:intensity ion:intensity ...' where m/z values are scaled by 10^6 as 64-bit integers and joined with normalized intensities by colons, with pairs space-delimited (e.g., the caffeine PR100026 spectrum 138.0641:71.59 195.0815:261.7). Compute the SHA256 hash of this canonical string using Python's hashlib and obtain the hexadecimal digest. Truncate the hexadecimal digest to its first 20 characters; this truncated string is the fourth SPLASH block. Verify against the published caffeine example: full SPLASH 'splash10-0002-0900000000-b112e4e059e1ecf98c5f' requires the truncated hash to equal 'b112e4e059e1ecf98c5f'. The same hashing is implemented in the reference Java implementation (SplashFactory/SplashUtil) and in the C++, Python, C#, R, Ruby and Scala reference implementations in the spectra-hash repository.

## Related tools

- **Python** (hashlib-based SHA256 implementation used to hash the canonical spectrum string and truncate to the 20-character fourth block) — https://github.com/berlinguyinca/spectra-hash
- **spectra-hash** (reference implementations (Java via SplashFactory/SplashUtil, plus C++, Python, C#, R, Ruby, Scala wrappers) that generate the full SPLASH including the SHA256 hash block) — https://github.com/berlinguyinca/spectra-hash
- **Java validation tool** (validation-1.8.jar cross-checks SPLASH generation against the reference implementation for consistent hash-block output) — https://github.com/berlinguyinca/spectra-hash

## Examples

```
import hashlib; block = hashlib.sha256('138.0641:71.59 195.0815:261.7'.encode()).hexdigest()[:20]
```

## Evaluation signals

- The truncated 20-character SHA256 block for the caffeine PR100026 spectrum must exactly equal 'b112e4e059e1ecf98c5f'
- The assembled SPLASH must equal the published 'splash10-0002-0900000000-b112e4e059e1ecf98c5f'
- The full (untruncated) SHA256 digest must be 64 hexadecimal characters before truncation
- Running the same spectrum through the validation tool against the reference implementation must produce matching SPLASHes (java -jar validation-1.8.jar with -t ms)

## Limitations

- Collision testing on 53.25 million spectra found identical SPLASHes only for single-ion spectra of the same mass, so single-peak spectra of equal m/z cannot be distinguished by the hash block
- Theoretical collision probability is approximately 10^-31 for a database of 10^9 spectra, but hash identity does not guarantee spectral identity for degenerate single-ion cases
- The hash depends on the exact canonical string (10^6-scaled integer m/z, normalized intensities); deviations in scaling or ordering yield a different block

## Evidence
