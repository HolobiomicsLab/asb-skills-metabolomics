---
name: smarts-superstructure-pattern-matching
description: Use when a chemical structure (SMILES/SDF/InChI) needs structural-feature
  detection to drive ChemOnt category assignment, i.e. ClassyFire's Feature Extraction
  step prior to rule-based category assignment.
when_to_use_negative:
- The compound is undergoing transformations (e.g. loss of oxygen, substitutions)
  such that it no longer matches the structural constraints encoded in a category
  description
- The only input is an IUPAC name that OPSIN cannot parse or that the ~200 regular-expression
  parsing rules cannot resolve to structural features — name-parsing rules alone are
  acknowledged as insufficient for accurate class assignment in some cases
- 'A fully automatic, source-available classifier is required: ClassyFire''s source
  code was not yet released at the time of writing (promised open source by December
  2016)'
tools:
- name: ClassyFire
  role: 'Executes the Feature Extraction step: superstructure-search operations of
    standardized query structures against the SMARTS/Markush pattern library, plus
    property calculations used alongside matching'
  repo: https://classyfire.wishartlab.com
- name: ChemAxon Marvin
  role: Used to create the ~4% of pattern-library entries that cannot be expressed
    as SMARTS, encoded in the Markush format
- name: ChemAxon JChem API 15.5.25.0
  role: Performs physico-chemical property calculations that complement superstructure-search
    feature detection, and hosts the pattern library via JChem Base
- name: OPSIN
  role: Converts IUPAC name inputs to structures so that feature detection can run;
    IUPAC names are also parsed with ~200 regular-expression rules to catch features
    missed by structure matching
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-016-0174-y
    title: 'ClassyFire: automated chemical classification with a comprehensive, computable
      taxonomy'
  source_content_hash: sha256:79da3519ff160b3a21c5d85b9a4a84126287bdcea1e71791ad93ee346d5c4453
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemAxon JChem API 15.5.25.0
  - ChemAxon Marvin
  - ClassyFire
  - OPSIN
  - Ruby
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# smarts-superstructure-pattern-matching

## Summary

Detects structural features in a query compound by running superstructure searches against ClassyFire's library of over 9000 manually designed SMARTS patterns (with ~4% Markush structures), as the Feature Extraction step of rule-based ChemOnt classification.

## When to use

Use when a chemical structure (SMILES/SDF/InChI) needs structural-feature detection to drive ChemOnt category assignment, i.e. ClassyFire's Feature Extraction step prior to rule-based category assignment.

## When NOT to use

- The compound is undergoing transformations (e.g. loss of oxygen, substitutions) such that it no longer matches the structural constraints encoded in a category description
- The only input is an IUPAC name that OPSIN cannot parse or that the ~200 regular-expression parsing rules cannot resolve to structural features — name-parsing rules alone are acknowledged as insufficient for accurate class assignment in some cases
- A fully automatic, source-available classifier is required: ClassyFire's source code was not yet released at the time of writing (promised open source by December 2016)

## Inputs

- Query compound structures in SMILES, SDF, InChI, IUPAC name, or FASTA format
- Library of >9000 manually designed SMARTS structural patterns (~4% as ChemAxon Markush structures) stored in ChemAxon JChem Base

## Outputs

- List of detected structural features per compound, each with a weight (count of non-hydrogen atoms) and mapped ChemOnt category
- Non-redundant assigned-category set passed to Category Reduction and Direct Parent selection

## How to apply

Standardize each query compound with the same normalization and aromatization operations applied to the pattern library, so pattern matching is performed on comparable representations. Run superstructure-search operations of each query against the library of over 9000 manually designed SMARTS patterns stored in ChemAxon JChem Base, where ~4% of patterns are Markush structures created with ChemAxon Marvin because they cannot be expressed as plain SMARTS. Supplement SMARTS matching with logical feature rules for inorganics and with ~200 regular-expression rules applied to IUPAC names (structures parsed from names via OPSIN) for features not capturable structurally. For each detected feature, record its weight as the count of non-hydrogen atoms and map it to its ChemOnt category using the manually compiled weight/category dictionary; this non-redundant feature/category list feeds directly into Category Reduction and Direct Parent selection.

## Related tools

- **ClassyFire** (Executes the Feature Extraction step: superstructure-search operations of standardized query structures against the SMARTS/Markush pattern library, plus property calculations used alongside matching) — https://classyfire.wishartlab.com
- **ChemAxon Marvin** (Used to create the ~4% of pattern-library entries that cannot be expressed as SMARTS, encoded in the Markush format)
- **ChemAxon JChem API 15.5.25.0** (Performs physico-chemical property calculations that complement superstructure-search feature detection, and hosts the pattern library via JChem Base)
- **OPSIN** (Converts IUPAC name inputs to structures so that feature detection can run; IUPAC names are also parsed with ~200 regular-expression rules to catch features missed by structure matching)

## Evaluation signals

- Each input compound yields a non-empty detected-feature list; features carry weights equal to counts of non-hydrogen atoms
- Every detected feature maps to a valid ChemOnt node via the manually compiled weight/category dictionary
- Spot-check expert-confirmed categories (e.g. androsterone from isomeric SMILES should classify as a 3-alpha-hydroxysteroid) match the resulting direct parent and hierarchy path
- Query and pattern-library structures have undergone the same standardization operations (normalization and aromatization) before matching
- Downstream Category Reduction retains only child categories (e.g. 1,2-dioxanes over dioxanes), confirming the feature/category list was complete enough to establish parent-child relationships

## Limitations

- Compounds from a given chemical category may undergo reactions (e.g. loss of oxygen, substitutions) such that they no longer match the constraints of a category description
- More IUPAC name-parsing rules are needed — the ~200 regular-expression rules are insufficient for accurate class assignment in some cases
- No specific software version was stated for ClassyFire itself, and its source code was not yet released at the time of writing (open source promised by December 2016)

## Evidence

- [methods] Feature extraction combines superstructure search with property calculations: "The second step in the ClassyFire program involves the generation of structural features based on a combination of superstructure-search operations and various property calculations."
- [methods] Structures are standardized identically for pattern library and queries: "each structure pattern was subjected to a set of standardization operations, including normalization and aromatization. Each query compound is subjected to the same operations"
- [methods] Markush format via Marvin used for patterns not expressible as SMARTS: "For these exceptions we used the Markush format [36], which is available through ChemAxon’s Marvin tool."
- [methods] Manually compiled weight/category dictionary maps features to categories: "each feature is then mapped to its corresponding category or node in the taxonomy. A manually compiled dictionary, which provides the weight and category for each feature, was used for the rule-based"
- [methods] IUPAC name parsing supplements structure-based feature detection: "We developed a module, which uses a set of ~200 regular expressions and rules in order to accurately detect structural features given a query compound by parsing IUPAC names."
