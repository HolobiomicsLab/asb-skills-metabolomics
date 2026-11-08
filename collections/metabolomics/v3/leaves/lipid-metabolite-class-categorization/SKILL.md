---
name: lipid-metabolite-class-categorization
description: Use when a set of lipids and/or polar metabolites measured from blood
  samples must be grouped into compound classes before querying pre-analytical stability
  in ALISTER's Sample search mode.
when_to_use_negative:
- The analyte of interest is a single, specific compound — 'Analyte search' allows
  individual drop-down selection instead of class grouping.
- The task is prospective sampling-protocol planning with a class-level selection
  — that is Protocol search, not retrospective class categorization for stability
  lookup.
- The input .csv already contains analytes that do not match lipid shorthand or RefMet
  nomenclature, so automatic class matching will fail.
edam_topics:
- http://edamontology.org/topic_3339
- http://edamontology.org/topic_0153
tools:
- name: ALISTER
  role: Web-app whose Sample search mode receives the class-categorized analyte selection
    and matches it against the aggregated stability database
  repo: https://github.com/Fraunhofer-ITMP/alister
- name: RefMet Database
  role: Provides the nomenclature used to name and classify polar metabolites
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.cca.2024.117858
    title: ALISTER – Application for lipid stability evaluation and research
  source_content_hash: sha256:198ad93e85ec9f4945e4ad3ab51d1d052a40a976ebd398795f28e80416838630
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Fraunhofer-ITMP/alister
  tools:
  - ALISTER
  - R
  - RefMet Database
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3339
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# lipid-metabolite-class-categorization

## Summary

Categorize measured analytes into their main lipid class (using lipid shorthand nomenclature) or main RefMet class (for polar metabolites) so that ALISTER's Sample search can aggregate stability data at the class level.

## When to use

Use when a set of lipids and/or polar metabolites measured from blood samples must be grouped into compound classes before querying pre-analytical stability in ALISTER's Sample search mode.

## When NOT to use

- The analyte of interest is a single, specific compound — 'Analyte search' allows individual drop-down selection instead of class grouping.
- The task is prospective sampling-protocol planning with a class-level selection — that is Protocol search, not retrospective class categorization for stability lookup.
- The input .csv already contains analytes that do not match lipid shorthand or RefMet nomenclature, so automatic class matching will fail.

## Inputs

- List of lipids named per lipid shorthand nomenclature
- List of polar metabolites named per RefMet nomenclature
- .csv measurement table with sample names in rows and analyte names in columns (for Data filtering mode)

## Outputs

- Set of selected compound classes (e.g. main lipid class or RefMet class groups) to feed into Sample search
- Class-matched analyte set against ALISTER's underlying stability database

## How to apply

Take the list of measured analytes and assign each one to a class: lipids are named and categorized by their main lipid class following lipid shorthand nomenclature, while polar metabolites are named after their RefMet nomenclature. In the Sample search interface, select the compound classes of interest from the list, with an option to 'Select all' groups at once. In the Data filtering mode, the app automatically matches analyte names uploaded in a .csv file (samples in rows, analytes in columns) against the underlying database using these nomenclatures. The rationale is that stability information in ALISTER is aggregated per class, so class-level selection is the entry point for retrospective stability assessment at the default 20% fold-change instability threshold (30% when conditions are combined).

## Related tools

- **ALISTER** (Web-app whose Sample search mode receives the class-categorized analyte selection and matches it against the aggregated stability database) — https://github.com/Fraunhofer-ITMP/alister
- **RefMet Database** (Provides the nomenclature used to name and classify polar metabolites)

## Evaluation signals

- Every analyte in the input list maps to exactly one main lipid class or one RefMet class before submission to Sample search
- Class names in the selection correspond to the ALISTER database vocabulary (lipid shorthand nomenclature / RefMet nomenclature); unmatched analyte names are flagged
- Uploaded .csv in Data filtering mode has sample names in rows and analyte names in columns
- The 'Select all' option, when used, yields a selection covering all available class groups

## Limitations

- Current app contents are limited to (EDTA) plasma and serum samples, so categorization is only useful for those matrices.
- Automatic matching in Data filtering mode requires analyte names to follow lipid shorthand or RefMet nomenclature.
- Protocol search (the prospective counterpart) is currently only available for plasma samples.

## Evidence

- [readme] Analytes are categorized by their main lipid class (when lipids) or their main class according to the RefMet database (for polar metabolites).: "Analytes are categorized by their main lipid class (when lipids) or their main class according to the RefMet database"
- [readme] Lipids are named after the lipid shorthand nomenclature, while polar metabolites are named after their RefMet nomenclature.: "Lipids are named after the lipid shorthand nomenclature[^2], while polar metabolites are named after their RefMet nomenclature[^3]."
- [intro] Current app contents are limited to the influences of time delay and temperature during processing of (EDTA) plasma and serum samples: "Current app contents are limited to the influences of time delay and temperature during processing of (EDTA) plasma and serum samples"
- [readme] Select the compound classes of interest from the list. You can choose to 'Select all' groups at once.: [evidence span withheld — anchor status: not_found]
- [readme] The app requires that the file contains the input data in tabular form, with sample names in the rows and analyte names in the columns.: "the app requires that the file contains the input data in tabular form, with sample names in the rows and analyte names in the columns."
