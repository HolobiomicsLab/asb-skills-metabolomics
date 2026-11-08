---
name: obo-ontology-authoring
description: Use when a computed or curated chemical taxonomy must be published as
  a formal ontology file with parent-child 'is_a' edges, English text definitions,
  synonyms, and a JSON mirror.
when_to_use_negative:
- The taxonomy already exists in a downstream tool's internal format and only needs
  querying rather than authoring.
- Input is a flat list of chemical classes without parent-child relationships, structural
  rules, or definitions—there is nothing to encode as an ontology.
- The target is a non-hierarchical controlled vocabulary with no 'is_a' links or definable
  structural rules.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3345
tools:
- name: OBO-Edit
  role: Generates the .obo ontology file from the enumerated category hierarchy, structural
    rules, definitions, and synonyms
- name: ChemAxon Marvin
  role: Used to author Markush-format structural rules for categories that cannot
    be expressed as plain SMARTS
- name: ClassyFire
  role: Consumes the resulting ChemOnt taxonomy to assign compounds to categories
    during classification
  repo: https://classyfire.wishartlab.com
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-016-0174-y
    title: 'ClassyFire: automated chemical classification with a comprehensive, computable
      taxonomy'
  source_content_hash: sha256:7970cab4609b35ebddfccae8f843340da0b65ac51cad65cfac93d556452725ed
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemAxon Marvin
  - ClassyFire
  - OBO-Edit
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3345
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# obo-ontology-authoring

## Summary

Encodes a chemical taxonomy as a formal OBO-format ontology (e.g. ChemOnt's 4825 hierarchical chemical categories) using OBO-Edit, mirroring it as a machine-readable JSON tree for programmatic access.

## When to use

Use when a computed or curated chemical taxonomy must be published as a formal ontology file with parent-child 'is_a' edges, English text definitions, synonyms, and a JSON mirror.

## When NOT to use

- The taxonomy already exists in a downstream tool's internal format and only needs querying rather than authoring.
- Input is a flat list of chemical classes without parent-child relationships, structural rules, or definitions—there is nothing to encode as an ontology.
- The target is a non-hierarchical controlled vocabulary with no 'is_a' links or definable structural rules.

## Inputs

- Hierarchical category list with parent-child links (e.g. 2 Kingdoms, 31 SuperClasses, 764 Classes, 1729 SubClasses, 2296 lower-level categories)
- Computable structural rules per category (SMARTS, Markush patterns, logical expressions over RingCount/AtomCount, union definitions)
- English text definitions and synonyms per category

## Outputs

- .obo ontology file generated with OBO-Edit
- JSON mirror of the taxonomy tree for programmatic access
- Validated hierarchy with 'is_a' parent edges, definitions, and synonyms on every node

## How to apply

Enumerate the taxonomic levels (Kingdom, SuperClass, Class, SubClass, and levels 5–11) and link each category to its parent via an 'is_a' relationship. For each node, attach a computable structural rule—a SMARTS string (the >9000 generated for ChemOnt), a Markush pattern (used for ~4% of patterns, authored with ChemAxon Marvin), a logical expression over features like RingCount and AtomCount, or a union-of-subcategories definition. Record the English text definition and synonyms, preferring the IUPAC or expert-selected term as the primary label. Generate the ontology file with OBO-Edit, and emit a JSON mirror of the same tree so programmatic agents can traverse it. Validate that the tree's maximum depth matches the design (11 for ChemOnt), every node has a parent edge and a definition, and the node count matches the enumerated categories.

## Related tools

- **OBO-Edit** (Generates the .obo ontology file from the enumerated category hierarchy, structural rules, definitions, and synonyms)
- **ChemAxon Marvin** (Used to author Markush-format structural rules for categories that cannot be expressed as plain SMARTS)
- **ClassyFire** (Consumes the resulting ChemOnt taxonomy to assign compounds to categories during classification) — https://classyfire.wishartlab.com

## Evaluation signals

- Total node count equals the enumerated categories (4825 for ChemOnt) plus the root 'Chemical entities' node
- Maximum tree depth equals 11 levels with an average depth of five levels per node
- Every node has a parent 'is_a' edge and a non-empty definition field
- Spot-check representative categories (e.g. thiazoles SMARTS, the alkane rule RingCount(A)=0 AND AtomCount(C,A)>0 AND AtomCount(H,A)=2*AtomCount(C,A)+2) match the intended structural rules
- The JSON mirror traverses identically to the .obo file and every category is reachable from the root

## Limitations

- Compounds undergoing reactions (e.g. loss of oxygen, substitutions) may not match the constraints described in a category description
- IUPAC name parsing rules may be insufficient for accurate class assignment in some cases, so categories relying solely on name-derived features may be incomplete
- Markush patterns were needed for about 4% of structural rules because SMARTS alone could not express them

## Evidence

- [abstract] Kingdom, SuperClass, Class, SubClass, etc.) with each of the categories defined by unambiguous, computable structural rules: "This new chemical taxonomy consists of up to 11 different levels (Kingdom, SuperClass, Class, SubClass, etc.) with each of the categories defined by unambiguous, computable structural rules"
- [methods] The resulting OBO file was generated with OBO-Edit: "The resulting OBO file was generated with OBO-Edit [34], and can be downloaded from the ClassyFire website."
- [methods] For these exceptions we used the Markush format [36], which is available through ChemAxon’s Marvin tool: "For these exceptions we used the Markush format [36], which is available through ChemAxon’s Marvin tool."
