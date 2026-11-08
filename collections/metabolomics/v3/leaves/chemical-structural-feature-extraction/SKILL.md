---
name: chemical-structural-feature-extraction
description: Use when you have standardized chemical structures (SMILES/Molfile) and
  need machine-readable structural features — functional groups, ring counts, heteroatom
  composition, property constraints — to drive rule-based category assignment.
when_to_use_negative:
- Your classification must be based on biological activity, use, or textual metadata
  rather than purely on chemical structure — ClassyFire assigns categories using only
  structures and structural features
- The compound is undergoing/representing a reaction state (e.g. loss of oxygen, substitutions)
  that may not satisfy category description constraints
- You only need parent-level labels without structural justification — a pre-computed
  annotation database (e.g. PubChem's ClassyFire classification) may suffice
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: Checkmol
  role: Freely available program used to automatically assign CO (carbonyl) functional
    groups to a given structure during feature extraction
- name: ChemAxon JChem API 15.5.25.0
  role: Calculates the physico-chemical properties that supplement substructure searches
    in the feature vector
- name: OPSIN
  role: Converts IUPAC names to the corresponding structure before any chemical object
    is created for feature extraction
- name: ChemAxon Marvin
  role: Provides Markush format for structural patterns that cannot be expressed as
    ordinary SMARTS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-016-0174-y
    title: 'ClassyFire: automated chemical classification with a comprehensive, computable
      taxonomy'
  source_content_hash: sha256:5fa27eac6be29b3f1b258d2630279caa3526ea385c84561a292b293290fb74fb
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# chemical-structural-feature-extraction

## Summary

Extract structural features (substructures, functional groups, ring systems, physico-chemical properties) from a chemical structure so it can be matched against rule-based taxonomy categories such as ChemOnt's >4800 categories. This is the core engine step of ClassyFire's purely structure-based classification.

## When to use

Use when you have standardized chemical structures (SMILES/Molfile) and need machine-readable structural features — functional groups, ring counts, heteroatom composition, property constraints — to drive rule-based category assignment.

## When NOT to use

- Your classification must be based on biological activity, use, or textual metadata rather than purely on chemical structure — ClassyFire assigns categories using only structures and structural features
- The compound is undergoing/representing a reaction state (e.g. loss of oxygen, substitutions) that may not satisfy category description constraints
- You only need parent-level labels without structural justification — a pre-computed annotation database (e.g. PubChem's ClassyFire classification) may suffice

## Inputs

- Standardized chemical structure representation (e.g. SMILES, Molfile, InChI) for the query compound
- IUPAC chemical name (convertible via OPSIN or parseable by the ~200-rule name-parsing module)
- Manually compiled feature-to-category dictionary with weights (from the Chemical Classification Dictionary)

## Outputs

- List of structural features (functional groups, ring systems, heteroatom composition, substructure matches) for the compound
- Computed physico-chemical property values
- Non-redundant candidate category list with weights, and the selected direct parent category (CHEMONTID)

## How to apply

First standardize the query structure: apply normalization and aromatization operations, exactly as patterns in the rule library were standardized, so matching is consistent. Then generate features via a combination of superstructure-search operations (SMARTS-like structural pattern matching) and physico-chemical property calculations (performed with ChemAxon's JChem API version 15.5.25.0). CO-containing functional groups can be assigned with Checkmol. If only an IUPAC name is available, convert it to a structure with OPSIN, or use a name-parsing module of ~200 regular expressions and rules to detect structural features directly from the name; Markush-format exceptions are handled through ChemAxon's Marvin. Each detected feature is then looked up in a manually compiled dictionary mapping feature to weight and taxonomy category, enabling rule-based category assignment and reduction (keep only the child of any parent-child category pair) before selecting the direct parent.

## Related tools

- **Checkmol** (Freely available program used to automatically assign CO (carbonyl) functional groups to a given structure during feature extraction)
- **ChemAxon JChem API 15.5.25.0** (Calculates the physico-chemical properties that supplement substructure searches in the feature vector)
- **OPSIN** (Converts IUPAC names to the corresponding structure before any chemical object is created for feature extraction)
- **ChemAxon Marvin** (Provides Markush format for structural patterns that cannot be expressed as ordinary SMARTS)

## Evaluation signals

- Standardization reproducibility: re-running normalization and aromatization on the same input yields identical feature sets
- Feature-to-category consistency: every detected feature maps to a valid category/node in the ChemOnt taxonomy (4825 categories, up to 11 levels)
- Category reduction invariant: for any parent-child category pair (e.g. dioxanes vs 1,2-dioxanes), only the child is retained in the non-redundant list
- Direct parent sanity check: the selected direct parent corresponds to the largest structural feature describing the compound
- Known-compound spot check: classification results for reference molecules match those published for DrugBank, HMDB, ChEBI or LIPID MAPS annotations

## Limitations

- Compounds from a given chemical category may undergo reactions (e.g. loss of oxygen, substitutions) that do not match the constraints described in a category description
- IUPAC name parsing rules are insufficient for accurate class assignment in some cases; adding more rules could help assign some classes more accurately
- At the time of writing, the ClassyFire source code had not yet been released (open-source release promised by December 2016), limiting reproducibility of the exact feature-extraction implementation

## Evidence

- [methods] The second step in the ClassyFire program involves the generation of structural features based on a combination of superstructure-search operations and various property calculations.: "The second step in the ClassyFire program involves the generation of structural features based on a combination of superstructure-search operations and various property calculations."
- [methods] Each query compound is subjected to the same standardization as the rule patterns.: "each structure pattern was subjected to a set of standardization operations, including normalization and aromatization. Each query compound is subjected to the same operations"
- [methods] The name-parsing module uses ~200 regular expressions to detect structural features.: "We developed a module, which uses a set of ~200 regular expressions and rules in order to accurately detect structural features given a query compound by parsing IUPAC names."
- [methods] ChemAxon's JChem API (version 15.5.25.0) computes the physico-chemical properties.: "The calculation of physico-chemical properties is performed using ChemAxon’s JChem API (version 15.5.25.0)."
- [methods] Features are mapped to categories via a manually compiled dictionary with weights.: "each feature is then mapped to its corresponding category or node in the taxonomy. A manually compiled dictionary, which provides the weight and category for each feature, was used for the rule-based"
