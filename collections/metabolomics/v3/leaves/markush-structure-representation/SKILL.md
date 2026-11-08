---
name: markush-structure-representation
description: Use when defining structural patterns for rule-based compound classification
  where the feature contains variable substituents or partial group definitions that
  plain SMARTS cannot express unambiguously.
when_to_use_negative:
- The structural feature can be expressed unambiguously with a standard SMARTS string
  — only the ~4% of ClassyFire patterns that cannot be represented in SMARTS require
  the Markush format.
- The target is a fully specified single compound structure rather than a generic
  classification pattern.
- No ChemAxon JChem license or Marvin tool is available.
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_2258
tools:
- name: ChemAxon Marvin
  role: Editor used to draw and generate the Markush-format structural patterns for
    ambiguous classification features
- name: ChemAxon JChem API 15.5.25.0
  role: Provides the JChem Base storage and search engine in which Markush patterns
    are stored and queried against standardized structures
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-016-0174-y
    title: 'ClassyFire: automated chemical classification with a comprehensive, computable
      taxonomy'
  source_content_hash: sha256:145fc97128a9f96a2cd930b1e90e097776674994457ba095f3e3c13a70f8f543
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemAxon JChem API 15.5.25.0
  - ChemAxon Marvin
  - ClassyFire
  - OPSIN
  - Ruby
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# markush-structure-representation

## Summary

Represents ambiguous or generic structural patterns (e.g. R1=OR2, R3=[H]) that cannot be expressed with standard SMARTS as ChemAxon Markush structures generated with Marvin 5.11.5 and stored in ChemAxon JChem Base, forming part of ClassyFire's ~9000-pattern library for structural-feature detection.

## When to use

Use when defining structural patterns for rule-based compound classification where the feature contains variable substituents or partial group definitions that plain SMARTS cannot express unambiguously.

## When NOT to use

- The structural feature can be expressed unambiguously with a standard SMARTS string — only the ~4% of ClassyFire patterns that cannot be represented in SMARTS require the Markush format.
- The target is a fully specified single compound structure rather than a generic classification pattern.
- No ChemAxon JChem license or Marvin tool is available.

## Inputs

- Generic structural feature definition (e.g. a functional group with one or more R-group substituents)
- ChemAxon Marvin structure editor
- ChemAxon JChem Base pattern library

## Outputs

- Markush-format structural pattern stored in the ChemAxon JChem Base library for later superstructure matching
- ChemOnt category mapping via the weight/category dictionary when the pattern is matched against a query compound

## How to apply

Draw the generic pattern in ChemAxon Marvin (version 5.11.5 as used in the ClassyFire feature library), express ambiguous regions as R-atoms or Markush atom lists, and save the pattern in Markush format for storage in the ChemAxon JChem Base library. Each stored Markush pattern is subsequently matched against standardized query structures (normalization and aromatization applied) during the superstructure-search feature-extraction step, with detected features mapped to a ChemOnt category via a manually compiled weight/category dictionary where feature weights are counted as non-hydrogen atoms.

## Related tools

- **ChemAxon Marvin** (Editor used to draw and generate the Markush-format structural patterns for ambiguous classification features)
- **ChemAxon JChem API 15.5.25.0** (Provides the JChem Base storage and search engine in which Markush patterns are stored and queried against standardized structures)

## Evaluation signals

- The pattern encodes ambiguous R-group positions rather than fixed atoms (e.g. R1=OR2, R3=[H] for carbonyl-group features)
- The saved pattern round-trips losslessly through JChem Base storage and retrieval
- Matching a known member of the target ChemOnt category (e.g. a confirmed carboxylic-acid ester) detects the Markush-defined feature
- Pattern is used only after query standardization (normalization and aromatization) matching library preprocessing

## Limitations

- Markush patterns represent a minority (~4%) of the >9000 feature patterns; most features use plain SMARTS
- Feature weights derived from Markush matches count non-hydrogen atoms, which can affect direct-parent tie-breaking
- The paper does not document a step-by-step authoring protocol; details of Markush generation are implicit in the feature-library construction

## Evidence

- [methods] Markush patterns capture ambiguous structural features in ClassyFire's pattern library: "For these exceptions we used the Markush format [36], which is available through ChemAxon's Marvin tool."
- [methods] ClassyFire feature extraction uses superstructure search against a large pattern library: "The second step in the ClassyFire program involves the generation of structural features based on a combination of superstructure-search operations and various property calculations."
- [methods] Structures are standardized before feature detection: "each structure pattern was subjected to a set of standardization operations, including normalization and aromatization. Each query compound is subjected to the same operations"
- [methods] Markush-format exceptions are encoded explicitly (R1=OR2, R3=[H]): [evidence span withheld — anchor status: not_found]
