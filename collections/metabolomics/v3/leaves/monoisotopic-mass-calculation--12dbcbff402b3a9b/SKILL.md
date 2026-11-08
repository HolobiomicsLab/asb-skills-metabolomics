---
name: monoisotopic-mass-calculation
description: Use when enumerating formula candidates from elemental compositions (e.g.,
  a 16-element ECS of C, H, As, B, Br, Cl, F, I, K, N, Na, O, P, S, Se, Si) and monoisotopic
  masses must be recorded per candidate to build or reuse an IPDB for LC/HRMS formula
  annotation.
when_to_use_negative:
- Input is raw LC/HRMS data (mzXML/mzML/netCDF) that has not been peak-picked — IDSL.IPA
  must first extract peaks; this skill assumes peaklists exist.
- A pre-calculated IPDB already covers the chemical space (e.g., RefMetDB positive/negative-mode
  IPDBs from Zenodo) — reload it via PARAM0004 instead of re-enumerating formulas
  and recomputing masses.
- The target chemical space requires elements outside the ECS element list (e.g.,
  metals outside K/Na) — the enumeration and mass tables do not cover them.
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3391
tools:
- name: IDSL.UFA
  role: Computes theoretical isotopic profiles for molecular formulas with monoisotopic
    masses, builds IPDB (.Rdata) libraries, and matches them against MS1 LC/HRMS data;
    consumes the UFA parameter spreadsheet
  repo: https://cran.r-project.org/package=IDSL.UFA
- name: IDSL.UFAx
  role: Exhaustive enumeration variant that performs elemental composition calculations
    over 15 elements (C, B, Br, Cl, K, S, Si, N, H, As, F, I, Na, O, P) for vast chemical
    spaces
- name: IDSL.IPA
  role: Generates the individual peaklists and aligned peak tables (m/z–RT chromatographic
    information) that the mass-calculated formula space is matched against
  repo: github.com/idslme/IDSL.IPA
- name: R
  role: Implementation language for enumerating the constrained ECS, computing monoisotopic
    masses, and writing chemical_space.tsv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.2c00563
    title: IDSL.UFA Assigns High-Confidence Molecular Formula Annotations for Untargeted
      LC/HRMS Data Sets in Metabolomics and Exposomics
  source_content_hash: sha256:17453d74b0efdece568286054b700a02d56915778b3295250b296be1a01d6020
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.IPA
  - IDSL.UFA
  - IDSL.UFAx
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# monoisotopic-mass-calculation

## Summary

Computes exact monoisotopic masses (and records element counts) for enumerated molecular formula candidates over a defined elemental chemical space (ECS), so that candidates can feed theoretical isotopic-profile database (IPDB) generation and MS1 matching in IDSL.UFA.

## When to use

Use when enumerating formula candidates from elemental compositions (e.g., a 16-element ECS of C, H, As, B, Br, Cl, F, I, K, N, Na, O, P, S, Se, Si) and monoisotopic masses must be recorded per candidate to build or reuse an IPDB for LC/HRMS formula annotation.

## When NOT to use

- Input is raw LC/HRMS data (mzXML/mzML/netCDF) that has not been peak-picked — IDSL.IPA must first extract peaks; this skill assumes peaklists exist.
- A pre-calculated IPDB already covers the chemical space (e.g., RefMetDB positive/negative-mode IPDBs from Zenodo) — reload it via PARAM0004 instead of re-enumerating formulas and recomputing masses.
- The target chemical space requires elements outside the ECS element list (e.g., metals outside K/Na) — the enumeration and mass tables do not cover them.

## Inputs

- Enumerated molecular formula candidates as element-count vectors over the ECS elements (C, H, As, B, Br, Cl, F, I, K, N, Na, O, P, S, Se, Si)
- IUPAC atomic mass/isotope tables (isotopesmatter.com) used for monoisotopic mass and isotope-distribution computation
- Constraint parameters: halogen caps (e.g., (br+cl) ≤ 8, (br+cl+f+i) ≤ 31 for the blood exposome) and maximum element count per formula
- IDSL.IPA peaklists and aligned peak table (individual m/z–RT peak information) as downstream consumers of the computed masses

