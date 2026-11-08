---
name: metabolite-count-verification
description: Use when auditing a spectral/metabolite standards library inventory (e.g.
  Supplementary Table listing metabolites with chemical class and ionization mode)
  to validate reported metabolite counts per mode and class-level composition percentages.
when_to_use_negative:
- Input is already an aggregated composition/summary table rather than a per-metabolite
  inventory, so counts cannot be independently recomputed
- The goal is feature annotation of untargeted LC-MS peak lists against a library,
  not auditing the library's own size and class composition
- The inventory lacks ionization-mode labels, making per-mode counting (1122 positive
  / 1119 negative style verification) impossible
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: PlantMetSuite
  role: Web-based plant metabolomics platform whose in-house plant standards spectral
    library is the subject of the count/coverage verification; the skill audits the
    library distributed with this tool
  repo: github.com/YaonanLab/PlantMetSuite
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/plants12152880
    title: 'PlantMetSuite: A User-Friendly Web-Based Tool for Metabolomics Analysis
      and Visualisation'
  source_content_hash: sha256:d5099f965cd4e59119681345bc110c459ea9e87e3f380a63f3b8cfbfa891cf65
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PlantMetSuite
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# metabolite-count-verification

## Summary

Verifies the composition of a metabolite standards spectral library by counting unique metabolites per ionisation mode (positive/negative) and computing per-chemical-class percentages, as demonstrated on the PlantMetSuite in-house plant standards library.

## When to use

Use when auditing a spectral/metabolite standards library inventory (e.g. Supplementary Table listing metabolites with chemical class and ionization mode) to validate reported metabolite counts per mode and class-level composition percentages.

## When NOT to use

- Input is already an aggregated composition/summary table rather than a per-metabolite inventory, so counts cannot be independently recomputed
- The goal is feature annotation of untargeted LC-MS peak lists against a library, not auditing the library's own size and class composition
- The inventory lacks ionization-mode labels, making per-mode counting (1122 positive / 1119 negative style verification) impossible

## Inputs

- Standards library inventory table (e.g. Supplementary Table S2) listing each metabolite, its chemical class, and ionization mode (positive/negative)

## Outputs

- Composition table (e.g. standards_library_composition.csv) with metabolite counts per ionization mode and per-class percentage of the library

## How to apply

Load the standards library inventory table (e.g. Supplementary Table S2) where each row records a metabolite, its chemical class, and its ionization mode. Count unique metabolites within each ionization mode separately — in the PlantMetSuite reference case this yields 1122 metabolites in positive mode and 1119 in negative mode — and confirm these match the published totals. Then aggregate metabolites by chemical class and compute each class's percentage of the library, checking the computed values against reported composition (carboxylic acids and derivatives 19%, organooxygen compounds 14%, fatty acyls 8%, flavonoids 4%, benzene and substituted derivatives 3%, organonitrogen compounds 3%, keto acids and derivatives 2%, phenols 2%). Rationale: the article stresses that plant metabolomics lacks dedicated plant-specific metabolite databases, so verifying library coverage and class balance is a prerequisite step before downstream annotation or matching with PlantMetSuite.

## Related tools

- **PlantMetSuite** (Web-based plant metabolomics platform whose in-house plant standards spectral library is the subject of the count/coverage verification; the skill audits the library distributed with this tool) — github.com/YaonanLab/PlantMetSuite

## Evaluation signals

- Unique-metabolite counts per mode reproduce the reported totals: 1122 in positive mode and 1119 in negative mode
- Computed class percentages match reported values (carboxylic acids and derivatives 19%, organooxygen compounds 14%, fatty acyls 8%, flavonoids 4%, benzene and substituted derivatives 3%, organonitrogen compounds 3%, keto acids and derivatives 2%, phenols 2%)
- Positive and negative mode counts differ by exactly 3 metabolites (1122 vs 1119), consistent with a small set of metabolites absent from one mode
- Per-class percentages sum to the library total (no metabolite double-counted or dropped across classes)
- Output CSV contains one row per chemical class plus mode-level counts matching the source inventory row count

## Limitations

- Verification depends entirely on the accuracy and completeness of the supplied inventory table (e.g. Supplementary Table S2); ambiguous or missing mode/class labels propagate directly
- Composition percentages reflect the specific in-house PlantMetSuite plant standards library and will not generalize to other spectral libraries
- PlantMetSuite is under active development with regular updates planned, so library counts (1122/1119) may change between versions

## Evidence

- [abstract] Existing plant metabolomics solutions lack a dedicated plant-specific metabolite database and pose usability challenges: "Existing solutions lack a dedicated plant-specific metabolite database and pose usability challenges."
- [abstract] PlantMetSuite encompasses interactive bioinformatics tools and databases specifically tailored to plant metabolomics data: "PlantMetSuite encompasses interactive bioinformatics tools and databases specifically tailored to plant metabolomics data, facilitating upstream-to-downstream analysis in metabolomics and supporting"
- [supplementary] Source card finding: library contains 1122 metabolites in positive mode and 1119 in negative mode with class composition percentages: [evidence span withheld — anchor status: not_found]
