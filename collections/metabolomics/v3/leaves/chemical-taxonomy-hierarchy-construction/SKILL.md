---
name: chemical-taxonomy-hierarchy-construction
description: Use when building or reconstructing a hierarchical chemical ontology
  from structural classification rules — e.g. assembling the ChemOnt tree from SMARTS/Markush
  rules into an OBO file with parent 'is_a' links.
when_to_use_negative:
- You need to classify a query compound against an already-built taxonomy — that is
  ClassyFire's classification pipeline (feature extraction, rule-based category assignment,
  direct-parent selection), not taxonomy construction
- The domain is fully covered by an existing ontology (e.g. organic compounds in established
  ontologies) — ChemOnt's dedicated construction effort targets the inorganic/organometallic
  coverage gap
- You need non-structural, usage-based classification rather than rules computable
  purely from chemical structure
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0089
tools:
- name: OBO-Edit
  role: Used to generate and encode the ChemOnt hierarchy in OBO format; the resulting
    OBO file is downloadable from the ClassyFire website
- name: ChemAxon Marvin
  role: Provides the Markush format used to express structural rules for the ~4% of
    category patterns that need exception handling
- name: ClassyFire
  role: Source of the taxonomy and its computable structural rules; hosts the ChemOnt
    OBO file and the inorganic-compound classification module
  repo: https://classyfire.wishartlab.com
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1186/s13321-016-0174-y
    title: 'ClassyFire: automated chemical classification with a comprehensive, computable
      taxonomy'
  source_content_hash: sha256:6a41b73fc15111c8a5f35c8a771947beda434aa7abd455fb4c7799660a2ca78e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemAxon Marvin
  - ClassyFire
  - OBO-Edit
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# chemical-taxonomy-hierarchy-construction

## Summary

Constructs a large, computable, purely structure-based chemical taxonomy (ChemOnt) as a hierarchy of >4800 categories across up to 11 Linnaean-style levels, encoding each category with machine-executable structural rules (SMARTS, Markush patterns, or logical expressions) in OBO format plus a JSON mirror.

## When to use

Use when building or reconstructing a hierarchical chemical ontology from structural classification rules — e.g. assembling the ChemOnt tree from SMARTS/Markush rules into an OBO file with parent 'is_a' links.

## When NOT to use

- You need to classify a query compound against an already-built taxonomy — that is ClassyFire's classification pipeline (feature extraction, rule-based category assignment, direct-parent selection), not taxonomy construction
- The domain is fully covered by an existing ontology (e.g. organic compounds in established ontologies) — ChemOnt's dedicated construction effort targets the inorganic/organometallic coverage gap
- You need non-structural, usage-based classification rather than rules computable purely from chemical structure

## Inputs

- Set of category definitions with structural rules (SMARTS strings, Markush patterns from ChemAxon Marvin, logical expressions over features like RingCount and AtomCount)
- English text definitions and synonyms for each category (9012 synonyms in ChemOnt)
- Parent-child ('is_a') relationships defining the hierarchy rooted at 'Chemical entities'

## Outputs

- ChemOnt taxonomy .obo file generated with OBO-Edit
- JSON mirror of the category tree for programmatic access
- Per-category records (name, parent link, taxonomic level, structural key) e.g. inorganic_taxonomy_categories.json with formula/atom-type structural keys

## How to apply

Enumerate the taxonomic levels from the taxonomy definition: 2 Kingdoms (organic vs inorganic), 31 SuperClasses (26 organic + 5 inorganic), 764 Classes, 1729 SubClasses, and 2296 lower-level categories, for 4825 categories below the root 'Chemical entities'. Attach to each category a computable structural rule: a SMARTS string (over 9000 are generated), a Markush pattern for exceptions (about 4% of patterns, authored via ChemAxon Marvin), a logical expression over features such as RingCount and AtomCount (e.g. the alkane rule RingCount(A)=0 AND AtomCount(C,A)>0 AND AtomCount(H,A)=2*AtomCount(C,A)+2), or a union-of-subcategories definition for composite classes like phenylpropanoids, polyketides, and alkaloids. Link categories by 'is_a' parent edges using the Linnaean-inspired level names (Kingdom, SuperClass, Class, SubClass, then levels 5–11), honoring special carbon-containing inorganic exceptions (cyanide/isocyanide derivatives, CO2, CO, carbon sulfide, carbon disulfide). Encode the hierarchy in Open Biological and Biomedical Ontologies (OBO) format using OBO-Edit, and emit a parallel JSON mirror for programmatic access. Annotate each category with an English text definition and synonyms (9012 synonyms in ChemOnt), preferring IUPAC or expert-selected terms as primary labels.

## Related tools

- **OBO-Edit** (Used to generate and encode the ChemOnt hierarchy in OBO format; the resulting OBO file is downloadable from the ClassyFire website)
- **ChemAxon Marvin** (Provides the Markush format used to express structural rules for the ~4% of category patterns that need exception handling)
- **ClassyFire** (Source of the taxonomy and its computable structural rules; hosts the ChemOnt OBO file and the inorganic-compound classification module) — https://classyfire.wishartlab.com

## Evaluation signals

- Total category count equals 4825 below the root, plus the root node 'Chemical entities'
- Maximum tree depth equals 11 levels with an average depth of five levels per node
- Level counts match: 2 Kingdoms, 31 SuperClasses (26 organic + 5 inorganic), 764 Classes, 1729 SubClasses, 2296 lower-level categories
- Every node carries an 'is_a' parent edge and a definition field (structural rule or union definition)
- Spot-check representative rules (e.g. thiazoles SMARTS, alkane rule RingCount(A)=0 AND AtomCount(H,A)=2*AtomCount(C,A)+2) match the published definitions; output file parses as valid JSON/OBO

## Limitations

- Compounds from a given category may undergo reactions (e.g. loss of oxygen, substitutions) that no longer match the constraints of the category description
- IUPAC name parsing rules are insufficient for accurate class assignment in some cases; more parsing rules would help
- Source code for ClassyFire was not yet released at the time of writing (promised open source by December 2016)

## Evidence

- [abstract] taxonomy levels with computable rules: "This new chemical taxonomy consists of up to 11 different levels (Kingdom, SuperClass, Class, SubClass, etc.)"
- [methods] OBO file generation: "The resulting OBO file was generated with OBO-Edit [34], and can be downloaded from the ClassyFire website."
- [methods] Markush patterns for exceptions: "For these exceptions we used the Markush format [36], which is available through ChemAxon’s Marvin tool."
- [intro] computable taxonomy and ontology goal: "we describe a comprehensive, flexible, computable, chemical taxonomy along with a fully annotated chemical ontology (ChemOnt)"
