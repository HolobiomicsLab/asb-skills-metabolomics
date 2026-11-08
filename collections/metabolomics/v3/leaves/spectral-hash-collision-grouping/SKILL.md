---
name: spectral-hash-collision-grouping
description: Use when you need to test the uniqueness of SPLASH identifiers across
  a large deposited spectral library (e.g. the 563,902-spectrum MoNA subset of MassBank,
  GNPS, HMDB, ReSpect, FiehnLib and NIST 14) by identifying and characterizing groups
  of spectra sharing the same fourth-block hash.
when_to_use_negative:
- Spectra are already grouped by SPLASH and you only need to look up, not recompute,
  collision groups
- The analysis goal is spectral similarity ranking or search-space reduction using
  the second summary block, not fourth-block uniqueness testing
- Input spectra contain structures/annotations rather than raw ion-intensity pairs,
  since not all spectra in the validation set are annotated with structures
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Java
  role: Reference SPLASH implementation used to generate SPLASH identifiers via SplashFactory.splashIt
    or SplashUtil.splash, providing the fourth-block SHA256 hash for grouping
  repo: https://github.com/berlinguyinca/spectra-hash
- name: Python
  role: Used for parsing spectra, extracting fourth blocks, grouping collisions, and
    writing splash_collision_groups.csv
- name: MoNA
  role: Repository hosting the openly deposited spectra subset (all but NIST) that
    serves as the input dataset
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/nbt.3689
    title: SPLASH, a hashed identifier for mass spectra
  source_content_hash: sha256:2adf496905b47463b0c437dae1538d6d192bff044dc2eef7c862cdb9a587bc25
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/berlinguyinca/spectra-hash
  tools:
  - Java
  - MoNA
  - Python
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# spectral-hash-collision-grouping

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Groups mass spectra by the SHA256-derived fourth block of their SPLASH identifier to detect hash collisions, verifying that identical SPLASHes only arise from single-ion spectra of the same mass, as established on 53.25 million spectra in the SPLASH paper.

## When to use

Use when you need to test the uniqueness of SPLASH identifiers across a large deposited spectral library (e.g. the 563,902-spectrum MoNA subset of MassBank, GNPS, HMDB, ReSpect, FiehnLib and NIST 14) by identifying and characterizing groups of spectra sharing the same fourth-block hash.

## When NOT to use

- Spectra are already grouped by SPLASH and you only need to look up, not recompute, collision groups
- The analysis goal is spectral similarity ranking or search-space reduction using the second summary block, not fourth-block uniqueness testing
- Input spectra contain structures/annotations rather than raw ion-intensity pairs, since not all spectra in the validation set are annotated with structures

## Inputs

- Mass spectra parsed into ion-intensity pairs from the openly deposited MoNA subset (MassBank, GNPS, HMDB, ReSpect, FiehnLib; excluding NIST)
- Spectrum objects constructed with SpectraType.MS
- SPLASH identifier strings whose fourth block is the SHA256-derived hash

## Outputs

- splash_collision_groups.csv with group sizes, member spectrum IDs, and single-ion status per collision group
- Overall hash-collision rate for the analyzed dataset

## How to apply

Download the openly deposited spectra subset (all but the NIST spectra, which cannot be released publicly) from MoNA at http://mona.fiehnlab.ucdavis.edu/. Parse each of the 563,902 mass spectra into ion-intensity pairs and build a Spectrum object per record (SpectraType.MS), then generate a SPLASH for every spectrum using the reference Java implementation via SplashFactory.splashIt or the SplashUtil.splash string shortcut. Extract the fourth block (the SHA256-derived hash) from each SPLASH and group spectra whose fourth blocks are identical. For each collision group, check whether members are mass spectra containing a single ion of the same mass — the only expected source of identical SPLASHes, since single-ion spectra are identical by definition after intensity normalization. Record group sizes, member spectrum IDs, and single-ion status (e.g. in splash_collision_groups.csv) and report the overall collision rate.

## Related tools

- **Java** (Reference SPLASH implementation used to generate SPLASH identifiers via SplashFactory.splashIt or SplashUtil.splash, providing the fourth-block SHA256 hash for grouping) — https://github.com/berlinguyinca/spectra-hash
- **Python** (Used for parsing spectra, extracting fourth blocks, grouping collisions, and writing splash_collision_groups.csv)
- **MoNA** (Repository hosting the openly deposited spectra subset (all but NIST) that serves as the input dataset)

## Examples

```
String splash = SplashUtil.splash("138.0641:71.59 195.0815:261.7", SpectraType.MS); String block4 = splash.split("-")[3];
```

## Evaluation signals

- Identical SPLASHes arise only from mass spectra containing a single ion of the same mass, matching the paper's finding on 53.25 million spectra
- No non-trivial collisions are found beyond single-ion same-mass spectra, consistent with a theoretical collision probability of approximately 10^-31 for a database of 10^9 spectra
- Each SPLASH's fourth block is a valid SHA256-derived hash string of length equal to the reference implementation's output (e.g. b112e4e059e1ecf98c5f for caffeine PR100026)
- splash_collision_groups.csv contains nonempty group-size, member-ID, and single-ion-status columns for every collision group
- The overall collision rate is reported and the total spectra count matches the analyzed dataset (e.g. 563,902)

## Limitations

- Identical SPLASHes can legitimately arise from mass spectra containing a single ion of the same mass — these are not true hash collisions
- NIST spectra cannot be released publicly and are therefore excluded from the openly deposited MoNA subset
- Not all spectra in the validation set are annotated with structures, limiting downstream structural comparisons

## Evidence

- [abstract] The theoretical collision probability is approximately 10^-31 for a database of 10^9 spectra: [evidence span withheld — anchor status: not_found]
