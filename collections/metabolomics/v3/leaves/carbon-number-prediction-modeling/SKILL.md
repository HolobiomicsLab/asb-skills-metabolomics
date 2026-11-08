---
name: carbon-number-prediction-modeling
description: Use when IDSL.IPA peaklists contain R13C peak properties and you need
  to predict carbon counts for elemental composition, or validate that experimental
  R13C agrees with theoretical isotope ratios from formulas.
when_to_use_negative:
- When peaks have not passed IDSL.IPA isotopologue pairing and peak quality thresholds
  — low-abundance or noisy peaks lacking reliable 12C/13C ion pairs yield unusable
  R13C values
- When the candidate formula's carbon count exceeds Cn = 92, beyond the validated
  range of the R13C̄ ≈ 1.0816 * Cn calibration
- When the goal is adduct/fragment identification or molecular formula annotation
  from aligned matrices rather than carbon-number estimation from R13C ratios
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: IDSL.IPA
  role: Generates the peaklists and R13C/nIsoPair chromatographic peak properties
    used to compute experimental mean R13C for each standard
  repo: https://cran.r-project.org/package=IDSL.IPA
- name: enviPat
  role: Calculates theoretical isotopic profiles for the standards' molecular formulas
    to derive theoretical R13C ground truth
- name: R
  role: Host language in which the correlation, calibration, and prediction models
    are implemented
  repo: https://cran.r-project.org/package=IDSL.IPA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00120
    title: IDSL.IPA Characterizes the Organic Chemical Space in Untargeted LC/HRMS
      Data Sets
  source_content_hash: sha256:01d5b574f0d5836e6f626203502b4cb20c2589dcd3085dec153035ead0de6287
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.IPA
  - R
  - enviPat
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# carbon-number-prediction-modeling

## Summary

Validate and calibrate a carbon-number estimator that predicts how many carbons a molecular formula can contain from the mean experimental R13C (cumulated 13C / 12C intensity ratio x 100) measured on IDSL.IPA peaklists, using enviPat-derived theoretical R13C as ground truth across 367 authentic standards.

## When to use

Use when IDSL.IPA peaklists contain R13C peak properties and you need to predict carbon counts for elemental composition, or validate that experimental R13C agrees with theoretical isotope ratios from formulas.

## When NOT to use

- When peaks have not passed IDSL.IPA isotopologue pairing and peak quality thresholds — low-abundance or noisy peaks lacking reliable 12C/13C ion pairs yield unusable R13C values
- When the candidate formula's carbon count exceeds Cn = 92, beyond the validated range of the R13C̄ ≈ 1.0816 * Cn calibration
- When the goal is adduct/fragment identification or molecular formula annotation from aligned matrices rather than carbon-number estimation from R13C ratios

## Inputs

- IDSL.IPA peaklists (peak areas, nIsoPair, R13C peak properties) from untargeted LC/HRMS data
- 367 authentic reference compound LC/HRMS dataset (Fusion Orbitrap, Zenodo 10.5281/zenodo.4743898)
- Molecular formulas of the analyzed standards
- Theoretical isotopic profiles for molecular formulas (enviPat)

## Outputs

- Per-compound table of experimental R13C, theoretical R13C, molecular formula, and carbon number (e.g. r13c_standards_results.csv)
- Spearman correlation statistics (p-value, rho) between theoretical and experimental R13C
- Fitted linear calibration equation R13C̄ ≈ 1.0816 * Cn linking mean R13C to carbon number
- Predicted carbon numbers for candidate elemental compositions

## How to apply

Process the 367 reference compound dataset (Zenodo 10.5281/zenodo.4743898, Fusion Orbitrap LC/HRMS) with IDSL.IPA to obtain peaklists carrying peak areas, nIsoPair, and R13C chromatographic peak properties; for each standard compute the experimental mean R13C as (cumulated 13C isotope-pair intensity / 12C monoisotopic intensity) x 100. Use the enviPat R package to calculate theoretical isotopic profiles for the standards' molecular formulas and derive corresponding theoretical R13C values. Fit the linear relationship R13C̄ ≈ 1.0816 * Cn (valid for Cn ≤ 92), where Cn is the number of possible carbons, so that carbon number can be predicted directly from a measured R13C. Judge success by the Spearman rank correlation between theoretical and experimental R13C across all 367 standards, which must reproduce p-value ≤ 2.2*10⁻¹⁶ and ρ ≤ 0.92, and by a fitted slope matching the reported 1.0816 coefficient.

## Related tools

- **IDSL.IPA** (Generates the peaklists and R13C/nIsoPair chromatographic peak properties used to compute experimental mean R13C for each standard) — https://cran.r-project.org/package=IDSL.IPA
- **enviPat** (Calculates theoretical isotopic profiles for the standards' molecular formulas to derive theoretical R13C ground truth)
- **R** (Host language in which the correlation, calibration, and prediction models are implemented) — https://cran.r-project.org/package=IDSL.IPA

## Examples

```
library(enviPat); r13c <- read.csv("peaklists/R13C.csv"); fit <- lm(R13Cbar ~ Cn, data = r13c)
```

## Evaluation signals

- Spearman correlation between theoretical and experimental R13C for 367 authentic standards reproduces p-value ≤ 2.2*10⁻¹⁶ and ρ ≤ 0.92
- Fitted slope of the R13C̄ vs carbon-number regression matches the reported 1.0816 coefficient
- Predicted carbon numbers remain valid only within Cn ≤ 92
- Experimental mean R13C is consistently computed as (cumulated 13C isotope-pair intensity / 12C monoisotopic intensity) x 100 for every compound

## Limitations

- Calibration is validated only for carbon numbers up to 92; extrapolating beyond Cn = 92 is unsupported
- Experimental R13C depends on IDSL.IPA peak quality — a majority of rejected EICs had low abundance but passed the S/N ratio threshold, so additional auto-filtering rules are still needed
- Agreement was demonstrated on authentic standards of common metabolites; performance on exposomics chemical space (e.g. FDA substance registry or EPA CompTox compounds) is not validated here

## Evidence

- [results] Spearman correlation p-value and rho: "the theoretical and experimental R13C¯ showed a good agreement (Spearman correlation test, p-value ≤ 2.2*10−16, ρ ≤ 0.92) for 367 authentic standards"
- [results] Linear calibration equation: "enable prediction of number of carbons for elemental composition (R13C¯≈1.0816∗Cn, where Cn indicates number of possible carbons, Cn ≤ 92)"
- [results] enviPat role: "The enviPat R package40 was used for calculating theoretical isotopic profiles for molecular formulas"
- [results] R13C as IDSL.IPA peak property: [evidence span withheld — anchor status: not_found]
- [supplementary] Dataset provenance: "367 reference compounds analyzed by a Fusion Orbitrap instrument (https://doi.org/10.5281/zenodo.4743898)"
