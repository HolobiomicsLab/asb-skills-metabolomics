---
name: direct-parent-selection-by-feature-weight
description: Use when a compound has multiple candidate ChemOnt categories after category
  reduction and you need to determine the single direct parent for a taxonomic classification
  record.
when_to_use_negative:
- Input is a single feature mapped to one category with no parent-child overlap —
  category reduction already yielded one candidate and no tie-break is needed
- Compounds from a category that undergo reactions (e.g. loss of oxygen, substitutions)
  that might not match the constraints described in a category description
- Input is an IUPAC name where parsing rules were insufficient to accurately assign
  the class — the upstream feature extraction step, not parent selection, is the failure
  point
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3314
tools:
- name: ClassyFire
  role: Program that implements feature-weight-based direct parent selection after
    category reduction in its rule-based classification pipeline
  repo: http://classyfire.wishartlab.com
- name: ChemAxon JChem API 15.5.25.0
  role: Used to standardize structures (normalization, aromatization) and support
    superstructure-search operations that generate the structural features weighted
    for parent selection
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-016-0174-y
    title: 'ClassyFire: automated chemical classification with a comprehensive, computable
      taxonomy'
  source_content_hash: sha256:62b45bd64b48d3bf9248a2d2a5a73a43356346fe43526b024e88484f6cb99363
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemAxon JChem API 15.5.25.0
  - ChemAxon Marvin
  - ClassyFire
  - OPSIN
  - Ruby
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# Direct parent selection by feature-weight

## Summary

Final classification step in ClassyFire: after category reduction, the direct parent is chosen as the category whose structural feature is largest (by weight), with weight ties broken using structural keys and a manually compiled exception list.

## When to use

Use when a compound has multiple candidate ChemOnt categories after category reduction and you need to determine the single direct parent for a taxonomic classification record.

## When NOT to use

- Input is a single feature mapped to one category with no parent-child overlap — category reduction already yielded one candidate and no tie-break is needed
- Compounds from a category that undergo reactions (e.g. loss of oxygen, substitutions) that might not match the constraints described in a category description
- Input is an IUPAC name where parsing rules were insufficient to accurately assign the class — the upstream feature extraction step, not parent selection, is the failure point

## Inputs

- Non-redundant list of ChemOnt candidate categories (after category reduction)
- Structural features with weights (counts of non-hydrogen atoms)
- Structural keys for each candidate category node
- Manually compiled exception list

## Outputs

- Single direct parent category for the compound
- Ancestor hierarchy path (Kingdom/SuperClass/Class/SubClass/...) rooted at the direct parent

## How to apply

After category reduction removes parent categories whenever a child is present, select the direct parent as the category defined by the largest structural feature that describes the compound. Feature weights are recorded as counts of non-hydrogen atoms during feature extraction. When two or more categories have equal weight, resolve the tie using each node's structural key — cycles, heterocycles, ring atoms, ring heteroatoms, halogen atoms, fused rings, and total heteroatoms — and apply the manually compiled exception list where expert judgment overrides weight alone (e.g. flavonoid glycoside over glycoside). The selected direct parent becomes the first entry in the ancestor hierarchy path emitted in the classification record.

## Related tools

- **ClassyFire** (Program that implements feature-weight-based direct parent selection after category reduction in its rule-based classification pipeline) — http://classyfire.wishartlab.com
- **ChemAxon JChem API 15.5.25.0** (Used to standardize structures (normalization, aromatization) and support superstructure-search operations that generate the structural features weighted for parent selection)

## Evaluation signals

- Spot-check expert-confirmed categories (e.g. androsterone as a 3-alpha-hydroxysteroid from an isomeric SMILES) match the computed direct parent and hierarchy path
- Verify output_records.csv contains a classification record per input compound, each with a single direct parent and full ancestor path
- Confirm the selected direct parent is defined by the largest structural feature among the non-redundant category list
- Check weight ties are resolved per structural key (cycles, heterocycles, ring atoms, ring heteroatoms, halogen atoms, fused rings, total heteroatoms) rather than arbitrarily
- Verify exception-list overrides (e.g. flavonoid glycoside over glycoside) produce the expert-preferred parent

## Limitations

- Compounds from a given chemical category undergo reactions (e.g. loss of oxygen, substitutions) that might not match the constraints described in a category description
- Creating more rules for IUPAC name parsing could help to assign some classes more accurately
- No specific software version stated for ClassyFire; source code was not yet released at the time of writing (promised by December 2016)

## Evidence

- [methods] The direct parent is the category defined by the largest structural feature that describes the compound. It is selected from the non-redundant list of categories obtained in the previous step.: "The direct parent is the category defined by the largest structural feature that describes the compound."
- [methods] For every pair of chemical categories, if there is a parent–child relationship (e.g. dioxanes [parent] and 1,2-dioxanes [child]), only the child node is retained: "For every pair of chemical categories, if there is a parent–child relationship (e.g. dioxanes [parent] and 1,2-dioxanes [child]), only the child node is retained"
- [methods] A manually compiled dictionary, which provides the weight and category for each feature, was used for the rule-based: "A manually compiled dictionary, which provides the weight and category for each feature, was used for the rule-based"
- [abstract] a Ruby API version is available at https://bitbucket.org/wishartlab/classyfire_api, which provides programmatic access to the ClassyFire server and database: "a Ruby API version is available at https://bitbucket.org/wishartlab/classyfire_api, which provides programmatic access to the ClassyFire server and database"
