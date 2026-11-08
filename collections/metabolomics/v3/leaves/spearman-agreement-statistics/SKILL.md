---
name: spearman-agreement-statistics
description: Use when validating that an experimentally derived isotopic ratio metric
  (e.g. R13C from LC/HRMS peaklists) agrees with theory computed from molecular formulas
  across a benchmark set of authentic standards.
when_to_use_negative:
- The input is raw MS1 spectra rather than an IDSL.IPA-derived peaklist with R13C
  and nIsoPair peak properties — compute those first.
- The validation set contains fewer than the 367 reference standards, limiting statistical
  power for the reported ρ ≤ 0.92 threshold.
- You are annotating formulas on unannotated peaks; use IDSL.UFA or IDSL.UFAx for
  isotopic profile matching instead, and reserve this skill for post-hoc agreement
  testing.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: IDSL.IPA
  role: Generates peaklists with nIsoPair and R13C chromatographic peak properties
    from Fusion Orbitrap raw data of authentic standards
  repo: https://github.com/idslme/IDSL.IPA
- name: enviPat
  role: Calculates theoretical isotopic profiles for the standards' molecular formulas
    to derive theoretical R13C values
- name: R
  role: Environment in which IDSL.IPA, enviPat, and the Spearman correlation test
    are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00120
    title: IDSL.IPA Characterizes the Organic Chemical Space in Untargeted LC/HRMS
      Data Sets
  source_content_hash: sha256:ef311af44964c4039811ba8b49fa663589d237318990aec18b784a0b892e707a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.IPA
  tools:
  - IDSL.IPA
  - R
  - enviPat
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spearman-agreement-statistics

## Summary

Computes and validates Spearman rank correlation between theoretical and experimentally measured R13C values (13C/12C integrated intensity ratios) across authentic standards, using enviPat-computed theoretical isotopic profiles against IDSL.IPA-derived peak properties on 367 authentic metabolite standards.

## When to use

Use when validating that an experimentally derived isotopic ratio metric (e.g. R13C from LC/HRMS peaklists) agrees with theory computed from molecular formulas across a benchmark set of authentic standards.

## When NOT to use

- The input is raw MS1 spectra rather than an IDSL.IPA-derived peaklist with R13C and nIsoPair peak properties — compute those first.
- The validation set contains fewer than the 367 reference standards, limiting statistical power for the reported ρ ≤ 0.92 threshold.
- You are annotating formulas on unannotated peaks; use IDSL.UFA or IDSL.UFAx for isotopic profile matching instead, and reserve this skill for post-hoc agreement testing.

## Inputs

- IDSL.IPA peaklists for 367 authentic standards (Fusion Orbitrap, Zenodo 10.5281/zenodo.4743898) with peak areas, nIsoPair and R13C chromatographic peak properties
- Molecular formulas for each authentic standard (from RefMet or FDA substance registry)
- Theoretical isotopic profiles computed via enviPat from the standards' molecular formulas

## Outputs

- Per-compound table of experimental R13C, theoretical R13C, molecular formula, and carbon number (r13c_standards_results.csv)
- Spearman correlation statistics (ρ and p-value) between theoretical and experimental R13C
- Fitted linear slope of R13C̄ vs. carbon number (expected ≈ 1.0816 for Cn ≤ 92)

## How to apply

Compute experimental R13C for each standard as (cumulated 13C isotope-pair intensity / 12C monoisotopic intensity) × 100 from IDSL.IPA peaklists (nIsoPair and R13C peak properties). Use enviPat to generate theoretical isotopic profiles from the standards' molecular formulas and derive theoretical R13C values. Then compute Spearman's rank correlation (ρ) between the two sets across all 367 authentic standards and check significance. A successful reproduction matches the paper's reported p-value ≤ 2.2e−16 and ρ ≤ 0.92. Finally, fit the linear relation R13C̄ ≈ 1.0816 × Cn (Cn ≤ 92) linking mean R13C to carbon number to confirm the theoretical coefficient.

## Related tools

- **IDSL.IPA** (Generates peaklists with nIsoPair and R13C chromatographic peak properties from Fusion Orbitrap raw data of authentic standards) — https://github.com/idslme/IDSL.IPA
- **enviPat** (Calculates theoretical isotopic profiles for the standards' molecular formulas to derive theoretical R13C values)
- **R** (Environment in which IDSL.IPA, enviPat, and the Spearman correlation test are executed)

## Examples

```
library(IDSL.IPA); IPA_workflow('IPA_parameters.xlsx') # generate peaklists with nIsoPair & R13C; then cor.test(theoretical_R13C, experimental_R13C, method='spearman')
```

## Evaluation signals

- Spearman correlation between theoretical and experimental R13C across 367 standards must yield p-value ≤ 2.2e−16 and ρ ≤ 0.92
- The fitted linear slope of mean R13C vs. carbon number (Cn) must match the reported 1.0816 coefficient (valid for Cn ≤ 92)
- Every standard in the output table must carry a non-empty molecular formula and carbon number used to derive its theoretical R13C
- Output CSV (r13c_standards_results.csv) must contain per-compound experimental R13C, theoretical R13C, molecular formula, carbon number, and correlation statistics
- All experimental R13C values must be computed as (cumulated 13C isotope-pair intensity / 12C monoisotopic intensity) × 100 from IDSL.IPA peak properties

## Limitations

- Agreement is only demonstrated on 367 authentic standards of common metabolites analyzed by a Fusion Orbitrap instrument; extrapolation to other instrument classes or compound classes is not validated in this study.
- The theoretical R13C depends on correct molecular formulas; formula assignment errors propagate directly into the theoretical values.
- The linear relation R13C̄ ≈ 1.0816 × Cn is only stated to hold for Cn ≤ 92.

## Evidence
