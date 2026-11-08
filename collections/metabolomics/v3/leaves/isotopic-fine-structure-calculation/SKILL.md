---
name: isotopic-fine-structure-calculation
description: Use when you need theoretical isotope patterns or carbon-number estimates
  for molecular formulas — e.g., validating R13C ratios from IDSL.IPA peaklists or
  assigning elemental compositions from MS1 data.
when_to_use_negative:
- Input peaks lack MS1 isotope information (e.g., MS2-only fragments or spectra without
  12C/13C pairs) — IDSL.IPA isotope pairing and R13C calculation are undefined.
- You need structural (not formula-level) annotation requiring MS2 fragmentation —
  this skill only validates carbon content via isotopic profiles; use MS2-based pipelines
  instead.
- Compounds with no carbon (pure inorganic species) — the 12C/13C pairing approach
  and R13C metric assume carbon-containing analytes.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: enviPat
  role: Calculate theoretical isotopic profiles for molecular formulas to derive theoretical
    R13C values
- name: IDSL.IPA
  role: Generate peaklists with nIsoPair and R13C peak properties from raw LC/HRMS
    data; identifies 12C/13C ion pairs per scan
  repo: https://github.com/idslme/IDSL.IPA
- name: IDSL.UFA
  role: Annotate IDSL.IPA peaklists with molecular formulas via isotopic profile matching
    (IPDB libraries) using MS1 data
  repo: https://github.com/idslme/IDSL.UFA
- name: R
  role: Execution environment for enviPat and IDSL.IPA scripts
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00120
    title: IDSL.IPA Characterizes the Organic Chemical Space in Untargeted LC/HRMS
      Data Sets
  source_content_hash: sha256:3df354a0da8537b94efda009c7dafeb1d2a236188afff146fbbd80176eb5d7e9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.IPA
  tools:
  - IDSL.IPA
  - IDSL.UFA
  - R
  - enviPat
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# isotopic-fine-structure-calculation

## Summary

Compute theoretical isotopic profiles for molecular formulas with enviPat and compare them against experimentally observed 12C/13C isotope-pair ratios (R13C) from IDSL.IPA peaklists to validate carbon-containing compound annotations and estimate carbon number.

## When to use

Use when you need theoretical isotope patterns or carbon-number estimates for molecular formulas — e.g., validating R13C ratios from IDSL.IPA peaklists or assigning elemental compositions from MS1 data.

## When NOT to use

- Input peaks lack MS1 isotope information (e.g., MS2-only fragments or spectra without 12C/13C pairs) — IDSL.IPA isotope pairing and R13C calculation are undefined.
- You need structural (not formula-level) annotation requiring MS2 fragmentation — this skill only validates carbon content via isotopic profiles; use MS2-based pipelines instead.
- Compounds with no carbon (pure inorganic species) — the 12C/13C pairing approach and R13C metric assume carbon-containing analytes.

## Inputs

- Molecular formulas (e.g., for metabolites or exposomics compounds)
- IDSL.IPA peaklists with peak areas, nIsoPair and R13C chromatographic peak properties
- MS1-level LC/HRMS raw data (mzXML, mzML, netCDF) from Fusion Orbitrap or similar instruments
- Reference compound dataset (e.g., Zenodo 10.5281/zenodo.4743898)

## Outputs

- Theoretical isotopic profiles per molecular formula
- Experimental R13C values per compound (mean of cumulated 13C isotope-pair intensity / 12C monoisotopic intensity × 100)
- Carbon-number predictions via R13C̄ ≈ 1.0816 × Cn (Cn ≤ 92)
- Spearman correlation statistics (p-value, rho) between theoretical and experimental R13C
- r13c_standards_results.csv with per-compound experimental R13C, theoretical R13C, molecular formula, carbon number, and correlation statistics

## How to apply

Take molecular formulas for the compounds of interest (e.g., 367 authentic standards from the Zenodo 4743898 Fusion Orbitrap dataset) and use the enviPat R package to calculate theoretical isotopic profiles. For each compound, compute the experimental R13C as (cumulated 13C isotope-pair intensity / 12C monoisotopic intensity) × 100 from IDSL.IPA peak properties such as nIsoPair and R13C. Compare theoretical and experimental R13C across the standards using a Spearman rank correlation — the reported agreement for 367 metabolite standards was p ≤ 2.2e-16 with ρ ≤ 0.92. Fit the linear relation R13C̄ ≈ 1.0816 × Cn (Cn ≤ 92) to predict the number of carbons from the mean R13C value. Use these profiles to confirm that peaks flagged by IDSL.IPA's 12C/13C ion pairing are genuine carbon-containing species rather than redundant isotope peaks.

## Related tools

- **enviPat** (Calculate theoretical isotopic profiles for molecular formulas to derive theoretical R13C values)
- **IDSL.IPA** (Generate peaklists with nIsoPair and R13C peak properties from raw LC/HRMS data; identifies 12C/13C ion pairs per scan) — https://github.com/idslme/IDSL.IPA
- **IDSL.UFA** (Annotate IDSL.IPA peaklists with molecular formulas via isotopic profile matching (IPDB libraries) using MS1 data) — https://github.com/idslme/IDSL.UFA
- **R** (Execution environment for enviPat and IDSL.IPA scripts)

## Examples

```
Rscript -e 'library(enviPat); data(isotopes); chem <- "C10H12N2O"; prof <- isopattern(chem, isotopes); x <- unlist(prof[[1]]); cn <- 10; r13c <- sum(x[2:length(x)]) / x[1] * 100; cat(r13c, 1.0816*cn)'
```

## Evaluation signals

- Spearman correlation between theoretical and experimental R13C across the 367 standards must reproduce p ≤ 2.2e-16 and ρ ≤ 0.92.
- Fitted slope of R13C̄ vs carbon number must match the reported 1.0816 coefficient for Cn ≤ 92.
- Each 12C m/z peak used in only one m/z cluster — verify no overlapping-cluster double counting of isotope pairs.
- IDSL.IPA should detect only the most abundant monoisotopic isotopologues; check redundant isotope peaks are removed before R13C computation.
- Per-compound output table (r13c_standards_results.csv) must include experimental R13C, theoretical R13C, molecular formula, carbon number, and correlation statistics.

## Limitations

- IDSL.IPA-aligned data matrices require further processing for adducts, fragments, statistics filtering, and molecular formula annotation — the R13C skill alone does not complete structural assignment.
- Low-abundance peaks may pass S/N thresholds but be manually rejected; additional auto-filter rules are still needed for such rejected low-abundant EICs.
- The linear R13C̄ ≈ 1.0816 × Cn relationship is validated only for carbon number up to 92 (Cn ≤ 92); extrapolation beyond this range is unsupported.
- No software version stated for IDSL.IPA in the article; version-specific behavior of isotope pairing is undocumented.
- Theoretical profiles depend on the enviPat atomic mass tables (IUPAC); changes in isotope abundance tables affect R13C thresholds.

## Evidence

- [readme] UFA isotopic profile matching description: [evidence span withheld — anchor status: not_found]
