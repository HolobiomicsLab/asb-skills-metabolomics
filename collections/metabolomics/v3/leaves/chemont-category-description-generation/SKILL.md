---
name: chemont-category-description-generation
description: Use when a taxonomy of chemical classes needs human-readable English
  descriptions derived from computable structural definitions (SMARTS, Markush, physico-chemical
  constraints) rather than free-text curation.
when_to_use_negative:
- The goal is class assignment for a new compound structure — that requires the ClassyFire
  four-step classification pipeline (preprocessing, feature extraction, rule-based
  assignment, direct-parent selection), not description generation for taxonomy nodes.
- Structures need IUPAC-name-only feature detection via the ~200 regular-expression
  rules — that is a query-side parsing step, not a category-description task.
- Descriptions must cover reaction products (e.g. loss of oxygen, substitutions) that
  fall outside the category's structural constraints — a known limitation of category
  description matching.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3314
tools:
- name: ClassyFire
  role: Source of the ChemOnt taxonomy, Chemical Classification Dictionary, and RESTful
    web application used to expose the generated category descriptions
  repo: https://classyfire.wishartlab.com
- name: OBO-Edit
  role: Tool used to generate the ChemOnt OBO file from which the 4825 category nodes
    and their definitions are loaded
- name: ChemAxon Marvin
  role: Provides the Markush format used to encode structural patterns for exceptional
    categories during feature definition
- name: ChemAxon JChem API 15.5.25.0
  role: Computes the physico-chemical properties that feed into category-level structural
    constraints reflected in the descriptions
- name: Ruby
  role: Language of the open-source API that provides programmatic access to the ClassyFire
    server, database, and the serialized description data
  repo: https://bitbucket.org/wishartlab/classyfire_api
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-016-0174-y
    title: 'ClassyFire: automated chemical classification with a comprehensive, computable
      taxonomy'
  source_content_hash: sha256:2996df212a6d0a912ee73f78204a6c14ccabbcf85b2aecf47cd52321b2b16ea6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemAxon JChem API 15.5.25.0
  - ChemAxon Marvin
  - ClassyFire
  - OBO-Edit
  - Ruby
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# chemont-category-description-generation

## Summary

Reconstructs the ClassyFire/ChemOnt textual description generator: for each of the 4825 ChemOnt category nodes, compose a 20–50 word English sentence describing the characteristic structural features (rings, heteroatoms, functional groups, formula rules) and attach it to the category's CHEMONTID identifier for distribution via the Chemical Classification Dictionary.

## When to use

Use when a taxonomy of chemical classes needs human-readable English descriptions derived from computable structural definitions (SMARTS, Markush, physico-chemical constraints) rather than free-text curation.

## When NOT to use

- The goal is class assignment for a new compound structure — that requires the ClassyFire four-step classification pipeline (preprocessing, feature extraction, rule-based assignment, direct-parent selection), not description generation for taxonomy nodes.
- Structures need IUPAC-name-only feature detection via the ~200 regular-expression rules — that is a query-side parsing step, not a category-description task.
- Descriptions must cover reaction products (e.g. loss of oxygen, substitutions) that fall outside the category's structural constraints — a known limitation of category description matching.

## Inputs

- ChemOnt OBO ontology file (4825 categories, up to 11 levels, generated with OBO-Edit)
- Chemical Classification Dictionary mapping features to weighted categories
- Per-category structural feature definitions (SMARTS patterns, Markush structures, physico-chemical constraints, formula rules)
- ChemOnt category identifiers (CHEMONTID:...)

## Outputs

- English textual description (20–50 words) per ChemOnt category node
- JSON file mapping each CHEMONTID category identifier to its textual description
- Description set distributed via the ClassyFire RESTful API / web server for annotating compounds (e.g. >100,000 compounds downstream)

## How to apply

Load the ChemOnt category hierarchy (4825 categories, up to 11 levels: Kingdom, SuperClass, Class, SubClass, etc.) from the ClassyFire/ChemOnt OBO file, which was generated with OBO-Edit. For each category node, extract the defining structural elements — SMARTS patterns, Markush structures (built with ChemAxon Marvin for exceptions), physico-chemical properties computed via ChemAxon's JChem API 15.5.25.0, and chemical formula rules — and identify the ring counts, heteroatom composition, functional groups, and dominant substructures shared by member compounds. Compose one English sentence per category that names the shared structural pattern plus distinguishing chemical context, e.g. 'a heterocyclic compound containing a five-member aromatic ring made up of one sulfur atom, one nitrogen, and three carbon atoms'. Attach each description to its CHEMONTID identifier and check consistency against the formal computable definitions in the Chemical Classification Dictionary, then serialize the identifier-to-description mapping as JSON for download via the ClassyFire RESTful API or web server. The rationale is that the same structural features ClassyFire extracts for rule-based category assignment can be verbalized, which is how ClassyFire generates textual descriptions for over 100,000 compounds downstream.

## Related tools

- **ClassyFire** (Source of the ChemOnt taxonomy, Chemical Classification Dictionary, and RESTful web application used to expose the generated category descriptions) — https://classyfire.wishartlab.com
- **OBO-Edit** (Tool used to generate the ChemOnt OBO file from which the 4825 category nodes and their definitions are loaded)
- **ChemAxon Marvin** (Provides the Markush format used to encode structural patterns for exceptional categories during feature definition)
- **ChemAxon JChem API 15.5.25.0** (Computes the physico-chemical properties that feed into category-level structural constraints reflected in the descriptions)
- **Ruby** (Language of the open-source API that provides programmatic access to the ClassyFire server, database, and the serialized description data) — https://bitbucket.org/wishartlab/classyfire_api

## Examples

```
curl http://classyfire.wishartlab.com/entities/CHEMONTID:0000223.json
```

## Evaluation signals

- Each of the 4825 ChemOnt categories has a non-empty, well-formed English description (target length 20–50 words).
- Each description's content matches the structural features defined in the corresponding Chemical Classification Dictionary entry.
- Every description is attached to a valid CHEMONTID identifier consistent with the ChemOnt OBO hierarchy.
- The serialized identifier-to-description mapping parses as valid JSON.
- Descriptions served through the ClassyFire RESTful API match the locally serialized JSON mapping.

## Limitations

- Compounds from a given category may undergo reactions (e.g. loss of oxygen, substitutions) that no longer match the constraints described in the category description.
- Source code for ClassyFire was not yet released at the time of writing; open-source availability was promised by December 2016, so the description generator may need to be reimplemented from the described rules.
- Descriptions are purely structure-based; they do not encode biological or therapeutic semantics.

## Evidence

- [abstract] ChemOnt dictionary structure: "automatically generate textual descriptions for, and/or infer biological properties of over 100,000 compounds"
- [abstract] Taxonomy scope: "This new chemical taxonomy consists of up to 11 different levels (Kingdom, SuperClass, Class, SubClass, etc.) with each of the categories defined by unambiguous, computable structural rules"
- [methods] OBO file generation: "The resulting OBO file was generated with OBO-Edit [34], and can be downloaded from the ClassyFire website."
- [methods] Markush format: "For these exceptions we used the Markush format [36], which is available through ChemAxon’s Marvin tool."
- [methods] Physico-chemical properties: "The calculation of physico-chemical properties is performed using ChemAxon’s JChem API (version 15.5.25.0)."
- [methods] RESTful access: "The entire ClassyFire program has been converted to a web-based resource. It is a RESTful web application located at http://classyfire.wishartlab.com."
