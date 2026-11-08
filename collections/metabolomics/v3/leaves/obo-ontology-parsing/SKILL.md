---
name: obo-ontology-parsing
description: Use when a chemical taxonomy or ontology is distributed as an OBO file
  (e.g. ChemOnt from the ClassyFire website) and you need term IDs, names, text definitions,
  synonyms, and parent–child hierarchy extracted into a machine-readable dictionary.
when_to_use_negative:
- The goal is to classify individual compound structures rather than parse the taxonomy
  itself — the ClassyFire classification workflow (feature extraction, rule-based
  category assignment) is the appropriate skill, not OBO parsing
- The ontology is not in OBO format (e.g. OWL/RDF serializations), where the OBO stanza
  parser described here would not apply directly
- You need the computable structural rules (SMARTS patterns, Markush structures, formula
  constraints) rather than the textual names, definitions, and synonyms stored in
  the OBO file
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_2258
tools:
- name: OBO-Edit
  role: Used to generate and edit the ChemOnt OBO file that this skill parses
- name: ClassyFire
  role: Source of the ChemOnt taxonomy OBO file downloadable from the ClassyFire website
  repo: https://classyfire.wishartlab.com
- name: Ruby
  role: Ruby API providing programmatic access to the ClassyFire server and database
    for retrieving the classification dictionary
  repo: https://bitbucket.org/wishartlab/classyfire_api
provenance:
  source_task_ids:
  - task_005
  - task_007
  - task_006
  source_papers:
  - doi: 10.1186/s13321-016-0174-y
    title: 'ClassyFire: automated chemical classification with a comprehensive, computable
      taxonomy'
  source_content_hash: sha256:f1316c0d16a3bdb775bcc5ab67fa4a6d05e580fe195703ebc21189927b2fb0dd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClassyFire
  - OBO-Edit
  - Ruby
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# obo-ontology-parsing

## Summary

Parses the ChemOnt chemical taxonomy delivered as an OBO file (generated with OBO-Edit) into structured category records — term IDs, preferred names, text definitions, synonym stanzas, and parent links — forming the Chemical Classification Dictionary that underlies ClassyFire's 4825-category hierarchy.

## When to use

Use when a chemical taxonomy or ontology is distributed as an OBO file (e.g. ChemOnt from the ClassyFire website) and you need term IDs, names, text definitions, synonyms, and parent–child hierarchy extracted into a machine-readable dictionary.

## When NOT to use

- The goal is to classify individual compound structures rather than parse the taxonomy itself — the ClassyFire classification workflow (feature extraction, rule-based category assignment) is the appropriate skill, not OBO parsing
- The ontology is not in OBO format (e.g. OWL/RDF serializations), where the OBO stanza parser described here would not apply directly
- You need the computable structural rules (SMARTS patterns, Markush structures, formula constraints) rather than the textual names, definitions, and synonyms stored in the OBO file

## Inputs

- ChemOnt chemical taxonomy in OBO format (downloaded from the ClassyFire website, generated with OBO-Edit)
- Cross-reference sources for synonym mapping: ChEBI, LIPID MAPS, MeSH

## Outputs

- JSON mapping of ChemOnt category identifiers (CHEMONTID:…) to preferred names, text definitions, and parent links
- Synonym table (~9012 English synonym entries) with exact/narrow/broad/related scope tags for ChEBI, LIPID MAPS, and MeSH mappings

## How to apply

Download the ChemOnt taxonomy in OBO format from the ClassyFire website (the file was generated with OBO-Edit) and parse the term stanzas, extracting term IDs (CHEMONTID:…), preferred names, text definitions, and synonym stanzas. The tree is rooted at 'Chemical entities' and spans 4825 categories (4146 organic, 678 inorganic) across up to 11 levels (Kingdom, SuperClass, Class, SubClass, etc.); validate level assignment using the paper's conventions — generic terms belong to SuperClass level, highly specific terms to Class/SubClass level. Build a synonym table mapping each term to alternate names, carrying exact/narrow/broad/related scope tags for cross-references to ChEBI, LIPID MAPS, and MeSH, resolving competing term choices by usage frequency while giving precedence to IUPAC terms (e.g. 'Imidazolines' with 229,000 Google hits over 'Dihydroimidazoles' with 4590; losing terms retained as synonyms). Serialize the category-identifier-to-name/definition/synonym mapping as JSON, since the ClassyFire RESTful API and web server deliver the dictionary in this form.

## Related tools

- **OBO-Edit** (Used to generate and edit the ChemOnt OBO file that this skill parses)
- **ClassyFire** (Source of the ChemOnt taxonomy OBO file downloadable from the ClassyFire website) — https://classyfire.wishartlab.com
- **Ruby** (Ruby API providing programmatic access to the ClassyFire server and database for retrieving the classification dictionary) — https://bitbucket.org/wishartlab/classyfire_api

## Evaluation signals

- The parsed term set covers all 4825 chemical categories (4146 organic, 678 inorganic, plus the root 'Chemical entities')
- The synonym table contains roughly 9012 English synonym entries and reproduces documented examples such as the Imidazolines/Dihydroimidazoles pair
- Every category record carries a valid parent link and a taxonomic level consistent with the tree depth of at most 11 levels
- The serialized output parses as valid JSON and each category has a non-empty, well-formed text definition
- Scope tags (exact/narrow/broad/related) are present on cross-references, e.g. ChEBI 'carboxylic acid' as a narrow synonym of ChemOnt 'carboxylic acids'

## Limitations

- Existing chemical ontologies (the reason ChemOnt was built) provide only limited coverage of inorganic and organometallic compounds, so cross-ontology synonym mappings may be sparse for these branches
- Manual mappings to external thesauri are incomplete — ClassyFire has only been partially mapped to the MeSH thesaurus used in PubChem
- The OBO file captures names, definitions, and synonyms; the computable structural rules live in the ClassyFire program and must be reconciled separately against the dictionary

## Evidence

- [methods] OBO file generated with OBO-Edit: "The resulting OBO file was generated with OBO-Edit [34], and can be downloaded from the ClassyFire website."
- [abstract] Taxonomy levels and computable rules: "This new chemical taxonomy consists of up to 11 different levels (Kingdom, SuperClass, Class, SubClass, etc.) with each of the categories defined by unambiguous, computable structural rules"
- [results] ChEBI category mapping via lookup table: "The ontology lookup table described in the Methods section of this paper was used to map categories returned by ClassyFire to the ChEBI classes."
- [methods] Partial MeSH mapping: "ClassyFire has also been manually mapped, although only partially, to the MeSH thesaurus, which is used in the PubChem database."
