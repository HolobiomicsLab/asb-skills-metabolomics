---
name: molecular-formula-parsing
description: Use when the input is a formula-annotated aligned peak table (output
  of IDSL.UFA) whose formula strings must be decomposed into element counts (C, H,
  Br, Cl, F, I, …) before class or progression analysis.
when_to_use_negative:
- The input is already a numeric element-count or atom table — parsing formula strings
  again would be redundant.
- The goal is isotopic-profile matching or formula assignment itself; that is IDSL.UFA/IPDB
  territory, not formula-string parsing.
- Peak lists are raw, un-annotated IDSL.IPA peaklists without molecular formula annotations
  — there is nothing to parse until UFA annotation runs.
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: IDSL.UFA
  role: Produces the annotated aligned peak table whose formula strings are parsed;
    also supplies the formula-class grouping (detect_formula_sets) that consumes the
    parsed counts
  repo: https://cran.r-project.org/package=IDSL.UFA
- name: IDSL.IPA
  role: Upstream generator of individual peaklists and the aligned peak table (m/z–RT
    chromatographic info) that UFA annotates and from which formulas are taken
  repo: https://github.com/idslme/IDSL.IPA
- name: R
  role: 'Implementation environment: formula tokenization and element-count extraction
    are scripted in R on the UFA output tables'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.2c00563
    title: IDSL.UFA Assigns High-Confidence Molecular Formula Annotations for Untargeted
      LC/HRMS Data Sets in Metabolomics and Exposomics
  source_content_hash: sha256:a0c50f9381347be73db698a6e40335ef3c1a5ae245c1be8a5b1b88084be8beb9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.IPA
  - IDSL.UFA
  - R
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# molecular-formula-parsing

## Summary

Parses molecular formula strings from an IDSL.UFA-annotated aligned peak table into per-element atom counts, enabling downstream formula-class detection such as ΔH/ΔC chain progressions and fixed-carbon halogenated classes.

## When to use

Use when the input is a formula-annotated aligned peak table (output of IDSL.UFA) whose formula strings must be decomposed into element counts (C, H, Br, Cl, F, I, …) before class or progression analysis.

## When NOT to use

- The input is already a numeric element-count or atom table — parsing formula strings again would be redundant.
- The goal is isotopic-profile matching or formula assignment itself; that is IDSL.UFA/IPDB territory, not formula-string parsing.
- Peak lists are raw, un-annotated IDSL.IPA peaklists without molecular formula annotations — there is nothing to parse until UFA annotation runs.

## Inputs

- Aligned annotated molecular formula table (.csv) produced by IDSL.UFA (per-peak formula hits with median rank and detection frequency)
- Individual per-file annotated peaklists from the IDSL.IPA/UFA workflow
- UFA_parameter.xlsx settings controlling which formula hits are reported (e.g. PARAM0006 aggregated aligned-table statistics)

## Outputs

- Element-count table mapping each unique molecular formula string to numeric counts of C, H, N, O, S, P, Br, Cl, F, I, etc.
- Normalized list of unique formulas ready for class-detection grouping (input to detect_formula_sets)

## How to apply

Load the aligned annotated molecular formula table produced by IDSL.UFA, where each peak row carries candidate formulas such as those with a median rank of ≤ 5 or a top rank ≤ 30 among ≤ 30 hits. Extract each unique formula string and tokenize it into element–count pairs, treating counts of 1 as implicit so that e.g. 'C12H4Cl4' yields C=12, H=4, Cl=4. Preserve elements relevant to the class-detection grouping — carbon, hydrogen, and the halogens Br, Cl, F, I whose sum Σ(H+Br+Cl+F+I) defines fixed-carbon halogenated classes like PCBs and PBDEs. Handle adduct/ion annotations and formula tags that IDSL.UFA writes in its annotated tables so that only the bare formula composition is parsed. The output is a tidy table of one row per formula with numeric atom counts that feeds directly into the class detection step (e.g. 'detect_formula_sets'), avoiding ad-hoc regex in every downstream analysis.

## Related tools

- **IDSL.UFA** (Produces the annotated aligned peak table whose formula strings are parsed; also supplies the formula-class grouping (detect_formula_sets) that consumes the parsed counts) — https://cran.r-project.org/package=IDSL.UFA
- **IDSL.IPA** (Upstream generator of individual peaklists and the aligned peak table (m/z–RT chromatographic info) that UFA annotates and from which formulas are taken) — https://github.com/idslme/IDSL.IPA
- **R** (Implementation environment: formula tokenization and element-count extraction are scripted in R on the UFA output tables)

## Examples

```
library(IDSL.UFA); counts <- parse_formula_strings(unique(aligned_table$MolecularFormula))  # feeds detect_formula_sets
```

## Evaluation signals

- Re-parsing counts and re-serializing the formula string reproduces the original formula exactly (round-trip identity check).
- Every row of the annotated table yields a parseable formula or an explicit non-formula annotation; no silent drop or duplicate formula entries.
- Counts are non-negative integers, and halogen sums Σ(H+Br+Cl+F+I) computed from parsed counts match manual spot checks for known classes (e.g. PCBs: constant C with varying Cl).
- Formulas flagged with median rank ≤ 5 / top rank ≤ 30 in the aligned table are all represented in the parsed output.
- Downstream detect_formula_sets runs without errors and its chain-progressions (ΔH/ΔC = 2 or 1/2) and fixed-carbon halogen classes reproduce counts reported by IDSL.UFA.

## Limitations

- Only meaningful for formulas IDSL.UFA actually assigned — database-dependent miss lists (exposure compounds poorly represented in biochemical databases) leave gaps in the parsed formula space.
- Parsing must respect UFA's ion/adduct tags; mis-stripping them corrupts the elemental composition.
- Accuracy is bounded by the annotation confidence of the source table (54.31–85.51% top-hit true positives in the article), so parsed counts inherit those error rates.

## Evidence

- [readme] Input is a formula-annotated aligned peak table: [evidence span withheld — anchor status: external_doc]
- [methods] parse each formula into element counts (C, H, Br, Cl, F, I, etc.): [evidence span withheld — anchor status: not_found]
- [readme] UFA runs on IDSL.IPA peaklists; the source tables to parse: [evidence span withheld — anchor status: not_found]
