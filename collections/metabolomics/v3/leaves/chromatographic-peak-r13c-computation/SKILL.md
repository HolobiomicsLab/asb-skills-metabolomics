---
name: chromatographic-peak-r13c-computation
description: Reference for quantifying the 12C/13C isotope-pair intensity ratio of
  individual chromatographic peaks and correlating it with molecular formulas or predicted
  carbon number.
when_to_use_negative:
- When peaks have already been aligned and annotated into a feature table with no
  isotopologue-pairing metadata (nIsoPair/R13C columns) — IDSL.IPA must be re-run
  on the raw mzXML/mzML to regenerate these properties.
- When the analyte set is expected to contain non-carbon-containing compounds, since
  IDSL.IPA specifically isolates signals pertaining to carbon-containing compounds
  via 12C/13C ion pairing.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: IDSL.IPA
  role: Generates the peaklists (peak areas, nIsoPair, R13C properties) from raw Fusion
    Orbitrap LC/HRMS data of the authentic standards; its 12C/13C ion-pairing algorithm
    underlies the experimental R13C values.
  repo: https://github.com/idslme/IDSL.IPA
- name: enviPat
  role: Computes theoretical isotopic profiles from molecular formulas, providing
    the theoretical R13C values used in the Spearman correlation agreement test.
- name: R
  role: Runtime environment executing the IDSL.IPA peak processing and the enviPat/theoretical-profile
    and correlation calculations.
  repo: https://cran.r-project.org/package=IDSL.IPA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00120
    title: IDSL.IPA Characterizes the Organic Chemical Space in Untargeted LC/HRMS
      Data Sets
  source_content_hash: sha256:b019b72d2f236d2d1b9d686123bcb88566aa51774fd6860289db115f6259ec3e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.IPA
  tools:
  - IDSL.IPA
  - R
  - enviPat
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# chromatographic-peak-r13c-computation

## Summary

Computes per-peak R13C values (the ratio of cumulated 13C isotope-pair intensity to 12C monoisotopic intensity) from IDSL.IPA peaklists for authentic standards, and validates them against theoretical R13C values from enviPat isotopic profiles to confirm that R13C tracks carbon number.

## When to use

Reference for quantifying the 12C/13C isotope-pair intensity ratio of individual chromatographic peaks and correlating it with molecular formulas or predicted carbon number.

## When NOT to use

- When peaks have already been aligned and annotated into a feature table with no isotopologue-pairing metadata (nIsoPair/R13C columns) — IDSL.IPA must be re-run on the raw mzXML/mzML to regenerate these properties.
- When the analyte set is expected to contain non-carbon-containing compounds, since IDSL.IPA specifically isolates signals pertaining to carbon-containing compounds via 12C/13C ion pairing.

## Inputs

- Untargeted LC/HRMS peaklists produced by IDSL.IPA containing peak areas, nIsoPair counts, and R13C chromatographic peak properties
- Authentic-standard reference compounds with known molecular formulas (e.g., the 367 standards from Zenodo 10.5281/zenodo.4743898, Fusion Orbitrap data)
- Molecular formulas for each standard (used with enviPat to compute theoretical isotopic profiles)

## Outputs

- Per-compound experimental R13C values derived from IDSL.IPA peak properties
- Theoretical R13C values from enviPat isotopic profile calculations
- Spearman correlation statistics (p-value, ρ) quantifying theoretical-vs-experimental agreement
- Fitted linear relationship R13C̄ ≈ 1.0816 × Cn (Cn ≤ 92) for carbon-number prediction
- Combined results table (e.g., r13c_standards_results.csv) with formula, carbon number, and both R13C estimates per compound

## How to apply

Process the raw LC/HRMS data (e.g., the 367 authentic standards from the Fusion Orbitrap dataset 10.5281/zenodo.4743898) with the IDSL.IPA pipeline to generate per-peak chromatographic properties, including nIsoPair and R13C. For each authentic standard, compute the experimental mean R13C as (cumulated 13C isotope-pair intensity / 12C monoisotopic intensity) × 100, aggregating over the standards' replicate measurements. In parallel, use the enviPat R package to calculate theoretical isotopic profiles from each standard's molecular formula and derive theoretical R13C values. The decision point is the agreement test: compute a Spearman rank correlation between the two R13C sets, which should reproduce the reported p-value ≤ 2.2×10⁻¹⁶ and ρ ≤ 0.92. The rationale is that R13C is roughly linear in carbon number — R13C̄ ≈ 1.0816 × Cn for Cn ≤ 92 — so matching theoretical and experimental values confirms both the isotope-pairing logic in IDSL.IPA and the formula-based carbon-count prediction.

## Related tools

- **IDSL.IPA** (Generates the peaklists (peak areas, nIsoPair, R13C properties) from raw Fusion Orbitrap LC/HRMS data of the authentic standards; its 12C/13C ion-pairing algorithm underlies the experimental R13C values.) — https://github.com/idslme/IDSL.IPA
- **enviPat** (Computes theoretical isotopic profiles from molecular formulas, providing the theoretical R13C values used in the Spearman correlation agreement test.)
- **R** (Runtime environment executing the IDSL.IPA peak processing and the enviPat/theoretical-profile and correlation calculations.) — https://cran.r-project.org/package=IDSL.IPA

## Examples

```
library(IDSL.IPA); IPA_workflow("IPA_parameters.xlsx")  # then in R: cor.test(theoretical_R13C, experimental_R13C, method="spearman")
```

## Evaluation signals

- Spearman correlation between theoretical and experimental R13C across the 367 authentic standards must reproduce the reported significance (p-value ≤ 2.2×10⁻¹⁶) and effect size (ρ ≤ 0.92).
- Fitted linear regression of mean experimental R13C against carbon number must recover the reported slope of ≈1.0816 for Cn ≤ 92.
- Every peak included in the R13C aggregation must have passed IDSL.IPA's isotopologue-pairing and peak-quality thresholds (peaks that failed are excluded).
- Per-compound output must contain, for each standard, its molecular formula, carbon number, experimental R13C, and theoretical R13C — a missing or null formula invalidates the comparison for that compound.
- Experimental R13C should be computed only from the 12C monoisotopic isotopologue as denominator, not from a redundant isotope peak that IDSL.IPA has already removed.

## Limitations

- IDSL.IPA detects only the most abundant monoisotopic isotopologues from 12C/13C isotope clusters and removes redundant isotope peaks, so R13C values depend on the pipeline's isotope-pairing logic rather than raw full-scan intensities.
- The linear carbon-number prediction R13C̄ ≈ 1.0816 × Cn is bounded by the observed carbon range (Cn ≤ 92); extrapolation beyond that is unsupported.
- Agreement was validated only on common-metabolite authentic standards; application to exposomics compounds outside this space has not been demonstrated.
- Low-abundance peaks that passed the S/N ratio threshold but were manually rejected indicate that IDSL.IPA's automated filtering may still need additional rules, which could affect R13C reproducibility for weak signals.

## Evidence

- [results] Theoretical vs experimental R13C agreement on 367 authentic standards: "the theoretical and experimental R13C¯ showed a good agreement (Spearman correlation test, p-value ≤ 2.2*10−16, ρ ≤ 0.92) for 367 authentic standards"
