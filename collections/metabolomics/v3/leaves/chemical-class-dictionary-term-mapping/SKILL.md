---
name: chemical-class-dictionary-term-mapping
description: Use when ClassyFire/ChemOnt category assignments must be translated into
  another chemical ontology (ChEBI classes, MeSH headings) for cross-database comparison
  or annotation exchange.
when_to_use_negative:
- The input structures have not yet been classified — run ClassyFire's rule-based
  structural classification first, since the lookup table maps category labels, not
  raw structures
- The target ontology is one the paper does not cover (mapping in the paper exists
  only for ChEBI and, partially and manually, for MeSH)
- A category is absent from the lookup table — dictionary-driven mapping cannot assign
  terms for categories it does not contain
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3316
tools:
- name: ClassyFire
  role: Produces the rule-based structural category assignments (ChemOnt categories)
    whose labels are then translated via the lookup table into ChEBI/MeSH terms
  repo: http://classyfire.wishartlab.com
- name: ClassyFire Ruby API
  role: Programmatic access to the ClassyFire server and database to retrieve category
    assignments and ChemOnt nodes for mapping
  repo: https://bitbucket.org/wishartlab/classyfire_api
- name: ChemOnt
  role: The computable chemical taxonomy whose categories (11 levels, >4800 categories)
    form the source side of the ontology lookup table
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-016-0174-y
    title: 'ClassyFire: automated chemical classification with a comprehensive, computable
      taxonomy'
  source_content_hash: sha256:1881ce520461b029437fed0ad1d50461f439952701b4b3b7ff6d2db93681d231
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemOnt
  - ClassyFire
  - ClassyFire Ruby API
  - OBO-Edit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# chemical-class-dictionary-term-mapping

## Summary

Maps chemical-classification category labels (e.g. ChemOnt/ClassyFire categories) onto terms in external ontologies such as ChEBI and MeSH using a manually compiled ontology lookup table (Chemical Classification Dictionary), so that classifications can be compared across vocabularies.

## When to use

Use when ClassyFire/ChemOnt category assignments must be translated into another chemical ontology (ChEBI classes, MeSH headings) for cross-database comparison or annotation exchange.

## When NOT to use

- The input structures have not yet been classified — run ClassyFire's rule-based structural classification first, since the lookup table maps category labels, not raw structures
- The target ontology is one the paper does not cover (mapping in the paper exists only for ChEBI and, partially and manually, for MeSH)
- A category is absent from the lookup table — dictionary-driven mapping cannot assign terms for categories it does not contain

## Inputs

- ClassyFire/ChemOnt category labels assigned to compounds (taxonomy node names, up to 11 levels from Kingdom to leaf)
- Ontology lookup table / Chemical Classification Dictionary mapping categories to target ontology terms
- ChEBI ontology (target classes and their hierarchy)
- MeSH thesaurus (target vocabulary for the partial manual mapping)

## Outputs

- Per-compound or per-category cross-references from ClassyFire categories to ChEBI classes
- Inferred ChEBI ancestor lists for each mapped category, enabling hierarchy-aware comparison
- Partial MeSH term correspondences for ClassyFire categories

## How to apply

Take the category labels returned by ClassyFire for each compound and look them up in the manually compiled ontology lookup table (the Chemical Classification Dictionary) to obtain the corresponding ChEBI class for each category. For comparison against ChEBI, run a script that infers the full ancestor list of each mapped ChEBI class so parent–child relationships in the target ontology are preserved during comparison. Treat the MeSH mapping as a separate, manual, partial crosswalk to the MeSH thesaurus used in PubChem rather than an exhaustive dictionary. Keep the lookup table as the single source of truth so the mapping is reproducible and dictionary-driven rather than inferred ad hoc.

## Related tools

- **ClassyFire** (Produces the rule-based structural category assignments (ChemOnt categories) whose labels are then translated via the lookup table into ChEBI/MeSH terms) — http://classyfire.wishartlab.com
- **ClassyFire Ruby API** (Programmatic access to the ClassyFire server and database to retrieve category assignments and ChemOnt nodes for mapping) — https://bitbucket.org/wishartlab/classyfire_api
- **ChemOnt** (The computable chemical taxonomy whose categories (11 levels, >4800 categories) form the source side of the ontology lookup table)

## Evaluation signals

- Every ClassyFire category in the comparison output resolves either to a ChEBI class or to an explicit unmapped entry — no silent drops
- The ancestor list inferred by the script for each mapped ChEBI class is consistent with the ChEBI ontology's own parent–child links
- Re-running the mapping with the same lookup table reproduces identical category-to-class correspondences (dictionary-driven determinism)
- MeSH crosswalk output is flagged/limited to the subset the paper reports as partially and manually mapped
- Mapped classes can be validated against known annotations, e.g. the >43,000 ChEBI small molecules ClassyFire was used to annotate

## Limitations

- Mapping to MeSH is manual and only partial, so MeSH correspondences cannot be assumed complete
- The lookup-table mapping depends on the manually compiled dictionary; categories outside the dictionary cannot be mapped automatically
- ClassyFire's own structural rules can misassign compounds that undergo reactions (e.g. loss of oxygen, substitutions), and such misassignments propagate into any downstream term mapping

## Evidence

- [results] Ontology lookup table maps categories to ChEBI classes: "The ontology lookup table described in the Methods section of this paper was used to map categories returned by ClassyFire to the ChEBI classes."
- [results] Script infers ancestor lists for ChEBI comparison: [evidence span withheld — anchor status: not_found]
- [methods] MeSH mapping is manual and partial: "ClassyFire has also been manually mapped, although only partially, to the MeSH thesaurus, which is used in the PubChem database."
