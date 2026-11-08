---
name: composition-table-generation
description: Use when a metabolite standards library inventory lists metabolites with
  chemical class and ionisation mode, and you must report mode-level totals and per-class
  composition percentages.
when_to_use_negative:
- Input is already an aggregated composition table or per-mode count summary rather
  than a metabolite-level inventory
- The library inventory lacks chemical class or ionisation-mode annotations, since
  class percentages and mode-level totals cannot be computed
- The goal is annotating experimental peaks against the library rather than characterising
  the library's own composition
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: PlantMetSuite
  role: Web-based metabolomics tool whose in-house plant standards spectral library
    is the subject of the composition analysis; supplies the metabolite/class/mode
    inventory
  repo: https://github.com/YaonanLab/PlantMetSuite
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/plants12152880
    title: 'PlantMetSuite: A User-Friendly Web-Based Tool for Metabolomics Analysis
      and Visualisation'
  source_content_hash: sha256:a0ed2efc0d8636f2812f8dece33b3d7d33ac614511d9b61f7b4eaa7b47ca8598
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YaonanLab/PlantMetSuite
  tools:
  - PlantMetSuite
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# composition-table-generation

## Summary

Generates a verified composition table for a metabolite standards spectral library by counting unique metabolites per ionisation mode and aggregating metabolites into chemical classes with per-class percentages, as demonstrated for the PlantMetSuite in-house plant standards library.

## When to use

Use when a metabolite standards library inventory lists metabolites with chemical class and ionisation mode, and you must report mode-level totals and per-class composition percentages.

## When NOT to use

- Input is already an aggregated composition table or per-mode count summary rather than a metabolite-level inventory
- The library inventory lacks chemical class or ionisation-mode annotations, since class percentages and mode-level totals cannot be computed
- The goal is annotating experimental peaks against the library rather than characterising the library's own composition

## Inputs

- Standards library inventory table (e.g. Supplementary Table S2) listing each metabolite, its chemical class, and ionisation mode

## Outputs

- Composition table (standards_library_composition.csv) with unique metabolite counts per ionisation mode (e.g. 1122 positive / 1119 negative) and per-class percentage columns

## How to apply

Load the standards library inventory (e.g. Supplementary Table S2 listing each metabolite, its chemical class, and ionization mode). Count unique metabolites per ionization mode and reconcile the totals against reported figures — for PlantMetSuite this is 1122 metabolites in positive mode and 1119 in negative mode, so any mismatch flags duplicated or misclassified entries. Aggregate metabolites by chemical class and compute each class's percentage of the library, then verify against reported values (carboxylic acids and derivatives 19%, organooxygen compounds 14%, fatty acyls 8%, flavonoids 4%, benzene and substituted derivatives 3%, organonitrogen compounds 3%, keto acids and derivatives 2%, phenols 2%). Write the verified composition table with mode-level counts and per-class percentages to a CSV output (e.g. standards_library_composition.csv). The rationale is that library composition and mode coverage determine which metabolite classes a tool's annotation workflow can support in positive and negative ionisation.

## Related tools

- **PlantMetSuite** (Web-based metabolomics tool whose in-house plant standards spectral library is the subject of the composition analysis; supplies the metabolite/class/mode inventory) — https://github.com/YaonanLab/PlantMetSuite

## Evaluation signals

- Unique metabolite counts per mode match reported totals: 1122 in positive mode and 1119 in negative mode
- Computed class percentages reproduce reported values: carboxylic acids and derivatives 19%, organooxygen compounds 14%, fatty acyls 8%, flavonoids 4%, benzene and substituted derivatives 3%, organonitrogen compounds 3%, keto acids and derivatives 2%, phenols 2%
- Every metabolite in the inventory is assigned exactly one chemical class and at least one ionisation mode before aggregation
- Output CSV contains both mode-level counts and per-class percentage columns
- Percentages across classes sum to approximately 100% of the library

## Limitations

- Composition percentages reflect only classes represented in the in-house standards library; metabolites without class annotations are excluded from per-class aggregation
- Positive- and negative-mode counts differ (1122 vs 1119), so mode-level totals should be reported separately rather than pooled

## Evidence

- [supplementary] Library composition counts: [evidence span withheld — anchor status: not_found]
- [abstract] Tool scope: "PlantMetSuite encompasses interactive bioinformatics tools and databases specifically tailored to plant metabolomics data, facilitating upstream-to-downstream analysis in metabolomics"
