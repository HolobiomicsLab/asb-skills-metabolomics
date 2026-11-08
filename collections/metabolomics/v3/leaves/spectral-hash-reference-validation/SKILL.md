---
name: spectral-hash-reference-validation
description: Use when a mass spectrum must be assigned a database-independent identifier,
  cross-referenced across MassBank/MoNA/GNPS/HMDB, or checked for consistent SPLASH
  generation across tools and implementations.
when_to_use_negative:
- The input is not a mass spectrum but a chemical structure — use an InChIKey-based
  approach instead, since SPLASH is explicitly a spectral counterpart to the InChIKey
  and not a general chemical identifier.
- The spectrum representation cannot be normalized to the 'ion:intensity ion:intensity
  …' single-line format — the canonicalization and SHA-256 hash block depend on this
  exact representation, and other encodings will yield a different identifier.
- You need to reverse the SPLASH back into the original peak list — the fourth block
  is a SHA-256 hash and does not encode the peaks recoverably.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0157
tools:
- name: spectra-hash (Java reference implementation)
  role: Reference implementation that computes the SPLASH from a spectrum; use as
    ground truth to validate other implementations
  repo: https://github.com/berlinguyinca/spectra-hash
- name: validation tool
  role: Java JAR used to validate SPLASH generation consistency between tools and
    implementations on CSV datasets
  repo: http://gose.fiehnlab.ucdavis.edu:55000/content/groups/public/edu/ucdavis/fiehnlab/splash/validation/1.8/validation-1.8.jar
- name: REST service
  role: Web service for SPLASH generation via HTTP endpoints at splash.fiehnlab.ucdavis.edu
  repo: http://splash.fiehnlab.ucdavis.edu
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/nbt.3689
    title: SPLASH, a hashed identifier for mass spectra
  source_content_hash: sha256:923fba17dbecd124e70c81c1f893f1831535ae7d2d84d2c652f6452d9dd04eac
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/berlinguyinca/spectra-hash
  tools:
  - Java
  - Python
  - REST service
  - spectra-hash (Java reference implementation)
  - validation tool
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# spectral-hash-reference-validation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

A method for generating and validating SPLASH identifiers (SPectraL hASH) for mass spectra, by building a four-block identifier (SPLASH10 prefix, two summary blocks, and a SHA-256 hash block) and verifying it against the reference Java implementation so that identical spectra from MassBank, MoNA, GNPS, HMDB, MetaboLights, and mzCloud converge on the same database-independent identifier.

## When to use

Use when a mass spectrum must be assigned a database-independent identifier, cross-referenced across MassBank/MoNA/GNPS/HMDB, or checked for consistent SPLASH generation across tools and implementations.

## When NOT to use

- The input is not a mass spectrum but a chemical structure — use an InChIKey-based approach instead, since SPLASH is explicitly a spectral counterpart to the InChIKey and not a general chemical identifier.
- The spectrum representation cannot be normalized to the 'ion:intensity ion:intensity …' single-line format — the canonicalization and SHA-256 hash block depend on this exact representation, and other encodings will yield a different identifier.
- You need to reverse the SPLASH back into the original peak list — the fourth block is a SHA-256 hash and does not encode the peaks recoverably.

## Inputs

- Mass spectrum in single-line 'ion:intensity ion:intensity …' string representation (e.g. '138.0641:71.59 195.0815:261.7' for MassBank record PR100026, caffeine)
- Spectra type (e.g. MS) to select the appropriate SPLASHer
- CSV test dataset with columns for spectrum text, spectrum origin/ID, and generated SPLASH for the validation tool

## Outputs

- Four-block SPLASH identifier string, e.g. splash10-0002-0900000000-b112e4e059e1ecf98c5f
- Validation report from the Java validation tool comparing SPLASHes across implementations
- REST query result from the splash.fiehnlab.ucdavis.edu service (optional)

## How to apply

Represent the spectrum as a single-line 'ion:intensity' string (e.g. '138.0641:71.59 195.0815:261.7' for the caffeine spectrum MassBank PR100026). Construct the four-block SPLASH: the SPLASH10 identifier block, two spectral summary blocks separated by dashes, and a fourth hash block derived via SHA-256 from the canonicalized peak representation. Generate the SPLASH with the reference Java implementation by building a `Spectrum` object from a list of Ion pairs and passing it to `SplashFactory.create().splashIt(spectrum)`, or use `SplashUtil.splash(...)` for a string representation. For cross-implementation validation, run the Java validation tool from the project root with flags that map the columns of the input CSV: k = column of the generated SPLASH, o = column of the spectrum ID/origin, s = column of the single-line spectrum, and t = type of spectrum (default MS). Compare the computed SPLASH to a published reference, e.g. splash10-0002-0900000000-b112e4e059e1ecf98c5f for the caffeine spectrum, requiring a byte-for-byte match.

## Related tools

- **spectra-hash (Java reference implementation)** (Reference implementation that computes the SPLASH from a spectrum; use as ground truth to validate other implementations) — https://github.com/berlinguyinca/spectra-hash
- **validation tool** (Java JAR used to validate SPLASH generation consistency between tools and implementations on CSV datasets) — http://gose.fiehnlab.ucdavis.edu:55000/content/groups/public/edu/ucdavis/fiehnlab/splash/validation/1.8/validation-1.8.jar
- **REST service** (Web service for SPLASH generation via HTTP endpoints at splash.fiehnlab.ucdavis.edu) — http://splash.fiehnlab.ucdavis.edu

## Examples

```
java -jar validation/target/validation-1.8.jar -c -s 2 -t ms ./base-dataset/spectra/notsplashed/test-set-v1.csv base-dataset/spectra/test-set-with-splash-v1.csv
```

## Evaluation signals

- Byte-for-byte match of the computed SPLASH with the published identifier for a known spectrum (e.g. splash10-0002-0900000000-b112e4e059e1ecf98c5f for MassBank PR100026 caffeine).
- Consistent SPLASH across independent implementations: the Java validation tool reports no discrepancies when run on the test-set CSV with the correct column flags (-k, -o, -s, -t ms).
- Identical SPLASH for equivalent records across databases: PR100026 (MassBank/MoNA), 5464 (HMDB), and CCMSLIB00000222858 (GNPS) must all produce the same identifier.
- Stable SHA-256 hash block: re-running on the same spectrum yields an unchanged fourth block; changing any peak (m/z or intensity) changes the hash.
- Absence of trivial hash collisions: distinct spectra with more than one ion of different mass must not share a fourth block (identical SPLASHes only arose from single-ion spectra of the same mass in the 53.25-million-spectrum test).

## Limitations

- Identical SPLASHes can arise for spectra containing a single ion of the same mass — these are not distinguishable by the hash block.
- Theoretical collision probability for a database of 10^9 spectra is approximately 10^-31, but collisions are not impossible, only vanishingly rare.
- Not all spectra in the validation set are annotated with structures, so SPLASH cannot be used to infer structural identity for every entry.
- The exact canonicalization of the peak list (ordering, formatting, precision) directly determines the SHA-256 hash block; deviations in representation change the identifier.

## Evidence
