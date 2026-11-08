---
name: molecular-formula-enumeration
description: Use when building a formula source or in-silico isotope-profile database
  for MS1 molecular formula annotation and you must bound the candidate formula space
  (organic exposome/metabolome composition) to a tractable set.
when_to_use_negative:
- You already have a restricted formula list from a known database (e.g. RefMet) —
  query it directly instead of exhaustively enumerating a chemical space
- Annotation requires MS2/structural evidence — enumeration plus MS1 isotope matching
  only assigns molecular formulas, not structures
- Peaks have not been extracted (no m/z–RT peaklists from IDSL.IPA); enumeration is
  useless without chromatographic peak candidates to match against
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: IDSL.UFA
  role: R package whose ECS enumeration rules (C/N boundary, extended SENIOR, halogen
    caps, max element count) define the chemical space and generate the IPDB
  repo: https://cran.r-project.org/package=IDSL.UFA
- name: R
  role: Implementation language for the enumeration rules and chemical-space table
    generation
- name: IDSL.UFAx
  role: Exhaustive enumeration variant applying a heuristic approach to enumerate
    vast chemical spaces (<10^27 molecular formulas) over a 15-element space
  repo: https://github.com/idslme/IDSL.UFAx
- name: IDSL.IPA
  role: Produces the peaklists (m/z–RT chromatographic information) that the enumerated
    formula space is subsequently matched against
  repo: https://github.com/idslme/IDSL.IPA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.2c00563
    title: IDSL.UFA Assigns High-Confidence Molecular Formula Annotations for Untargeted
      LC/HRMS Data Sets in Metabolomics and Exposomics
  source_content_hash: sha256:eae77cf4ab7a6356267cefd141647ccd8587b0278cd8d5419a6144af57f25410
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
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# molecular-formula-enumeration

## Summary

Systematically enumerate a tractable enumerated chemical space (ECS) of molecular formulas — constrained by four rules from IDSL.UFA — so that theoretical isotopic profiles (IPDB) can be precomputed for MS1-only formula annotation of untargeted LC/HRMS peaks.

## When to use

Use when building a formula source or in-silico isotope-profile database for MS1 molecular formula annotation and you must bound the candidate formula space (organic exposome/metabolome composition) to a tractable set.

## When NOT to use

- You already have a restricted formula list from a known database (e.g. RefMet) — query it directly instead of exhaustively enumerating a chemical space
- Annotation requires MS2/structural evidence — enumeration plus MS1 isotope matching only assigns molecular formulas, not structures
- Peaks have not been extracted (no m/z–RT peaklists from IDSL.IPA); enumeration is useless without chromatographic peak candidates to match against

## Inputs

- Element-space definition (list of allowed elements, e.g. the 16-element ECS) with atomic mass/isotope tables (IUPAC)
- Rule parameters: C/N elemental boundaries, halogen caps (e.g. (br+cl) ≤ 8, (br+cl+f+i) ≤ 31), maximum element count per formula
- Optional mass range or ion-mode targets for which candidates are needed

## Outputs

- chemical_space.tsv — structured table of enumerated formulas with element counts and monoisotopic masses, restricted to formulas passing all four rules
- Isotopic Profile DataBase (IPDB, .Rdata) of theoretical isotopic profiles generated from the enumerated space

## How to apply

Define the 16-element chemical space ECS (C, H, As, B, Br, Cl, F, I, K, N, Na, O, P, S, Se, Si) and enumerate compositions under four constraints implemented in R: (1) a C/N chemical-space elemental-boundary rule for organic compounds, (2) the extended SENIOR rule ensuring s- and p- valence shells are filled (valence feasibility), (3) halogen caps such as (br+cl) ≤ 8 and (br+cl+f+i) ≤ 31 for the blood exposome, and (4) a maximum-number-of-elements rule to skip unrealistically complex formulas. For each surviving formula, record element counts and the monoisotopic mass, then write the space as a structured table (chemical_space.tsv) for downstream IPDB generation in IDSL.UFA. Validation is to confirm only formulas passing all four rules are retained — i.e., every retained composition passes extended SENIOR and respects the halogen and element-count caps.

## Related tools

- **IDSL.UFA** (R package whose ECS enumeration rules (C/N boundary, extended SENIOR, halogen caps, max element count) define the chemical space and generate the IPDB) — https://cran.r-project.org/package=IDSL.UFA
- **R** (Implementation language for the enumeration rules and chemical-space table generation)
- **IDSL.UFAx** (Exhaustive enumeration variant applying a heuristic approach to enumerate vast chemical spaces (<10^27 molecular formulas) over a 15-element space) — https://github.com/idslme/IDSL.UFAx
- **IDSL.IPA** (Produces the peaklists (m/z–RT chromatographic information) that the enumerated formula space is subsequently matched against) — https://github.com/idslme/IDSL.IPA

## Examples

```
library(IDSL.UFA); UFA_workflow("Address of the UFA parameter spreadsheet")
```

## Evaluation signals

- chemical_space.tsv exists and every retained formula satisfies the extended SENIOR rule (no valence-infeasible compositions remain)
- No enumerated formula violates the halogen caps: (br+cl) ≤ 8 and (br+cl+f+i) ≤ 31 for blood-exposome settings
- No formula exceeds the configured maximum number of elements per formula
- All formulas draw only from the 16 allowed ECS elements; monoisotopic masses recorded per formula match IUPAC atomic mass calculations
- Downstream IPDB (.Rdata) is loadable and isotopic profiles reproduce the enumerated formula list

## Limitations

- Enumeration bounds are chemical-space-specific — halogen caps like (br+cl) ≤ 8 and (br+cl+f+i) ≤ 31 were calibrated for the blood exposome and may miss or over-include formulas for other sample types
- MS1-only isotope matching cannot distinguish compounds with identical or near-identical elemental compositions; annotations are formulas, not structures
- The exhaustive space can be enormous (<10^27 compositions for IDSL.UFAx's 15-element space); unconstrained enumeration is computationally intractable without the four rules

## Evidence

- [other] rule details from findings: [evidence span withheld — anchor status: not_found]
- [readme] readme ipdb: [evidence span withheld — anchor status: external_doc]
- [readme] readme ufax: [evidence span withheld — anchor status: external_doc]
- [readme] readme ms1: [evidence span withheld — anchor status: external_doc]
