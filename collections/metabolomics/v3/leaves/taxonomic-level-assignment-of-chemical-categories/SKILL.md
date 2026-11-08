---
name: taxonomic-level-assignment-of-chemical-categories
description: Use when curating or rebuilding a chemical classification ontology from
  OBO-format term stanzas, and each term must be assigned to a taxonomic level with
  preferred names and synonyms resolved.
when_to_use_negative:
- When the goal is to classify individual compound structures into categories — use
  ClassyFire's rule-based structural classification instead, not dictionary curation.
- When the taxonomy is already level-annotated and only structural feature extraction
  or category reduction is needed for a query compound.
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: OBO-Edit
  role: Used to generate and edit the OBO-format file carrying ChemOnt term IDs, definitions,
    and synonym stanzas that the level assignment is performed on
- name: ClassyFire
  role: Hosts the ChemOnt taxonomy and Chemical Classification Dictionary; the OBO
    file is downloadable from the ClassyFire website
  repo: https://classyfire.wishartlab.com
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-016-0174-y
    title: 'ClassyFire: automated chemical classification with a comprehensive, computable
      taxonomy'
  source_content_hash: sha256:0f38340e73bf1294cb644ca6a286afe1364a4589c8e5c26ec1b96d0d5d05f12e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClassyFire
  - OBO-Edit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Taxonomic-level assignment of chemical categories

## Summary

Assigns each category in the ChemOnt chemical taxonomy to its correct taxonomic level (Kingdom, SuperClass, Class, SubClass, etc.) and records competing term choices as synonyms, producing the computable Chemical Classification Dictionary underlying ClassyFire.

## When to use

Use when curating or rebuilding a chemical classification ontology from OBO-format term stanzas, and each term must be assigned to a taxonomic level with preferred names and synonyms resolved.

## When NOT to use

- When the goal is to classify individual compound structures into categories — use ClassyFire's rule-based structural classification instead, not dictionary curation.
- When the taxonomy is already level-annotated and only structural feature extraction or category reduction is needed for a query compound.

## Inputs

- ChemOnt taxonomy in OBO format (term IDs, preferred names, definitions, synonym stanzas)
- Usage-frequency evidence for competing term names (e.g. Google hit counts)
- Cross-ontology mapping tables to ChEBI, LIPID MAPS, and MeSH

## Outputs

- Chemical Classification Dictionary covering all 4825 ChemOnt categories with level assignments
- Synonym table (~9012 English synonym entries) with exact/narrow/broad/related scope tags
- 20–50 word textual definitions per category usable for automated compound description generation

## How to apply

Download the ChemOnt taxonomy in OBO format from the ClassyFire website (the OBO file was generated with OBO-Edit) and parse the term IDs, preferred names, text definitions, and synonym stanzas for all 4825 categories (4146 organic, 678 inorganic, plus the root 'Chemical entities'). Assign levels using the paper's convention: generic terms covering large numbers of compounds (e.g. broad functional-group classes) belong at the SuperClass level, while highly specific structural terms belong at Class or SubClass level within the up-to-11-level hierarchy (Kingdom, SuperClass, Class, SubClass, etc.). When two competing terms describe the same category, select by recorded usage frequency (e.g. 'Imidazolines' with 229,000 Google hits over 'Dihydroimidazoles' with 4,590), give precedence to IUPAC terminology and expert-panel recommendations, and demote the losing term to a synonym. Retain cross-ontology synonym mappings to ChEBI, LIPID MAPS, and MeSH with their exact/narrow/broad/related scope tags (e.g. ChEBI 'carboxylic acid' as a narrow synonym of ChemOnt 'carboxylic acids'). The 20–50 word text definitions attached to each category then serve as a dictionary from which English descriptions of individual compounds can be automatically generated.

## Related tools

- **OBO-Edit** (Used to generate and edit the OBO-format file carrying ChemOnt term IDs, definitions, and synonym stanzas that the level assignment is performed on)
- **ClassyFire** (Hosts the ChemOnt taxonomy and Chemical Classification Dictionary; the OBO file is downloadable from the ClassyFire website) — https://classyfire.wishartlab.com

## Evaluation signals

- Synonym table covers all 4825 categories (4146 organic, 678 inorganic, plus the root 'Chemical entities') with roughly 9012 English synonym entries.
- Level assignments follow the convention that generic high-coverage terms are SuperClass-level and highly specific terms are Class/SubClass-level.
- Spot-check documented term resolutions, e.g. 'Imidazolines' preferred over 'Dihydroimidazoles' (229,000 vs 4,590 Google hits), with the losing term retained as a synonym.
- ChEBI/LIPID MAPS/MeSH cross-mappings preserve their scope tags (e.g. ChEBI 'carboxylic acid' as narrow synonym of ChemOnt 'carboxylic acids').
- Each category carries a 20–50 word definition suitable for automatically generating English descriptions of compounds.

## Limitations

- Term-name selection by usage frequency is heuristic; IUPAC terms and expert-panel recommendations must override raw hit counts.
- Compounds undergoing reactions (e.g. loss of oxygen, substitutions) may not match the constraints described in a category's textual definition.
- The dictionary describes categories, not individual structures — assigning a query compound requires ClassyFire's separate rule-based classification pipeline.

## Evidence

- [abstract] Taxonomy levels span up to 11 levels with computable rules: "This new chemical taxonomy consists of up to 11 different levels (Kingdom, SuperClass, Class, SubClass, etc.) with each of the categories defined by unambiguous, computable structural rules"
- [intro] ChemOnt and Classification Dictionary deliverables: "we describe a comprehensive, flexible, computable, chemical taxonomy along with a fully annotated chemical ontology (ChemOnt) and a Chemical Classification Dictionary"
- [methods] OBO file generated with OBO-Edit and downloadable: "The resulting OBO file was generated with OBO-Edit [34], and can be downloaded from the ClassyFire website."
- [abstract] Taxonomy size of over 4800 categories: "automatically assign all known chemical compounds to a taxonomy consisting of >4800 different categories"
