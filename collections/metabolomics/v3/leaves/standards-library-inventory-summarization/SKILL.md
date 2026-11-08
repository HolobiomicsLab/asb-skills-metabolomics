---
name: standards-library-inventory-summarization
description: Use when you have an inventory table of metabolite standards with chemical
  class and ionization mode columns and need mode-level metabolite counts plus per-class
  composition percentages.
when_to_use_negative:
- Input is a raw feature table or untargeted LC-MS peak list rather than a curated
  standards library inventory with class annotations.
- The goal is spectral matching or metabolite annotation against the library, not
  summarizing the library's own composition.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: PlantMetSuite
  role: Web-based plant metabolomics toolkit whose in-house plant standards spectral
    library (Supplementary Table S2) is the inventory being summarized; library metadata
    accompanies the tool.
  repo: https://github.com/YaonanLab/PlantMetSuite
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/plants12152880
    title: 'PlantMetSuite: A User-Friendly Web-Based Tool for Metabolomics Analysis
      and Visualisation'
  source_content_hash: sha256:261151de63f55dd6801e402de78332d7d35d4aee8955bccd56d0190dbe8551ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YaonanLab/PlantMetSuite
  tools:
  - PlantMetSuite
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# standards-library-inventory-summarization

## Summary

Summarize the composition of a metabolite standards spectral library by counting unique metabolites per ionization mode and aggregating metabolites by chemical class into per-class percentages, as done for the PlantMetSuite in-house plant standards library.

## When to use

Use when you have an inventory table of metabolite standards with chemical class and ionization mode columns and need mode-level metabolite counts plus per-class composition percentages.

## When NOT to use

- Input is a raw feature table or untargeted LC-MS peak list rather than a curated standards library inventory with class annotations.
- The goal is spectral matching or metabolite annotation against the library, not summarizing the library's own composition.

## Inputs

- Supplementary table or CSV inventory of metabolite standards with metabolite identifier, chemical class label, and ionization mode (positive/negative) columns (e.g. Supplementary Table S2)

## Outputs

- Composition table (e.g. standards_library_composition.csv) with metabolite counts per ionization mode and per-chemical-class percentage shares

## How to apply

Load the standards library inventory (e.g. Supplementary Table S2) listing each metabolite, its chemical class, and ionization mode. Count unique metabolites separately per ionization mode — in PlantMetSuite this yields 1122 metabolites in positive mode and 1119 in negative mode, so the two modes are near-equal but not identical, and uniqueness must be verified rather than assumed. Aggregate metabolites by chemical class (using the class labels provided in the inventory, e.g. ClassyFire-style categories like carboxylic acids and derivatives) and compute each class's percentage of the total library. Cross-check the computed percentages against any reported values (carboxylic acids and derivatives 19%, organooxygen compounds 14%, fatty acyls 8%, flavonoids 4%, benzene and substituted derivatives 3%, organonitrogen compounds 3%, keto acids and derivatives 2%, phenols 2%). Write the verified mode-level counts and per-class percentages to a tabular output such as standards_library_composition.csv.

## Related tools

- **PlantMetSuite** (Web-based plant metabolomics toolkit whose in-house plant standards spectral library (Supplementary Table S2) is the inventory being summarized; library metadata accompanies the tool.) — https://github.com/YaonanLab/PlantMetSuite

## Examples

```
python summarize_library.py --input Supplementary_Table_S2_standards.csv --mode-column ionization --class-column chemical_class --output standards_library_composition.csv
```

## Evaluation signals

- Positive-mode count equals 1122 and negative-mode count equals 1119 when deduplicated by unique metabolite.
- Computed class percentages reproduce the reported distribution: carboxylic acids and derivatives 19%, organooxygen compounds 14%, fatty acyls 8%, flavonoids 4%.
- Sum of all per-class percentages equals ~100% and every metabolite is assigned to exactly one class.
- Output CSV contains both mode-level counts and per-class percentage columns.

## Limitations

- Class composition depends entirely on the class labels pre-assigned in the source inventory; relabeling under a different ontology changes the percentages.
- Counts per mode may differ slightly (1122 vs 1119 for PlantMetSuite), so per-mode uniqueness must be recomputed rather than copied from one mode to the other.
- The summarized library is plant-specific and in-house; the composition snapshot may change as PlantMetSuite undergoes regular updates and expansions.

## Evidence

- [results] The in-house standards library contains 1122 metabolites in positive mode and 1119 in negative mode, and is composed of carboxylic acids and derivatives (19%), organooxygen compounds (14%), fatty acyls (8%), flavonoids (4%): [evidence span withheld — anchor status: not_found]
- [abstract] PlantMetSuite provides interactive bioinformatics tools and databases tailored to plant metabolomics data, supporting upstream-to-downstream and multi-omics analysis: "interactive bioinformatics tools and databases specifically tailored to plant metabolomics data, facilitating upstream-to-downstream analysis in metabolomics"
- [abstract] PlantMetSuite is browser-accessible without installation or programming skills and is freely available with regular updates planned: "accessed directly through a user's browser without the need for installation or programming skills. The tool is freely available and will undergo regular updates"
