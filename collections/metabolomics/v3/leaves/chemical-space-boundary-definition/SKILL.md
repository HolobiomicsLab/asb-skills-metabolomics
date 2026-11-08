---
name: chemical-space-boundary-definition
description: Use when you need a database-independent molecular formula candidate
  space for untargeted LC/HRMS annotation (e.g., exposomics compounds absent from
  metabolite databases) and must constrain which elemental compositions are enumerated
  before building a theoretical isotopic profile database (IPDB).
when_to_use_negative:
- Your formula candidates come from a known biochemical/database-dependent source
  (e.g., RefMet, reaction-network or MS2-based tools like SIRIUS/ZODIAC) — the enumeration
  space is for database-independent annotation where such databases would miss exposome
  compounds
- The raw enumerated space exceeds ~10^27 compositions and exhaustive enumeration
  is intractable — IDSL.UFAx applies a heuristic approach instead
- The goal is peak picking rather than formula-space definition — use IDSL.IPA first
  to generate mz-RT peaklists from mzXML/mzML/netCDF, then apply this skill to define
  the annotation space
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: IDSL.UFA
  role: Implements the four enumeration rules in R and consumes the constrained chemical
    space to generate the IPDB of theoretical isotopic profiles; driven via UFA_workflow()
    with a 30-parameter spreadsheet (PARAM0004 = IPDB .Rdata address).
  repo: https://cran.r-project.org/package=IDSL.UFA
- name: IDSL.UFAx
  role: Alternative exhaustive enumeration engine applying a heuristic approach to
    vast chemical spaces (<10^27 formulas) over a 15-element subset of the ECS.
- name: IDSL.IPA
  role: Upstream peak-picking package producing the chromatographic peaklists and
    aligned peak table that the constrained formula space is matched against.
  repo: github.com/idslme/IDSL.IPA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.2c00563
    title: IDSL.UFA Assigns High-Confidence Molecular Formula Annotations for Untargeted
      LC/HRMS Data Sets in Metabolomics and Exposomics
  source_content_hash: sha256:9aeb4147ab778f1f53f7d33dcdcdc66304aa8783ecc4c2169ce076e2c931f18e
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# chemical-space-boundary-definition

## Summary

Defines and constrains a database-independent enumerated chemical space (ECS) of molecular formulas by applying IDSL.UFA's four enumeration rules — the 16-element set, the C/N chemical-space elemental-boundary rule for organics, the extended SENIOR valence rule, halogen-count caps, and a maximum-number-of-elements cap — so a tractable candidate set can feed isotope-profile database (IPDB) generation for MS1 formula annotation.

## When to use

Use when you need a database-independent molecular formula candidate space for untargeted LC/HRMS annotation (e.g., exposomics compounds absent from metabolite databases) and must constrain which elemental compositions are enumerated before building a theoretical isotopic profile database (IPDB).

## When NOT to use

- Your formula candidates come from a known biochemical/database-dependent source (e.g., RefMet, reaction-network or MS2-based tools like SIRIUS/ZODIAC) — the enumeration space is for database-independent annotation where such databases would miss exposome compounds
- The raw enumerated space exceeds ~10^27 compositions and exhaustive enumeration is intractable — IDSL.UFAx applies a heuristic approach instead
- The goal is peak picking rather than formula-space definition — use IDSL.IPA first to generate mz-RT peaklists from mzXML/mzML/netCDF, then apply this skill to define the annotation space

## Inputs

- Choice of element set (16-element ECS: C, H, As, B, Br, Cl, F, I, K, N, Na, O, P, S, Se, Si)
- Chemical-space scope defining halogen caps (e.g., blood exposome: (br+cl) ≤ 8, (br+cl+f+i) ≤ 31)
- Maximum-number-of-elements cap per formula
- Valence-shell rule definition for the extended SENIOR rule

## Outputs

- chemical_space.tsv: enumerated formula candidates with element counts and monoisotopic masses, all passing the four rules
- Isotopic Profile DataBase (IPDB, .Rdata) of theoretical isotopic profiles built from the constrained space
- Validation log confirming exclusion of valence-infeasible, halogen-exceeding, and overly element-rich formulas

