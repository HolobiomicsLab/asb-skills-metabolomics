---
name: structural-key-annotation-extraction
description: Use when building annotation records for taxonomy categories or compounds
  that must be defined by unambiguous, computable structural keys (elemental composition,
  formula, oxyanions) parsed from a ChemOnt OBO file, as ClassyFire does for its inorganic
  branch.
when_to_use_negative:
- Classifying organic compounds by functional group — use the standard ClassyFire
  feature-extraction and rule-based category-assignment pipeline instead
- Compounds that undergo reactions (e.g. loss of oxygen, substitutions) may not match
  the constraints described in a category description, so structural-key annotation
  alone can mis-assign them
- Query compounds for which only an IUPAC name is available and the ~200 name-parsing
  rules are insufficient for accurate class assignment
edam_topics:
- http://edamontology.org/topic_0089
tools:
- name: ClassyFire
  role: Source of the inorganic-compound classification module whose category definitions
    encode the elemental-content, formula, and oxyanion structural keys
  repo: http://classyfire.wishartlab.com
- name: OBO-Edit
  role: Tool with which the ChemOnt OBO file containing the category tree and parent
    links was generated; the OBO file is the parse target for structural-key extraction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-016-0174-y
    title: 'ClassyFire: automated chemical classification with a comprehensive, computable
      taxonomy'
  source_content_hash: sha256:7a9ceed754d93f071d57ee9eef57a52ed7451ede803ff321920a5525f9132ccb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClassyFire
  - OBO-Edit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# structural-key-annotation-extraction

## Summary

Extracts computable structural keys (elemental content, molecular formula, oxyanion lists) from the ChemOnt taxonomy's category definitions — especially the inorganic branch — and writes them as parent-linked, machine-readable annotation records.

## When to use

Use when building annotation records for taxonomy categories or compounds that must be defined by unambiguous, computable structural keys (elemental composition, formula, oxyanions) parsed from a ChemOnt OBO file, as ClassyFire does for its inorganic branch.

## When NOT to use

- Classifying organic compounds by functional group — use the standard ClassyFire feature-extraction and rule-based category-assignment pipeline instead
- Compounds that undergo reactions (e.g. loss of oxygen, substitutions) may not match the constraints described in a category description, so structural-key annotation alone can mis-assign them
- Query compounds for which only an IUPAC name is available and the ~200 name-parsing rules are insufficient for accurate class assignment

## Inputs

- ChemOnt taxonomy OBO file (rooted at 'Chemical entities', up to 11 taxonomic levels)
- Inorganic Kingdom branch categories (5 SuperClasses, up to 674 categories by molecular formula and atom types)
- ClassyFire inorganic-compound module definitions (elemental content, molecular formula, oxyanion lists)

## Outputs

- inorganic_taxonomy_categories.json — records with category name, parent link, taxonomic level, and elemental/formula-based structural key
- Oxyanion membership lists per inorganic category

## How to apply

Obtain and parse the ChemOnt OBO file (generated with OBO-Edit and downloadable from the ClassyFire website) to load the full category tree rooted at 'Chemical entities', which spans up to 11 levels (Kingdom, SuperClass, Class, SubClass, etc.). Isolate the inorganic Kingdom branch — 5 inorganic SuperClasses versus 26 organic SuperClasses — and collect categories at all levels up to maximum depth 11. For each category, extract the structural key as encoded in the ClassyFire inorganic-handling module: elemental content, molecular formula, and the list of oxyanions it contains, honoring the special carbon-containing exceptions (cyanide/isocyanide derivatives, carbon monoxide, carbon dioxide, carbon sulfide, carbon disulfide). Map each feature to its corresponding category node, keeping the parent–child relationships intact (for parent–child pairs, only the child node carries the finest key). Write category records (name, parent link, taxonomic level, formula/atom-type structural key) to a JSON file such as inorganic_taxonomy_categories.json.

## Related tools

- **ClassyFire** (Source of the inorganic-compound classification module whose category definitions encode the elemental-content, formula, and oxyanion structural keys) — http://classyfire.wishartlab.com
- **OBO-Edit** (Tool with which the ChemOnt OBO file containing the category tree and parent links was generated; the OBO file is the parse target for structural-key extraction)

## Examples

```
python parse_chemont_inorganic.py --obo ChemOnt.obo --root 'Chemical entities' --branch inorganic --out inorganic_taxonomy_categories.json
```

## Evaluation signals

- Output file exists and parses as valid JSON
- Every category record carries a parent link into the ChemOnt tree rooted at 'Chemical entities'
- Each record's structural key is consistent with the inorganic branch: elemental content, molecular formula, and oxyanion list present
- Special carbon-containing exceptions (cyanide/isocyanide derivatives, CO, CO2, carbon sulfide, carbon disulfide) appear in the inorganic branch despite containing carbon
- Taxonomic levels do not exceed the maximum tree depth of 11

## Limitations

- Source code for ClassyFire was not yet released at the time of writing (promised by December 2016), so the inorganic module's exact key logic may need to be reproduced from category descriptions
- Compounds undergoing reactions (e.g. loss of oxygen, substitutions) may not match the constraints described in a category description
- No specific software version is stated for ClassyFire, complicating reproducibility of the key extraction

## Evidence

- [methods] OBO file provenance: "The resulting OBO file was generated with OBO-Edit [34], and can be downloaded from the ClassyFire website."
- [abstract] Computable structural rules across 11 levels: "This new chemical taxonomy consists of up to 11 different levels (Kingdom, SuperClass, Class, SubClass, etc.) with each of the categories defined by unambiguous, computable structural rules"
- [methods] Feature-to-category mapping via dictionary: "A manually compiled dictionary, which provides the weight and category for each feature, was used for the rule-based"
- [methods] Child-node retention in category reduction: "For every pair of chemical categories, if there is a parent–child relationship (e.g. dioxanes [parent] and 1,2-dioxanes [child]), only the child node is retained"
