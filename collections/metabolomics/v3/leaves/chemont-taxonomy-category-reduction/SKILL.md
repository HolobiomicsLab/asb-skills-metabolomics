---
name: chemont-taxonomy-category-reduction
description: Use when a compound's detected structural features have been mapped to
  ChemOnt categories and you must prune redundant ancestor categories before picking
  the direct parent in a ClassyFire-style pipeline.
when_to_use_negative:
- Before feature detection — if structural features have not yet been mapped to ChemOnt
  categories via superstructure search against the ~9000 SMARTS/Markush pattern library
  or the ~200 IUPAC-name parsing rules, there is nothing to reduce
- When the goal is selecting the Direct Parent itself — this skill only prunes ancestor
  nodes; the Direct Parent selection uses the reduced list plus feature weights, structural
  keys and the exception list
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: ClassyFire
  role: Encapsulates the Category Reduction step within its four-step classification
    process (Category Assignment and Category Reduction precede Direct Parent selection)
  repo: https://classyfire.wishartlab.com
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-016-0174-y
    title: 'ClassyFire: automated chemical classification with a comprehensive, computable
      taxonomy'
  source_content_hash: sha256:b237da9edf799edea534e1cb84e3e680fd968ae21e4a6ab65e9629a577d1f43d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemAxon JChem API 15.5.25.0
  - ChemAxon Marvin
  - ClassyFire
  - OPSIN
  - Ruby
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# chemont-taxonomy-category-reduction

## Summary

Part of the ClassyFire classification engine: after structural features are detected and mapped to ChemOnt categories, this skill reduces the assigned category set by iteratively removing parent categories whenever a child category is present (e.g. retain 1,2-dioxanes over dioxanes), before selecting the direct parent.

## When to use

Use when a compound's detected structural features have been mapped to ChemOnt categories and you must prune redundant ancestor categories before picking the direct parent in a ClassyFire-style pipeline.

## When NOT to use

- Before feature detection — if structural features have not yet been mapped to ChemOnt categories via superstructure search against the ~9000 SMARTS/Markush pattern library or the ~200 IUPAC-name parsing rules, there is nothing to reduce
- When the goal is selecting the Direct Parent itself — this skill only prunes ancestor nodes; the Direct Parent selection uses the reduced list plus feature weights, structural keys and the exception list

## Inputs

- Set of ChemOnt categories assigned to a compound from detected structural features (output of the Rule-based Category Assignment step)
- ChemOnt taxonomy with parent–child relationships between categories
- Manually compiled dictionary providing the weight and category for each detected feature (feature weights as counts of non-hydrogen atoms)

## Outputs

- Non-redundant list of categories (child nodes only) ready for Direct Parent selection

## How to apply

Take the non-redundant list of ChemOnt categories assigned to a compound by feature detection, then apply the Category Reduction rule: for every pair of categories, if there is a parent–child relationship in the ChemOnt taxonomy (e.g. dioxanes [parent] and 1,2-dioxanes [child]), retain only the child node, iterating until no ancestor of an assigned category remains in the list. This guarantees that the subsequent Direct Parent selection step operates on the most specific categories rather than both a node and its descendant, which would otherwise bias the largest-feature weight comparison toward overly general classes. The reduction runs after Feature Extraction and before Selection of the Direct Parent in the four-step ClassyFire classification process (Creation and Preprocessing of the Chemical Entity; Feature Extraction; Rule-based Category Assignment and Category Reduction; Selection of the Direct Parent). Weights come from a manually compiled dictionary that provides the weight and category for each detected feature, with feature weights recorded as counts of non-hydrogen atoms; ties between equally weighted categories are broken using each node's structural key (cycles, heterocycles, ring atoms, ring heteroatoms, halogen atoms, fused rings, total heteroatoms) and a manually compiled exception list (e.g. flavonoid glycoside over glycoside).

## Related tools

- **ClassyFire** (Encapsulates the Category Reduction step within its four-step classification process (Category Assignment and Category Reduction precede Direct Parent selection)) — https://classyfire.wishartlab.com

## Evaluation signals

- For every retained category, no ancestor of that category (as defined by the ChemOnt taxonomy) is also present in the reduced list
- Output of the reduction step feeds directly into Direct Parent selection — the direct parent must be one of the retained (non-redundant) child categories
- Spot-check a known case: a compound assigned both 'dioxanes' and '1,2-dioxanes' should retain only '1,2-dioxanes'
- A compound whose expert-confirmed direct parent is a specific child (e.g. androsterone as a 3-alpha-hydroxysteroid) must not be reclassified to a more general parent category after reduction and selection

## Limitations

- Compounds undergoing reactions (e.g. loss of oxygen, substitutions) may not match the constraints described in a category description, affecting which parent–child pairs are detected
- IUPAC name parsing rules (~200 regular expressions) may be insufficient for accurate class assignment in some cases, leading to missing child categories that would have pruned a parent
- The parent–child pruning rule and the exception list are manually compiled and depend on the quality of the ChemOnt taxonomy and the Chemical Classification Dictionary

## Evidence

- [methods] Category Reduction rule defined as keeping only child nodes: "For every pair of chemical categories, if there is a parent–child relationship (e.g. dioxanes [parent] and 1,2-dioxanes [child]), only the child node is retained"
- [methods] Category Reduction is step 3 of the 4-step classification process: "the ClassyFire classification process involves four steps: (1) Creation and Preprocessing of the Chemical Entity; (2) Feature Extraction; (3) Rule-based Category Assignment and Category Reduction;"
- [methods] Direct Parent selection uses the reduced list and largest structural feature: "The direct parent is the category defined by the largest structural feature that describes the compound. It is selected from the non-redundant list of categories obtained in the previous step."
- [methods] Weights come from a manually compiled dictionary used for rule-based assignment: "A manually compiled dictionary, which provides the weight and category for each feature, was used for the rule-based"
