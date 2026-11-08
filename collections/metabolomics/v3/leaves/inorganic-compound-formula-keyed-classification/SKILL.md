---
name: inorganic-compound-formula-keyed-classification
description: Decision support for classifying inorganic, organometallic, or carbon-exception
  compounds (cyanides, CO, CO2, carbon disulfide) when structural-rule-based taxonomy
  categories are needed rather than name- or text-based labels.
when_to_use_negative:
- Compounds that clearly belong to the organic Kingdom (the 26 organic SuperClasses),
  which use functional-group-based rather than elemental/formula-based structural
  keys
- Text- or name-based classification needs where no chemical structure or molecular
  formula is available
- Compounds whose structures are only available as IUPAC names that require the ~200-rule
  IUPAC name-parsing module instead of direct structural feature extraction
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: OBO-Edit
  role: Used to generate the ChemOnt OBO taxonomy file that must be parsed to load
    the inorganic category tree
- name: ClassyFire
  role: Contains the module specifically handling inorganic compounds, encoding elemental
    content, molecular formula, and oxyanion structural keys for each inorganic category
  repo: http://classyfire.wishartlab.com
- name: Ruby API (classyfire_api)
  role: Provides programmatic access to the ClassyFire server and database for retrieving
    classification results
  repo: https://bitbucket.org/wishartlab/classyfire_api
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-016-0174-y
    title: 'ClassyFire: automated chemical classification with a comprehensive, computable
      taxonomy'
  source_content_hash: sha256:4ff4076f61aac72428cd96663ad06db1b2db2c872b3804381c9bc0c66c15ebf7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClassyFire
  - OBO-Edit
  - Ruby API (classyfire_api)
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# inorganic-compound-formula-keyed-classification

## Summary

Reconstructs ClassyFire's inorganic and organometallic taxonomy branch from the ChemOnt OBO file, capturing all 674 inorganic categories keyed by computable structural features (elemental content, molecular formula, oxyanion lists). This fills the coverage gap left by existing chemical ontologies for non-organic compound types.

## When to use

Decision support for classifying inorganic, organometallic, or carbon-exception compounds (cyanides, CO, CO2, carbon disulfide) when structural-rule-based taxonomy categories are needed rather than name- or text-based labels.

## When NOT to use

- Compounds that clearly belong to the organic Kingdom (the 26 organic SuperClasses), which use functional-group-based rather than elemental/formula-based structural keys
- Text- or name-based classification needs where no chemical structure or molecular formula is available
- Compounds whose structures are only available as IUPAC names that require the ~200-rule IUPAC name-parsing module instead of direct structural feature extraction

## Inputs

- ChemOnt taxonomy OBO file (downloadable from the ClassyFire website, generated with OBO-Edit)
- Query compound structures or molecular formulas for rule-based category assignment

## Outputs

- inorganic_taxonomy_categories.json — records with category name, parent link, taxonomic level, and elemental/formula-based structural key (elemental content, molecular formula, oxyanion list)
- Taxonomy category assignments for query inorganic/organometallic compounds

## How to apply

Obtain the ChemOnt taxonomy OBO file (generated with OBO-Edit and downloadable from the ClassyFire website) and parse it into the full category tree rooted at 'Chemical entities', which spans up to 11 levels (Kingdom, SuperClass, Class, SubClass, etc.). Isolate the inorganic Kingdom branch — 5 inorganic SuperClasses versus 26 organic SuperClasses — and collect every inorganic category down to the maximum tree depth of 11. For each category, record its computable structural key exactly as encoded in the ClassyFire module dedicated to inorganic compounds: elemental content, molecular formula, and the list of oxyanions it contains. Honor the special carbon-containing exceptions (cyanide/isocyanide derivatives, carbon monoxide, carbon dioxide, carbon sulfide, carbon disulfide), which are routed into the inorganic branch despite containing carbon. Write each record (category name, parent link, taxonomic level, formula/atom-type structural key) to inorganic_taxonomy_categories.json.

## Related tools

- **OBO-Edit** (Used to generate the ChemOnt OBO taxonomy file that must be parsed to load the inorganic category tree)
- **ClassyFire** (Contains the module specifically handling inorganic compounds, encoding elemental content, molecular formula, and oxyanion structural keys for each inorganic category) — http://classyfire.wishartlab.com
- **Ruby API (classyfire_api)** (Provides programmatic access to the ClassyFire server and database for retrieving classification results) — https://bitbucket.org/wishartlab/classyfire_api

## Evaluation signals

- Output file inorganic_taxonomy_categories.json exists, parses as valid JSON, and contains 674 category records
- Every category record carries a parent link consistent with the 'Chemical entities' root of ChemOnt
- Each record carries an elemental/formula-based structural key (elemental content, molecular formula, oxyanion list) rather than a functional-group key
- Inorganic categories span up to 11 taxonomic levels, with 5 inorganic SuperClasses distinguishable from the 26 organic SuperClasses
- Carbon-containing exceptions (cyanide/isocyanide derivatives, CO, CO2, carbon sulfide, carbon disulfide) are present in the inorganic branch despite containing carbon

## Limitations

- Compounds from a given category may undergo reactions (e.g. loss of oxygen, substitutions) that do not match the constraints described in the category description
- IUPAC name parsing rules (~200 regular expressions) may be insufficient for accurate class assignment in some cases
- ClassyFire's source code was not yet released at the time of writing, limiting direct access to the inorganic module's internal rules

## Evidence

- [abstract] This new chemical taxonomy consists of up to 11 different levels (Kingdom, SuperClass, Class, SubClass, etc.) with each of the categories defined by unambiguous, computable structural rules: "This new chemical taxonomy consists of up to 11 different levels (Kingdom, SuperClass, Class, SubClass, etc.) with each of the categories defined by unambiguous, computable structural rules"
- [methods] The resulting OBO file was generated with OBO-Edit [34], and can be downloaded from the ClassyFire website.: "The resulting OBO file was generated with OBO-Edit [34], and can be downloaded from the ClassyFire website."
- [methods] The entire ClassyFire program has been converted to a web-based resource. It is a RESTful web application located at http://classyfire.wishartlab.com.: "The entire ClassyFire program has been converted to a web-based resource. It is a RESTful web application located at http://classyfire.wishartlab.com."
