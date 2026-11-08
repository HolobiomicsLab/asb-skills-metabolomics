---
name: hash-block-truncation
description: Use when generating or validating a SPLASH identifier for a mass spectrum
  and the fourth (hash) block must be computed from the canonical ion:intensity representation,
  or when checking an existing SPLASH's hash block.
when_to_use_negative:
- You only need the second or third SPLASH summary blocks (e.g. the search-space-reducing
  summary block), not the unique hash block
- The identifier must encode structural/chemical information rather than being derived
  from the spectrum itself — SPLASH is a spectrum identifier analogous to InChIKey
  for structures, not a compound-structure key
- The spectrum text has not been canonicalized (m/z scaled by 10^6 as 64-bit integers,
  pairs sorted and space-delimited) — hashing a non-canonical string will not match
  the reference SPLASH
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Python
  role: Reference language for computing the SHA256 hash of the canonical spectrum
    string with hashlib and truncating the hex digest to 20 characters
  repo: https://github.com/berlinguyinca/spectra-hash
- name: spectra-hash
  role: Reference SPLASH implementations (Java, Python, C++, C#, R, Ruby, Scala, JavaScript)
    that generate the hash block and a validation tool to check consistency across
    implementations
  repo: https://github.com/berlinguyinca/spectra-hash
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/nbt.3689
    title: SPLASH, a hashed identifier for mass spectra
  source_content_hash: sha256:bc99488f3cadbe71507e81ff6db2f16f1ba2dfb564f0ae93cd1e7193afbb7941
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/berlinguyinca/spectra-hash
  tools:
  - Python
  - spectra-hash
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# hash-block-truncation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Computes the fourth block of a SPLASH identifier by hashing the canonical single-line spectrum string with SHA256 and truncating the hexadecimal digest to 20 characters, as specified for mass spectra such as the caffeine example.

## When to use

Use when generating or validating a SPLASH identifier for a mass spectrum and the fourth (hash) block must be computed from the canonical ion:intensity representation, or when checking an existing SPLASH's hash block.

## When NOT to use

- You only need the second or third SPLASH summary blocks (e.g. the search-space-reducing summary block), not the unique hash block
- The identifier must encode structural/chemical information rather than being derived from the spectrum itself — SPLASH is a spectrum identifier analogous to InChIKey for structures, not a compound-structure key
- The spectrum text has not been canonicalized (m/z scaled by 10^6 as 64-bit integers, pairs sorted and space-delimited) — hashing a non-canonical string will not match the reference SPLASH

## Inputs

- Mass spectrum peak list as ion:intensity pairs (e.g. m/z and intensity pairs from a MassBank record such as caffeine PR100026: 138.0641:71.59 195.0815:261.7)
- Single-line spectrum string in the format 'ion:intensity ion:intensity ...'

## Outputs

- 20-character hexadecimal SHA256 hash block (e.g. 'b112e4e059e1ecf98c5f') suitable as the fourth block of a full SPLASH identifier

## How to apply

First construct the canonical single-line spectrum string: sort the m/z-intensity pairs, scale m/z values to 64-bit integers by multiplying by 10^6, and join each m/z with its normalized intensity by a colon, with pairs space-delimited (e.g. '138.0641:71.59 195.0815:261.7' for caffeine PR100026). Compute the SHA256 hash of this canonical string using Python's hashlib (or the reference SPLASH implementations in Java, Python, C++, C#, R, Ruby, Scala, or JavaScript from berlinguyinca/spectra-hash). Take the hexadecimal SHA256 digest and truncate it to the first 20 characters; for the caffeine spectrum this yields 'b112e4e059e1ecf98c5f', completing the full SPLASH 'splash10-0002-0900000000-b112e4e059e1ecf98c5f'. The hash block is deterministic and database-independent, so the same spectrum must always produce the identical 20-character block across libraries and implementations.

## Related tools

- **Python** (Reference language for computing the SHA256 hash of the canonical spectrum string with hashlib and truncating the hex digest to 20 characters) — https://github.com/berlinguyinca/spectra-hash
- **spectra-hash** (Reference SPLASH implementations (Java, Python, C++, C#, R, Ruby, Scala, JavaScript) that generate the hash block and a validation tool to check consistency across implementations) — https://github.com/berlinguyinca/spectra-hash

## Examples

```
import hashlib; h = hashlib.sha256('138.0641:71.59 195.0815:261.7'.encode()).hexdigest(); print(h[:20])  # b112e4e059e1ecf98c5f
```

## Evaluation signals

- The truncated 20-character hash block exactly equals 'b112e4e059e1ecf98c5f' for the caffeine spectrum PR100026
- The assembled SPLASH matches the full reference string 'splash10-0002-0900000000-b112e4e059e1ecf98c5f'
- Running the same spectrum through the project's validation tool (java -jar validation-1.8.jar) produces a hash block consistent with the reference implementation
- Identical spectra from different libraries (MassBank PR100026, HMDB 5464, GNPS CCMSLIB00000222858) yield the same hash block
- The digest is exactly 20 hexadecimal characters (10 bytes of the SHA256 hex output)

## Limitations

- Identical SPLASHes can arise for mass spectra containing a single ion of the same mass; collision probability is approximately 10^-31 for a database of 10^9 spectra, so single-ion spectra are the known edge case
- The hash block is only reproducible if the canonical spectrum string is constructed exactly as specified (sorted pairs, m/z scaled by 10^6 as 64-bit integers, colon-delimited ion:intensity pairs joined by spaces); any deviation breaks matching

## Evidence

- [abstract] The fourth block of the SPLASH is a hash of the full spectrum computed with SHA256 in hexadecimal notation and truncated to 20 characters: [evidence span withheld — anchor status: not_found]
