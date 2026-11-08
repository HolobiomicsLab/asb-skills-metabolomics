---
name: iupac-name-feature-parsing
description: Use when input compounds carry only IUPAC names (no structure strings)
  and structural features must still be extracted for taxonomy classification, or
  when name-parsing must be validated against OPSIN-converted structures.
when_to_use_negative:
- Input is already a structure string (SMILES/SDF/InChI) — use the structure-based
  superstructure search path instead
- Query compounds need physico-chemical property calculation rather than feature detection
tools:
- name: OPSIN
  role: Converts parsed IUPAC names to structures so regex-derived features can be
    cross-validated
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-016-0174-y
    title: 'ClassyFire: automated chemical classification with a comprehensive, computable
      taxonomy'
  source_content_hash: sha256:40ab5e5529d704b62326b7b8f0f130a3aaf6388fe1cab800aa273af16ccc756c
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
yes_rate: 0.9
n_claims: 10
---

# iupac-name-feature-parsing

## Summary

Detect structural features from IUPAC chemical names alone, using ~200 regular-expression rules as a fallback when a structure (SMILES/InChI) is unavailable, in support of ClassyFire's rule-based taxonomy assignment.

## When to use

Use when input compounds carry only IUPAC names (no structure strings) and structural features must still be extracted for taxonomy classification, or when name-parsing must be validated against OPSIN-converted structures.

## When NOT to use

- Input is already a structure string (SMILES/SDF/InChI) — use the structure-based superstructure search path instead
- Query compounds need physico-chemical property calculation rather than feature detection

## Inputs

- IUPAC chemical name strings (no structure string available)
- Optionally OPSIN-converted structures for cross-validation

## Outputs

- List of detected structural features with weights (counts of non-hydrogen atoms)
- Mapped ChemOnt category identifiers

## How to apply

Run the regex-based name parsing module, which applies a set of ~200 regular expressions and rules to detect structural features directly from the query compound's IUPAC name. This module is one of several feature-detection paths inside ClassyFire, complementing SMARTS/Markush superstructure searches against the >9000 pattern library. The parsed features are mapped to ChemOnt categories via the manually compiled weight/category dictionary, where each feature's weight counts non-hydrogen atoms. Any feature assigned this way then enters the same category-reduction and direct-parent selection logic as structure-derived features. When an OPSIN library is available, parsed IUPAC names can be converted to structures first, allowing cross-validation of the regex-derived features against structure-derived ones.

## Related tools

- **OPSIN** (Converts parsed IUPAC names to structures so regex-derived features can be cross-validated)

## Examples

```
python classyfire.py --name "(2S,4R)-2-methyl-4-(propan-2-yl)cyclohexan-1-ol"
```

## Evaluation signals

- Detected features must map to valid ChemOnt categories
- Feature weights must be non-zero, non-negative integer counts of non-hydrogen atoms
- Regex-derived features should be consistent with OPSIN-converted structure features for the same compound
- A spot-check that a known compound (e.g. androsterone as an isomeric SMILES or IUPAC name) yields the expected direct-parent category

## Limitations

- The IUPAC name parsing rules are insufficient for accurate class assignment in some cases; creating more parsing rules could improve accuracy
- Compounds undergoing reactions (e.g. loss of oxygen, substitutions) may not match category description constraints even when features are correctly parsed

## Evidence

- [methods] ~200 regular expressions and rules to detect structural features from IUPAC names: "We developed a module, which uses a set of ~200 regular expressions and rules in order to accurately detect structural features given a query compound by parsing IUPAC names."
- [methods] IUPAC names converted to structures via OPSIN before chemical objects are created: [evidence span withheld — anchor status: not_found]
- [methods] ClassyFire's standardization: normalization and aromatization: "each structure pattern was subjected to a set of standardization operations, including normalization and aromatization. Each query compound is subjected to the same operations"
- [methods] ClassyFire's mapping of features to taxonomy categories via a manually compiled dictionary: "A manually compiled dictionary, which provides the weight and category for each feature, was used for the rule-based"
- [methods] Direct parent chosen as the category with the largest structural feature: "The direct parent is the category defined by the largest structural feature that describes the compound."