## How to apply

Define the ECS over 16 elements (C, H, As, B, Br, Cl, F, I, K, N, Na, O, P, S, Se, Si) as IDSL.UFA does in R. Enumerate elemental compositions and apply the four constraints: (1) the C/N chemical-space elemental-boundary rule to restrict to organic-compound compositions; (2) the extended SENIOR rule, which checks that s- and p- valence shells are filled so valence-infeasible formulas are excluded; (3) maximum-halogen caps — e.g., (br+cl) ≤ 8 and (br+cl+f+i) ≤ 31 for the blood exposome chemical space; and (4) a maximum-number-of-elements rule to skip unrealistically complex formulas. Record element counts and monoisotopic mass per surviving candidate, and write the enumerated space as chemical_space.tsv (or use IDSL.UFA's parameter spreadsheet / IPDB .Rdata path) for downstream IPDB generation. Halogen-cap values are chemical-space-specific: choose them per study scope (the article's caps are given for the blood exposome), and verify every retained formula passes all four rules.

## Related tools

- **IDSL.UFA** (Implements the four enumeration rules in R and consumes the constrained chemical space to generate the IPDB of theoretical isotopic profiles; driven via UFA_workflow() with a 30-parameter spreadsheet (PARAM0004 = IPDB .Rdata address).) — https://cran.r-project.org/package=IDSL.UFA
- **IDSL.UFAx** (Alternative exhaustive enumeration engine applying a heuristic approach to vast chemical spaces (<10^27 formulas) over a 15-element subset of the ECS.)
- **IDSL.IPA** (Upstream peak-picking package producing the chromatographic peaklists and aligned peak table that the constrained formula space is matched against.) — github.com/idslme/IDSL.IPA

## Examples

```
library(IDSL.UFA); UFA_workflow("Address of the UFA parameter spreadsheet")
```

## Evaluation signals

- Every formula retained in chemical_space.tsv passes the extended SENIOR rule (no valence-infeasible compositions remain)
- No retained formula exceeds the halogen caps (e.g., (br+cl) ≤ 8 and (br+cl+f+i) ≤ 31 for the blood exposome space)
- Every retained formula uses only elements from the 16-element ECS and respects the maximum-number-of-elements cap
- Monoisotopic mass recorded for each candidate matches the composition implied by its element counts
- chemical_space.tsv exists, is loadable for downstream IPDB generation, and contains only four-rule-passing formulas

## Limitations

- Halogen caps are chemical-space-specific tuning parameters; caps validated for the blood exposome (e.g., 8 for br+cl) may not transfer to other sample types
- The maximum-number-of-elements rule deliberately skips highly complex formulas, which could exclude legitimately multi-element compositions
- Enumeration depends on a predefined element set (16 elements in IDSL.UFA; 15 in IDSL.UFAx, which omits Se); formulas containing elements outside this set are never generated
- MS1-only isotope matching constrains annotation confidence — top-hit rates in validation ranged 54.31–85.51%, with 90.58–100% only within the top five hits

## Evidence

- [other] then applies four rules in R to constrain it: a C/N chemical-space elemental-boundary rule for organic compounds, an extended SENIOR rule ensuring s- and p- valence shells are filled: [evidence span withheld — anchor status: not_found]
- [other] maximum-halogen caps (e.g., (br+cl) ≤ 8 and (br+cl+f+i) ≤ 31 for the blood exposome), and a maximum-number-of-elements rule to skip unrealistically complex formulas: [evidence span withheld — anchor status: not_found]
- [readme] Generating comprehensive *in-silico* theoretical libraries (known as [IPDB]) using natural isotopic distribution profiles: [evidence span withheld — anchor status: external_doc]
- [readme] IDSL.UFAx applies a hursitc approach to enumerate vast chemical spaces: [evidence span withheld — anchor status: not_found]
- [readme] annotate a number of peaks from the [**IDSL.IPA**](https://github.com/idslme/IDSL.IPA) peaklists using an exhaustive chemical enumeration-based approach: [evidence span withheld — anchor status: external_doc]
