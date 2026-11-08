---
name: mona-spectral-dataset-retrieval
description: Use when a study requires the openly deposited mass spectra used to develop
  and validate SPLASH identifiers, e.g. to regenerate SPLASHes or reproduce the 53.25-million-spectrum
  hash-collision test.
when_to_use_negative:
- You need the complete 563,902-spectrum validation set including NIST 14 entries
  — NIST spectra 'cannot be released publically' and are therefore not retrievable
  from MoNA
- You need MS/MS raw instrument files or chromatographic raw data rather than library
  peak-list spectra
- You need spectra from repositories not listed in the paper's dataset (e.g. mzCloud
  content), which is referenced separately and not part of the MoNA-deposited open
  subset
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3315
tools:
- name: MoNA
  role: Public spectral library portal from which the open subset of the 563,902-spectrum
    dataset (all non-NIST entries) is downloaded
- name: spectra-hash
  role: Reference SPLASH implementation (Java, with Python/C++/C#/R/Scala/JavaScript
    APIs) used downstream to generate SPLASHes from the retrieved spectra
  repo: https://github.com/berlinguyinca/spectra-hash
- name: Java
  role: Language of the reference implementation and validation tool applied to the
    retrieved spectra
- name: Python
  role: One of the reference implementation languages used to parse retrieved spectra
    and generate SPLASHes
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/nbt.3689
    title: SPLASH, a hashed identifier for mass spectra
  source_content_hash: sha256:ff12c09924f689cede9d70d4ce3b9ee75f065c32cbeaae20c2628a364fb82b80
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/berlinguyinca/spectra-hash
  tools:
  - Java
  - MoNA
  - Python
  - spectra-hash
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# mona-spectral-dataset-retrieval

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Retrieves the openly deposited subset of the 563,902-spectrum SPLASH validation dataset (MassBank, GNPS, HMDB, ReSpect, FiehnLib — all but NIST, which cannot be publicly released) from MoNA at http://mona.fiehnlab.ucdavis.edu, as the input corpus for mass-spectra identifier and hash-collision analyses.

## When to use

Use when a study requires the openly deposited mass spectra used to develop and validate SPLASH identifiers, e.g. to regenerate SPLASHes or reproduce the 53.25-million-spectrum hash-collision test.

## When NOT to use

- You need the complete 563,902-spectrum validation set including NIST 14 entries — NIST spectra 'cannot be released publically' and are therefore not retrievable from MoNA
- You need MS/MS raw instrument files or chromatographic raw data rather than library peak-list spectra
- You need spectra from repositories not listed in the paper's dataset (e.g. mzCloud content), which is referenced separately and not part of the MoNA-deposited open subset

## Inputs

- MoNA REST query for spectra from MassBank, GNPS, HMDB, ReSpect, and FiehnLib (all deposited libraries except NIST)
- Target dataset size expectation: the open subset of the 563,902-spectrum collection from MassBank, GNPS, HMDB, ReSpect, FiehnLib and NIST 14

## Outputs

- A parsed corpus of mass spectra as ion-intensity pairs, one Spectrum object per record (SpectraType.MS)
- A provenance table mapping each spectrum to its source library (MassBank/MoNA, GNPS, HMDB, ReSpect, FiehnLib) and original record identifier (e.g. PR100026)

## How to apply

Follow the paper's dataset description and download all spectra except the NIST library entries from MoNA (http://mona.fiehnlab.ucdavis.edu/), since 'all but the NIST spectra (which cannot be released publically) are available on MoNA'. The full dataset comprises 563,902 mass spectra from MassBank, GNPS, HMDB, ReSpect, FiehnLib and NIST 14; the MoNA-retrievable portion is the open subset used for the collision analysis reported on 53.25 million spectra. Each downloaded record should carry ion-intensity pairs plus its library origin (e.g. the caffeine spectrum appears as PR100026 in MassBank/MoNA, 5464 in HMDB, and CCMSLIB00000222858 in GNPS), which lets you cross-link identical spectra across repositories. Parse each record into a Spectrum object (SpectraType.MS) for downstream SPLASH generation via SplashFactory or SplashUtil. Record each spectrum's source library so downstream filtering (e.g. excluding NIST-derived entries) and provenance checks remain possible.

## Related tools

- **MoNA** (Public spectral library portal from which the open subset of the 563,902-spectrum dataset (all non-NIST entries) is downloaded)
- **spectra-hash** (Reference SPLASH implementation (Java, with Python/C++/C#/R/Scala/JavaScript APIs) used downstream to generate SPLASHes from the retrieved spectra) — https://github.com/berlinguyinca/spectra-hash
- **Java** (Language of the reference implementation and validation tool applied to the retrieved spectra)
- **Python** (One of the reference implementation languages used to parse retrieved spectra and generate SPLASHes)

## Examples

```
python fetch_mona_spectra.py --url http://mona.fiehnlab.ucdavis.edu --exclude-library NIST --out spectra.csv
```

## Evaluation signals

- The retrieved corpus excludes NIST 14 library entries; every record maps to one of MassBank, GNPS, HMDB, ReSpect, or FiehnLib
- Record count is consistent with the paper's 563,902-spectrum development dataset (open subset only)
- Cross-repository identity checks work: the caffeine spectrum is retrievable as PR100026 (MassBank/MoNA), 5464 (HMDB), and CCMSLIB00000222858 (GNPS) and parses to identical ion-intensity pairs
- Every parsed record yields a valid Spectrum object of type SpectraType.MS with ion:intensity pairs suitable for SplashUtil.splash string input

## Limitations

- NIST spectra (part of the 563,902-spectrum dataset from NIST 14) cannot be publicly released, so the MoNA-retrieved corpus is necessarily an incomplete replication of the paper's full validation set
- Not all spectra in the validation set are annotated with structures, limiting compound-level provenance analyses
- The collision analysis based on this corpus required scaling beyond the open subset (the paper reports collision testing on 53.25 million spectra), which the open subset alone cannot fully reproduce

## Evidence
