---
name: ionization-mode-library-stratification
description: Use when auditing a metabolite standards library (e.g. Supplementary
  Table-style inventory with metabolite, chemical class, ionization mode columns)
  to report mode-level metabolite counts and per-class percentage composition.
when_to_use_negative:
- Input is already an aggregated summary of library composition rather than a per-metabolite
  inventory — recounting adds no value
- The library lacks ionization-mode or chemical-class annotations, so mode stratification
  and class aggregation cannot be performed
- The goal is spectral matching or annotation of unknowns against the library rather
  than characterizing the library itself
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: PlantMetSuite
  role: Web-based plant metabolomics tool whose in-house plant standards spectral
    library is the object being characterized; accessed via browser without installation
  repo: https://github.com/YaonanLab/PlantMetSuite
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/plants12152880
    title: 'PlantMetSuite: A User-Friendly Web-Based Tool for Metabolomics Analysis
      and Visualisation'
  source_content_hash: sha256:1de65f15664af570214ee971b6185d29c8803b059fe1de8d381ab688fc22e237
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YaonanLab/PlantMetSuite
  tools:
  - PlantMetSuite
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ionization-mode-library-stratification

## Summary

Characterizes the size and chemical-class composition of a metabolite standards spectral library by stratifying counts per ionization mode (positive/negative) and aggregating metabolites by chemical class into percentage shares.

## When to use

Use when auditing a metabolite standards library (e.g. Supplementary Table-style inventory with metabolite, chemical class, ionization mode columns) to report mode-level metabolite counts and per-class percentage composition.

## When NOT to use

- Input is already an aggregated summary of library composition rather than a per-metabolite inventory — recounting adds no value
- The library lacks ionization-mode or chemical-class annotations, so mode stratification and class aggregation cannot be performed
- The goal is spectral matching or annotation of unknowns against the library rather than characterizing the library itself

## Inputs

- Standards library inventory table (e.g. Supplementary Table S2) with one row per metabolite, annotated with chemical class and ionization mode (positive/negative)

## Outputs

- Composition table (CSV) with unique metabolite counts per ionization mode and per-chemical-class percentages
- Verified class-share listing (e.g. carboxylic acids and derivatives 19%, organooxygen compounds 14%, fatty acyls 8%, flavonoids 4%)

## How to apply

Load the standards library inventory listing each metabolite with its chemical class and ionization mode. Count unique metabolites per ionization mode separately — in PlantMetSuite's plant standards library this yields 1122 metabolites in positive mode and 1119 in negative mode, so near-but-unequal totals across modes are expected rather than an error. Aggregate metabolites by chemical class and compute each class's percentage of the library; verify computed shares against reported values (carboxylic acids and derivatives 19%, organooxygen compounds 14%, fatty acyls 8%, flavonoids 4%, benzene and substituted derivatives 3%, organonitrogen compounds 3%, keto acids and derivatives 2%, phenols 2%). Decision point: discrepancies above rounding tolerance (~1%) indicate parsing errors such as duplicate metabolite names or inconsistent class labels that must be reconciled before reporting. Write the verified mode-level counts and per-class percentages to a composition table (e.g. standards_library_composition.csv).

## Related tools

- **PlantMetSuite** (Web-based plant metabolomics tool whose in-house plant standards spectral library is the object being characterized; accessed via browser without installation) — https://github.com/YaonanLab/PlantMetSuite

## Evaluation signals

- Unique metabolite counts per mode match reported totals: 1122 in positive mode and 1119 in negative mode
- Per-class percentages match reported shares: carboxylic acids and derivatives 19%, organooxygen compounds 14%, fatty acyls 8%, flavonoids 4%, benzene and substituted derivatives 3%, organonitrogen compounds 3%, keto acids and derivatives 2%, phenols 2%
- Class percentages sum to approximately 100% within rounding tolerance
- Mode-level counts differ slightly (1122 vs 1119), confirming a metabolite is missing from one mode rather than an off-by-more-than-a-handful discrepancy
- Output CSV contains both mode-level counts and per-class percentage columns

## Limitations

- Composition percentages cover only the top classes (~55% combined); long-tail classes each fall below the 2% reporting threshold
- Counts depend on the library inventory being deduplicated and consistently class-labelled; inconsistent class labels will distort percentages
- Library size and class shares reflect PlantMetSuite's in-house plant standards at publication time (2023) and will shift as the tool undergoes 'regular updates and expansions'

## Evidence

- [supplementary] The in-house standards library contains 1122 metabolites in positive mode and 1119 in negative mode (source card task_004): [evidence span withheld — anchor status: not_found]
- [supplementary] Class composition percentages (source card task_004): "carboxylic acids and derivatives (19%), organooxygen compounds (14%), fatty acyls (8%), flavonoids (4%)"
- [abstract] PlantMetSuite is a browser-accessible tool requiring no installation: "PlantMetSuite can be accessed directly through a user's browser without the need for installation or programming skills."
- [discussion] Repository location for PlantMetSuite: "Source: github:github.com__YaonanLab__PlantMetSuite"
