---
name: smarts-structural-rule-encoding
description: Use when building a structure-based chemical taxonomy or classifier where
  each node/category must be matched to compounds by computable structural rules (e.g.
  SMARTS, Markush, or feature-logic) rather than expert curation alone.
when_to_use_negative:
- The task requires classifying compounds by biological activity, use, or MeSH-style
  semantic terms rather than purely structural features — ClassyFire's rules use 'only
  chemical structures and structural features'
- Structures cannot be standardized (e.g. mixtures or ill-defined polymers) where
  normalization/aromatization cannot produce a consistent match target
- You need per-class quantitative performance metrics not yet published for a specific
  category set — the paper notes IUPAC-name parsing rules may be insufficient for
  some classes
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0081
tools:
- name: ClassyFire
  role: Applies the encoded SMARTS/Markush/feature rules via superstructure-search
    and property calculations to assign compounds to taxonomy categories
  repo: https://classyfire.wishartlab.com
- name: ChemAxon Marvin
  role: Used to express the ~4% of category rules that require Markush patterns instead
    of plain SMARTS
- name: OBO-Edit
  role: Encodes the rule-annotated taxonomy (category nodes with their computable
    rules) into the downloadable OBO file
- name: OPSIN
  role: Converts IUPAC names of query compounds into structures before rule matching
- name: ChemAxon JChem API 15.5.25.0
  role: Computes physico-chemical/structural properties (e.g. RingCount, AtomCount)
    used in feature-logic category rules
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-016-0174-y
    title: 'ClassyFire: automated chemical classification with a comprehensive, computable
      taxonomy'
  source_content_hash: sha256:67313ba30c51a611fe985afa2cd10f804a2b94079693d5204de5f996adf902d0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemAxon JChem API 15.5.25.0
  - ChemAxon Marvin
  - ClassyFire
  - OBO-Edit
  - OPSIN
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0081
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# smarts-structural-rule-encoding

## Summary

Encodes each category of a structural chemical taxonomy as a computable rule — a SMARTS string (>9000 generated), a Markush pattern, or a logical expression over structural features — so that compounds can be matched to categories purely from structure.

## When to use

Use when building a structure-based chemical taxonomy or classifier where each node/category must be matched to compounds by computable structural rules (e.g. SMARTS, Markush, or feature-logic) rather than expert curation alone.

## When NOT to use

- The task requires classifying compounds by biological activity, use, or MeSH-style semantic terms rather than purely structural features — ClassyFire's rules use 'only chemical structures and structural features'
- Structures cannot be standardized (e.g. mixtures or ill-defined polymers) where normalization/aromatization cannot produce a consistent match target
- You need per-class quantitative performance metrics not yet published for a specific category set — the paper notes IUPAC-name parsing rules may be insufficient for some classes

## Inputs

- Chemical structure patterns for each taxonomy category (SMARTS strings, Markush patterns, or feature-based logical expressions)
- Query compound structures (SMILES/MOL) parsed or converted from IUPAC names via OPSIN where needed
- Manually compiled feature-to-category dictionary with weights

## Outputs

- Per-category computable structural rules embedded in an OBO-format taxonomy file (plus JSON mirror)
- Assigned taxonomy category for each query compound, with parent–child category reduction applied
- Direct parent category (largest structural feature defining the compound)

## How to apply

For each taxonomy category, derive a computable structural rule: prefer a SMARTS string generated from the category's characteristic structure pattern (the article reports >9000 SMARTS rules); when SMARTS cannot express the pattern, fall back to a Markush pattern (about 4% of patterns) authored in ChemAxon's Marvin; for categories defined by global properties rather than substructures (e.g. alkanes), write a logical expression over computed features such as RingCount and AtomCount (e.g. RingCount(A)=0 AND AtomCount(C,A)>0 AND AtomCount(H,A)=2*AtomCount(C,A)+2); for categories like phenylpropanoids, polyketides, and alkaloids, define the rule as a union of child subcategories. Standardize both reference patterns and query compounds with normalization and aromatization before matching so that equivalent resonance/tautomer forms map consistently. Store rules alongside the category in OBO format (built with OBO-Edit) with a JSON mirror, and wire the rules into the rule-based category assignment step of the classification workflow (feature extraction → rule mapping → category reduction → direct-parent selection).

## Related tools

- **ClassyFire** (Applies the encoded SMARTS/Markush/feature rules via superstructure-search and property calculations to assign compounds to taxonomy categories) — https://classyfire.wishartlab.com
- **ChemAxon Marvin** (Used to express the ~4% of category rules that require Markush patterns instead of plain SMARTS)
- **OBO-Edit** (Encodes the rule-annotated taxonomy (category nodes with their computable rules) into the downloadable OBO file)
- **OPSIN** (Converts IUPAC names of query compounds into structures before rule matching)
- **ChemAxon JChem API 15.5.25.0** (Computes physico-chemical/structural properties (e.g. RingCount, AtomCount) used in feature-logic category rules)

## Examples

```
curl -X POST https://classyfire.wishartlab.com/entities -d '{"input":"CC(=O)Oc1ccccc1C(=O)O","input_format":"smiles"}' -H 'Content-Type: application/json'
```

## Evaluation signals

- Rule coverage: >9000 SMARTS strings generated; verify count and that no category lacks a rule, Markush pattern, feature-logic expression, or union-of-children definition
- Markush share: about 4% of structural patterns require the Markush fallback; a much larger fraction suggests the SMARTS derivation pipeline failed
- Standardization invariant: query compounds and category patterns undergo identical normalization and aromatization; check that a compound re-submitted in an equivalent resonance/tautomer form maps to the same category
- Category-reduction invariant: when parent–child categories both match (e.g. dioxanes and 1,2-dioxanes), only the child node is retained
- Consistency of direct parent: the selected direct parent must be the category defined by the largest matching structural feature

## Limitations

- Compounds that undergo reactions (e.g. loss of oxygen, substitutions) may no longer match the structural constraints of their expected category
- IUPAC name parsing rules (~200 regular expressions) can be insufficient for accurate class assignment in some cases; more parsing rules are needed
- Source code for ClassyFire was not yet fully released at the time of writing (promised by December 2016), limiting local rule inspection

## Evidence

- [abstract] SMARTS strings encode computable structural rules: "consists of up to 11 different levels (Kingdom, SuperClass, Class, SubClass, etc.) with each of the categories defined by unambiguous, computable structural rules"
- [methods] Markush fallback in Marvin: "For these exceptions we used the Markush format [36], which is available through ChemAxon’s Marvin tool."
- [methods] Standardization of patterns and queries: "each structure pattern was subjected to a set of standardization operations, including normalization and aromatization. Each query compound is subjected to the same operations"
- [methods] Rule-based category assignment weight dictionary: "A manually compiled dictionary, which provides the weight and category for each feature, was used for the rule-based"
- [methods] Category reduction keeps child only: "For every pair of chemical categories, if there is a parent–child relationship (e.g. dioxanes [parent] and 1,2-dioxanes [child]), only the child node is retained"