## Outputs

- chemical_space.tsv: structured table of formula candidates passing all four rules, with element counts and monoisotopic mass per formula
- IPDB (.Rdata): in-silico theoretical isotope profile database built from natural isotopic distribution profiles for the enumerated formulas
- Ranked molecular-formula annotations per peak (median rank and detection frequency on the aligned annotated peak table)

## How to apply

Represent each candidate formula as a vector of element counts and compute its monoisotopic mass from atomic mass tables provided by IUPAC (as used by the combinatorial isotope-prediction rules in IDSL.UFA). Before recording the mass, filter compositions through the paper's four constraints on the ECS: the C/N chemical-space elemental-boundary rule for organic compounds, the extended SENIOR rule (s- and p- valence shells filled, i.e., valence-consistent compositions), halogen caps such as (br+cl) ≤ 8 and (br+cl+f+i) ≤ 31 for the blood exposome, and a maximum-number-of-elements cap. Retain only formulas passing all four rules, writing element counts and monoisotopic mass per candidate to a structured table (chemical_space.tsv) suitable for downstream IPDB generation in IDSL.UFA. The resulting masses underpin theoretical isotopic profiles that IDSL.UFA matches against MS1 isotopic profiles of peaks from IDSL.IPA peaklists; ranked hits are summarized on the aligned peak table via detection frequency and median rank.

## Related tools

- **IDSL.UFA** (Computes theoretical isotopic profiles for molecular formulas with monoisotopic masses, builds IPDB (.Rdata) libraries, and matches them against MS1 LC/HRMS data; consumes the UFA parameter spreadsheet) — https://cran.r-project.org/package=IDSL.UFA
- **IDSL.UFAx** (Exhaustive enumeration variant that performs elemental composition calculations over 15 elements (C, B, Br, Cl, K, S, Si, N, H, As, F, I, Na, O, P) for vast chemical spaces)
- **IDSL.IPA** (Generates the individual peaklists and aligned peak tables (m/z–RT chromatographic information) that the mass-calculated formula space is matched against) — github.com/idslme/IDSL.IPA
- **R** (Implementation language for enumerating the constrained ECS, computing monoisotopic masses, and writing chemical_space.tsv)

## Examples

```
library(IDSL.UFA); UFA_workflow("Address of the UFA parameter spreadsheet")
```

## Evaluation signals

- Every formula retained in chemical_space.tsv passes all four rules: extended SENIOR (valence-consistent), C/N elemental-boundary rule, halogen caps ((br+cl) ≤ 8, (br+cl+f+i) ≤ 31 for the blood exposome), and the maximum-element-count cap.
- Each row's monoisotopic mass matches an independent computation from IUPAC atomic mass tables for the recorded element counts.
- The IPDB (.Rdata) built from the chemical space loads successfully and is addressable via PARAM0004 in the UFA parameter spreadsheet.
- Downstream UFA_workflow runs annotate benchmark peaklists (e.g., ST002263) with formulas whose median ranks fall in the ≤ 5 reporting band used in the paper.
- No formula appears in chemical_space.tsv that violates SENIOR-type valence consistency (spot-check against a valence-infeasible composition that must be excluded).

## Limitations

- Enumeration spaces can be astronomically large (IDSL.UFAx notes chemical spaces below 10^27 molecular formulas handled by a heuristic approach), so full enumeration is not always tractable and pruning rules are essential.
- Only a fixed element set is supported — IDSL.UFAx covers 15 elements (without Se) and the paper's ECS covers 16 (adding Se) — compositions with other elements cannot be annotated.
- Theoretical isotope-profile matching targets carbon-containing compounds, per the IDSL.UFA background description.

## Evidence

- [readme] The isotopic pattern for a chemical structure can be accurately predicted using a set of combinatorial rules that uses atomic mass tables provided by the: [evidence span withheld — anchor status: external_doc]
- [readme] This package can perform elemental composition calculations using the following 15 elements : C, B, Br, Cl, K, S, Si, N, H, As, F, I, Na, O, and P.: [evidence span withheld — anchor status: external_doc]
