---
name: smarts-pattern-interpretation
description: Use when assigning chemical compounds to taxonomic categories based on
  computed structural features, or when building/interpreting computable chemical
  ontologies (e.g. ChemOnt) where category definitions rely on SMARTS-pattern substructure
  matching.
when_to_use_negative:
- When the input is a textual/keyword-based chemical description (e.g. a plain English
  paragraph) rather than an explicit molecular structure — ClassyFire's structural
  matching cannot operate on it.
- When classifying compounds that undergo reactions (e.g. loss of oxygen, substitutions)
  that fall outside the constraints of the current category definitions, as the article
  notes these 'might not match the constraints described in a category description'.
- When only a partial IUPAC name is available and parsing rules cannot resolve it
  to a full structure — the article notes that 'creating more rules for IUPAC name
  parsing could help to assign some classes more accurately' but currently the rules
  are insufficient in some cases.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3314
tools:
- name: ClassyFire
  role: The web-based application that executes the four-step classification pipeline
    (preprocessing, feature extraction, rule-based category assignment/reduction,
    direct parent selection) using the SMARTS-pattern feature definitions of the ChemOnt
    taxonomy.
  repo: https://classyfire.wishartlab.com
- name: OPSIN
  role: Converts IUPAC names to structures so that SMARTS-based substructure matching
    can be performed when only a name is provided as input.
- name: ChemAxon JChem API 15.5.25.0
  role: Calculates physico-chemical properties that supplement the SMARTS/Markush
    structural features during feature extraction.
- name: Checkmol
  role: Automatically assigns functional groups (e.g. CO functional groups) to a given
    structure, complementing the SMARTS-pattern extraction.
- name: OBO-Edit
  role: Used to generate the OBO file that stores the ChemOnt hierarchy and the computable
    structural rules (SMARTS patterns, Markush structures) for each category.
- name: ChemAxon Marvin
  role: Used to generate Markush format definitions for categories where plain SMARTS
    patterns cannot fully describe the structural variation.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-016-0174-y
    title: 'ClassyFire: automated chemical classification with a comprehensive, computable
      taxonomy'
  source_content_hash: sha256:1bd1c88304e5760a4730b28427922e16a08cf5c16960bb53ca569f5ec5cc6924
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Checkmol
  - ChemAxon JChem API 15.5.25.0
  - ChemAxon Marvin
  - ClassyFire
  - OBO-Edit
  - OPSIN
  - Ruby
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# smarts-pattern-interpretation

## Summary

Interprets SMARTS (SMILES ARbitrary Target Specification) patterns and other structural descriptors as the defining features that drive automated rule-based chemical classification, as implemented in ClassyFire for the ChemOnt taxonomy.

## When to use

Use when assigning chemical compounds to taxonomic categories based on computed structural features, or when building/interpreting computable chemical ontologies (e.g. ChemOnt) where category definitions rely on SMARTS-pattern substructure matching.

## When NOT to use

- When the input is a textual/keyword-based chemical description (e.g. a plain English paragraph) rather than an explicit molecular structure — ClassyFire's structural matching cannot operate on it.
- When classifying compounds that undergo reactions (e.g. loss of oxygen, substitutions) that fall outside the constraints of the current category definitions, as the article notes these 'might not match the constraints described in a category description'.
- When only a partial IUPAC name is available and parsing rules cannot resolve it to a full structure — the article notes that 'creating more rules for IUPAC name parsing could help to assign some classes more accurately' but currently the rules are insufficient in some cases.

## Inputs

- Canonical chemical structures (SMILES, InChI, or molecular graphs) for query compounds
- IUPAC chemical names (to be converted to structures via OPSIN before SMARTS matching)
- ChemOnt OBO file containing the category hierarchy and their computable structural rules (SMARTS patterns, Markush structures, physico-chemical constraints, formula rules)
- Manually compiled feature-to-category dictionary (feature → weight, category)

## Outputs

- Taxonomic assignment of each compound to one or more ChemOnt categories across up to 11 levels (Kingdom, SuperClass, Class, SubClass, ...)
- Direct parent category (the category defined by the largest structural feature describing the compound)
- Structured classification results (e.g. JSON/JSON-LD) listing category identifiers (CHEMONTID:...) and their English textual descriptions

## How to apply

