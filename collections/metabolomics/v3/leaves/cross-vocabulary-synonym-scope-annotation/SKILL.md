---
name: cross-vocabulary-synonym-scope-annotation
description: Use when compiling a classification dictionary or synonym table for a
  chemical taxonomy (e.g. ChemOnt OBO terms) that must carry scoped mappings to external
  vocabularies like ChEBI, LIPID MAPS, or MeSH.
when_to_use_negative:
- The goal is structural classification of compounds rather than building the taxonomy/synonym
  dictionary itself — use ClassyFire's rule-based category assignment instead.
- The source ontology already provides fully scoped cross-references, so re-deriving
  synonym scopes would duplicate existing annotations without ground truth for frequency-based
  name arbitration.
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3332
tools:
- name: OBO-Edit
  role: Used to generate and edit the OBO-format ChemOnt file whose term and synonym
    stanzas are parsed to build the synonym table
- name: ClassyFire
  role: Provides the ChemOnt taxonomy and web server from which the OBO file underlying
    the Chemical Classification Dictionary is obtained
  repo: https://classyfire.wishartlab.com
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-016-0174-y
    title: 'ClassyFire: automated chemical classification with a comprehensive, computable
      taxonomy'
  source_content_hash: sha256:b7dd449345648455aa2fda24d0cb0e326f126dad1e634cd7953c42ea1d484b09
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClassyFire
  - OBO-Edit
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# Cross-vocabulary synonym scope annotation

## Summary

Building a synonym table that maps each ChemOnt category (4825 chemical classes) to alternate names and to cross-ontology terms in ChEBI, LIPID MAPS, and MeSH, with explicit exact/narrow/broad/related scope tags. The resulting ~9012-entry synonym table underpins the Chemical Classification Dictionary used to generate textual descriptions of compounds.

## When to use

Use when compiling a classification dictionary or synonym table for a chemical taxonomy (e.g. ChemOnt OBO terms) that must carry scoped mappings to external vocabularies like ChEBI, LIPID MAPS, or MeSH.

## When NOT to use

- The goal is structural classification of compounds rather than building the taxonomy/synonym dictionary itself — use ClassyFire's rule-based category assignment instead.
- The source ontology already provides fully scoped cross-references, so re-deriving synonym scopes would duplicate existing annotations without ground truth for frequency-based name arbitration.

## Inputs

- ChemOnt chemical taxonomy in OBO format (downloaded from the ClassyFire website)
- Term stanzas with IDs, preferred names, text definitions, and synonym lists
- External vocabulary terms from ChEBI, LIPID MAPS, and MeSH for cross-mapping
- Usage-frequency counts (e.g. Google hit counts) for competing term names

## Outputs

- Chemical Classification Dictionary synonym table covering all 4825 ChemOnt categories
- ~9012 English synonym entries mapping each ChemOnt term to alternate names
- Cross-vocabulary mapping records with exact/narrow/broad/related scope tags (e.g. to ChEBI, LIPID MAPS, MeSH)

## How to apply

Parse the ChemOnt OBO file (generated with OBO-Edit and downloadable from the ClassyFire website) to extract term IDs, preferred names, definitions, and synonym stanzas for all 4825 categories (4146 organic, 678 inorganic, plus the root 'Chemical entities'). For each term, resolve competing name choices by recorded usage frequency (e.g. 'Imidazolines' with 229,000 Google hits over 'Dihydroimidazoles' with 4590), giving precedence to IUPAC terms and expert-panel recommendations; retain losing terms as synonyms. When mapping to external ontologies, record the scope of each mapping explicitly — for example, ChEBI 'carboxylic acid' is a narrow synonym of ChemOnt 'carboxylic acids' — using exact/narrow/broad/related tags. Validate that the table covers all 4825 categories with roughly 9012 English synonym entries and spot-check documented pairs such as Imidazolines/Dihydroimidazoles.

## Related tools

- **OBO-Edit** (Used to generate and edit the OBO-format ChemOnt file whose term and synonym stanzas are parsed to build the synonym table)
- **ClassyFire** (Provides the ChemOnt taxonomy and web server from which the OBO file underlying the Chemical Classification Dictionary is obtained) — https://classyfire.wishartlab.com

## Evaluation signals

- Table covers all 4825 ChemOnt categories (4146 organic, 678 inorganic, plus root 'Chemical entities') with no missing terms.
- Total synonym count is approximately 9012 English entries.
- Every cross-vocabulary mapping carries one of the exact/narrow/broad/related scope tags (e.g. ChEBI 'carboxylic acid' = narrow synonym of ChemOnt 'carboxylic acids').
- Documented arbitration pairs resolve correctly, e.g. 'Imidazolines' preferred over 'Dihydroimidazoles' (229,000 vs 4590 Google hits) with the loser retained as a synonym.
- Preferred names follow IUPAC terms and expert-panel recommendations where applicable.

## Limitations

- Mappings to MeSH are only partial: 'ClassyFire has also been manually mapped, although only partially, to the MeSH thesaurus, which is used in the PubChem database.'
- Term-name arbitration by Google hit count is a heuristic; IUPAC terms and expert-panel recommendations must take precedence to avoid popular-but-nonstandard names.
- The OBO file was generated with OBO-Edit, so synonym stanzas must be re-parsed if the taxonomy is regenerated or updated.

## Evidence

- [methods] Synonym stanzas for 4825 categories: "we describe a comprehensive, flexible, computable, chemical taxonomy along with a fully annotated chemical ontology (ChemOnt) and a Chemical Classification Dictionary"
- [abstract] Synonym scope tags: "automatically assign all known chemical compounds to a taxonomy consisting of >4800 different categories"
- [methods] OBO file provenance: "The resulting OBO file was generated with OBO-Edit [34], and can be downloaded from the ClassyFire website."
- [methods] Partial MeSH mapping: "ClassyFire has also been manually mapped, although only partially, to the MeSH thesaurus, which is used in the PubChem database."
