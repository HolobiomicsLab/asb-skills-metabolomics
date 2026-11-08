---
name: neme-mass-error-statistics
description: Use when validating molecular formula annotations from IDSL.UFA (or any
  MS1 isotopic-profile matching tool) by quantifying monoisotopic mass error in mDa
  against authentic reference standards.
when_to_use_negative:
- The input requires MS2/MSn-based structural validation — NEME and PCS operate on
  MS1 isotopic profiles only, since the IDSL.UFA pipeline only requires MS1 for molecular
  formula annotation
- No authentic reference standards or deposited theoretical/experimental isotopic
  profile spectra are available to compare against
- The data are already an aligned annotated peak table of formula ranks and frequencies
  (e.g. median rank/detection frequency summaries) rather than isotopic profile spectra
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: IDSL.UFA
  role: Assigns molecular formulas to LC/HRMS peaks via theoretical vs. experimental
    isotopic profile matching; its assigned monoisotopic masses are the subject of
    the NEME/PCS validation
  repo: https://cran.r-project.org/package=IDSL.UFA
- name: R
  role: Implementation language for computing per-compound NEME (mDa) and PCS and
    aggregating means across the 367 standards
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.2c00563
    title: IDSL.UFA Assigns High-Confidence Molecular Formula Annotations for Untargeted
      LC/HRMS Data Sets in Metabolomics and Exposomics
  source_content_hash: sha256:b3050b29a7e4697fa88139422bf843292d214ff80bc06b3f77ed3740923dda36
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.UFA
  - R
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# neme-mass-error-statistics

## Summary

Computes the normalized Euclidean mass error (NEME, in mDa) of monoisotopic masses for IDSL.UFA-assigned molecular formulas against authentic reference standards, alongside profile cosine similarity (PCS), to quantify how accurately MS1 isotopic-profile matching assigns formulas.

## When to use

Use when validating molecular formula annotations from IDSL.UFA (or any MS1 isotopic-profile matching tool) by quantifying monoisotopic mass error in mDa against authentic reference standards.

## When NOT to use

- The input requires MS2/MSn-based structural validation — NEME and PCS operate on MS1 isotopic profiles only, since the IDSL.UFA pipeline only requires MS1 for molecular formula annotation
- No authentic reference standards or deposited theoretical/experimental isotopic profile spectra are available to compare against
- The data are already an aligned annotated peak table of formula ranks and frequencies (e.g. median rank/detection frequency summaries) rather than isotopic profile spectra

## Inputs

- MSV000088661 authentic reference standards LC/HRMS dataset
- Deposited IDSL.UFA output from Zenodo record 5803968 (theoretical and experimental integrated isotopic profile spectra across chromatography)
- Per-compound IDSL.UFA annotation results with assigned molecular formulas

## Outputs

- Per-compound table of NEME values (monoisotopic mass error in mDa)
- Per-compound pairwise cosine similarity (PCS) values between experimental and theoretical isotopic profiles
- Aggregate mean NEME and mean PCS across all compounds

## How to apply

Download the MSV000088661 authentic reference standards LC/HRMS dataset and the deposited IDSL.UFA output from Zenodo record 5803968, which contains the theoretical and experimental integrated isotopic profile spectra across chromatography for each standard. Load the per-compound IDSL.UFA annotation results in R and extract the theoretical and experimental integrated isotopic profile spectra for each of the 367 reference standards. For each compound's assigned formula, compute the NEME (mean error of the monoisotopic mass in mDa) and the pairwise cosine similarity (PCS) between the theoretical and experimental isotopic profiles. Assemble a per-compound table of NEME and PCS values, then aggregate means across all compounds. Correct application is confirmed when the aggregate mean NEME is close to the reported ~0.70 mDa and the aggregate mean PCS is close to ~99.968%.

## Related tools

- **IDSL.UFA** (Assigns molecular formulas to LC/HRMS peaks via theoretical vs. experimental isotopic profile matching; its assigned monoisotopic masses are the subject of the NEME/PCS validation) — https://cran.r-project.org/package=IDSL.UFA
- **R** (Implementation language for computing per-compound NEME (mDa) and PCS and aggregating means across the 367 standards)

## Examples

```
library(IDSL.UFA); UFA_workflow("Address of the UFA parameter spreadsheet")
```

## Evaluation signals

- Aggregate mean NEME across the 367 authentic standard compounds is close to the reported ~0.70 mDa
- Aggregate mean PCS across compounds is close to the reported ~99.968%
- Per-compound table contains one NEME (mDa) and one PCS value per reference standard, computed from matched theoretical and experimental isotopic profiles
- PCS values fall in the 0–100% range and are computed as pairwise cosine similarities between experimental and theoretical isotopic profiles
- Theoretical isotopic profiles used for comparison derive from the deposited Zenodo record 5803968, not re-predicted from scratch

## Limitations

- Validation is bounded to 367 authentic reference standard compounds from MSV000088661; accuracy on non-standard, database-dependent, or exposure chemicals is not established by these metrics
- Version of IDSL.UFA (and IDSL.IPA) is not stated in the context, complicating exact reproduction
- NEME captures monoisotopic mass error only; full annotation confidence additionally relies on PCS and tandem mass spectrometry support

## Evidence

- [results] Aggregate NEME/PCS benchmark: [evidence span withheld — anchor status: not_found]
- [readme] Theoretical isotope prediction basis: [evidence span withheld — anchor status: external_doc]