Represent the query compound as a canonical structure (InChI/SMILES; convert IUPAC names via OPSIN) and standardize it through normalization and aromatization so that SMARTS-pattern matching is consistent with the reference features. Extract structural features by performing superstructure-search operations against the SMARTS patterns, Markush structures, and structural rules that define each ChemOnt category node, complementing these with physico-chemical property calculations (e.g. via ChemAxon JChem API) and functional-group detection (e.g. via Checkmol). Each detected feature is mapped to a category through a manually compiled feature-to-category dictionary that encodes weight and category, after which a rule-based assignment step reduces the candidate categories by removing parent nodes when a child node (a more specific substructure match) is present. The rationale is that ClassyFire's categories are defined by 'unambiguous, computable structural rules' rather than free-text labels, so the interpretation of SMARTS patterns directly determines which taxonomic branch (Kingdom, SuperClass, Class, SubClass, up to 11 levels) the compound is assigned to.

## Related tools

- **ClassyFire** (The web-based application that executes the four-step classification pipeline (preprocessing, feature extraction, rule-based category assignment/reduction, direct parent selection) using the SMARTS-pattern feature definitions of the ChemOnt taxonomy.) — https://classyfire.wishartlab.com
- **OPSIN** (Converts IUPAC names to structures so that SMARTS-based substructure matching can be performed when only a name is provided as input.)
- **ChemAxon JChem API 15.5.25.0** (Calculates physico-chemical properties that supplement the SMARTS/Markush structural features during feature extraction.)
- **Checkmol** (Automatically assigns functional groups (e.g. CO functional groups) to a given structure, complementing the SMARTS-pattern extraction.)
- **OBO-Edit** (Used to generate the OBO file that stores the ChemOnt hierarchy and the computable structural rules (SMARTS patterns, Markush structures) for each category.)
- **ChemAxon Marvin** (Used to generate Markush format definitions for categories where plain SMARTS patterns cannot fully describe the structural variation.)

## Evaluation signals

- Each of the 4825 ChemOnt categories must be associated with a non-empty, well-formed SMARTS/Markush/property-based rule and a corresponding English textual description.
- For every pair of categories, if a parent–child relationship exists (e.g. dioxanes → 1,2-dioxanes), only the child node is retained after category reduction — the parent node must not appear in the final non-redundant list.
- Query compounds must be subjected to the same standardization operations (normalization, aromatization) as the SMARTS pattern definitions, so that a compound known to belong to a category (e.g. a validated reference compound) is correctly matched by that category's SMARTS pattern.
- The direct parent selected must correspond to the largest structural feature that describes the compound, i.e. the most specific match in the taxonomy, and must be consistent with the feature-to-category dictionary weights.
- Output must be valid JSON (or JSON-LD) with non-empty category identifiers (CHEMONTID:...) and consistent English descriptions matching the formal computable definitions stored in the Chemical Classification Dictionary.

## Limitations

- Compounds undergoing reactions (e.g. loss of oxygen, substitutions) may not match the constraints of their nominal category description.
- IUPAC name parsing rules (~200 regular expressions) are sometimes insufficient to accurately resolve structural features for class assignment.
- A freely available version of ClassyFire with full source code was still under development at the time of writing (promised by December 2016), limiting reproducibility.

## Evidence

- [methods] SMARTS patterns define ChemOnt categories: "with each of the categories defined by unambiguous, computable structural rules"
- [methods] Feature extraction uses superstructure search: "The second step in the ClassyFire program involves the generation of structural features based on a combination of superstructure-search operations and various property calculations."
- [methods] Rule-based assignment uses a manually compiled dictionary: "each feature is then mapped to its corresponding category or node in the taxonomy. A manually compiled dictionary, which provides the weight and category for each feature, was used for the rule-based"
- [methods] Category reduction retains child over parent: "For every pair of chemical categories, if there is a parent–child relationship (e.g. dioxanes [parent] and 1,2-dioxanes [child]), only the child node is retained"
- [methods] Direct parent is the largest structural feature: "The direct parent is the category defined by the largest structural feature that describes the compound. It is selected from the non-redundant list of categories obtained in the previous step."
- [methods] Structure standardization ensures consistent matching: "each structure pattern was subjected to a set of standardization operations, including normalization and aromatization. Each query compound is subjected to the same operations"
- [abstract] Taxonomy has up to 11 levels: "This new chemical taxonomy consists of up to 11 different levels (Kingdom, SuperClass, Class, SubClass, etc.)"
- [methods] IUPAC parsing uses regular expressions: "We developed a module, which uses a set of ~200 regular expressions and rules in order to accurately detect structural features given a query compound by parsing IUPAC names."
- [intro] Checkmol assigns functional groups: "CO functional groups can be automatically assigned to a given structure by Checkmol [23], a freely available program."
