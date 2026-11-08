---
name: lipid-refmet-analyte-class-mapping
description: Use when a user-supplied analyte list or uploaded measurement table must
  be matched to ALISTER's stability database — e.
when_to_use_negative:
- When the user's analytes use a naming scheme that is neither lipid shorthand nomenclature
  (for lipids) nor RefMet nomenclature (for polar metabolites) and cannot be reconciled
  — the app requires names matched against its underlying database
- When the uploaded .csv is not in tabular form with sample names in rows and analyte
  names in columns, since the app requires that structure and will not alter or store
  the data
- When the goal is prospective protocol planning or retrospective stability assessment
  for a single, individually named analyte in Analyte search rather than class-based
  or bulk mapping
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3172
tools:
- name: ALISTER
  role: Hosts the underlying stability database against which user-supplied analyte
    names are automatically matched, and provides the class-categorized selection
    lists (main lipid class or RefMet main class) used to drive Sample search, Protocol
    search, Analyte search, and the Data filtering mode.
  repo: https://github.com/Fraunhofer-ITMP/alister
- name: RShiny
  role: Implements the ALISTER web-application in which the analyte-name matching
    against the stability database and the class-based selection interface are delivered
    to the user.
- name: RefMet Database
  role: Supplies the standard nomenclature for polar metabolites so their names in
    the user dataset can be matched against ALISTER's database and grouped by RefMet
    main class.
  repo: https://metabolomicsworkbench.shinyapps.io/refmet_name_search/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.cca.2024.117858
    title: ALISTER – Application for lipid stability evaluation and research
  source_content_hash: sha256:739f9b5d537ecdc61b04eb717eb8ded10290c74c71e9ace6e32d136ac5f7cc7c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Fraunhofer-ITMP/alister
  tools:
  - ALISTER
  - R
  - RShiny
  - RefMet Database
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# lipid-refmet-analyte-class-mapping

## Summary

Maps lipid and polar-metabolite analyte names in user datasets to ALISTER's underlying stability database by matching against lipid shorthand nomenclature (lipids) and RefMet nomenclature (polar metabolites), so that analytes can be grouped by main class for class-based selection and stability queries.

## When to use

Use when a user-supplied analyte list or uploaded measurement table must be matched to ALISTER's stability database — e.g. lipid names in lipid shorthand nomenclature or polar metabolites in RefMet nomenclature — to enable class-based analyte selection in Sample search, Protocol search, Analyte search, or the Data filtering mode.

## When NOT to use

- When the user's analytes use a naming scheme that is neither lipid shorthand nomenclature (for lipids) nor RefMet nomenclature (for polar metabolites) and cannot be reconciled — the app requires names matched against its underlying database
- When the uploaded .csv is not in tabular form with sample names in rows and analyte names in columns, since the app requires that structure and will not alter or store the data
- When the goal is prospective protocol planning or retrospective stability assessment for a single, individually named analyte in Analyte search rather than class-based or bulk mapping

## Inputs

- User .csv file for upload into the app environment, containing input data in tabular form with sample names in the rows and analyte names in the columns (possibly transposed when displayed); can be an empty table with only analyte names
- List of lipid names following the lipid shorthand nomenclature
- List of polar metabolite names following the RefMet nomenclature
- Selection of compound classes of interest (main lipid class or main class according to the RefMet database)

## Outputs

- Automatically matched analyte names aligned to ALISTER's underlying database entries
- Class-categorized analyte sets (grouped by main lipid class or RefMet main class) suitable for class-based selection in the search modes
- Annotated dataset (in the Data filtering mode) where each analyte carries a stability indicator derived from the matched database entry

## How to apply

First, obtain the list of analytes of interest — either manually selected classes from the ALISTER app or a user .csv uploaded to the Data filtering mode containing sample names in rows and analyte names in columns. Second, automatically match each analyte name against ALISTER's underlying database: lipids are named after the lipid shorthand nomenclature while polar metabolites are named after their RefMet nomenclature, so name resolution depends on choosing the correct naming standard per analyte type. Third, use the matched main classes (main lipid class for lipids, RefMet main class for polar metabolites) to select compound classes of interest, with the option to 'Select all' groups at once. Fourth, when the uploaded names differ from database entries, rely on the automatic matching step to reconcile them before proceeding with stability evaluation; the README notes that the app requires tabular input with sample names in rows and analyte names in columns, and that data are not altered or stored. The rationale is that class-level grouping lets a user query stability for whole analyte classes (e.g. lipid classes) rather than individual compounds, which is how ALISTER's search modes are designed to be driven.

## Related tools

- **ALISTER** (Hosts the underlying stability database against which user-supplied analyte names are automatically matched, and provides the class-categorized selection lists (main lipid class or RefMet main class) used to drive Sample search, Protocol search, Analyte search, and the Data filtering mode.) — https://github.com/Fraunhofer-ITMP/alister
- **RShiny** (Implements the ALISTER web-application in which the analyte-name matching against the stability database and the class-based selection interface are delivered to the user.)
- **RefMet Database** (Supplies the standard nomenclature for polar metabolites so their names in the user dataset can be matched against ALISTER's database and grouped by RefMet main class.) — https://metabolomicsworkbench.shinyapps.io/refmet_name_search/

## Examples

```
upload .csv via ALISTER Data filtering mode (Shiny app: https://itmp.shinyapps.io/alister/) with sample names in rows and analyte names in columns; names auto-matched against ALISTER database
```

## Evaluation signals

- Every analyte name in the uploaded .csv has been matched against ALISTER's underlying database — no unmatched or silently dropped analytes remain before stability evaluation proceeds.
- Lipid analytes are classified by their main lipid class and polar metabolites by their main class according to the RefMet database, matching the categorization ALISTER uses in its class-selection lists.
- The uploaded file retains the required structure (sample names in rows, analyte names in columns) and the app confirms that the data were not altered or stored.
- For the Data filtering mode, each analyte in the annotated output dataset carries a visual stability indicator, confirming the name-mapping step fed the correct database entry.
- Hovering over an analyte in the app reveals how the user's input was interpreted (generalization of conditions), which validates that the matched database entry corresponds to the intended analyte.

## Limitations

- Automatic matching is only as good as the user's adherence to lipid shorthand nomenclature (lipids) or RefMet nomenclature (polar metabolites); non-standard names may fail to resolve or resolve to the wrong entry.
- ALISTER's underlying database coverage is limited to analytes for which pre-analytical stability data exist; analytes not in the database cannot be mapped.
- Current app contents are limited to influences of time delay and temperature during processing of (EDTA) plasma and serum samples, so mapped analytes are evaluated only in that pre-analytical context.
- Protocol search is currently only available for plasma samples, constraining the downstream use of mapped analyte classes in that specific search mode.

## Evidence

- [methods] Lipids are named after the lipid shorthand nomenclature, while polar metabolites are named after their RefMet nomenclature.: [evidence span withheld — anchor status: not_found]
- [readme] The analyte names will be automatically matched against the ones in ALISTER´s underlying database.: "The analyte names will be automatically matched against the ones in ALISTER´s underlying database."
- [readme] Analytes are categorized by their main lipid class (when lipids) or their main class according to the RefMet database (for polar metabolites).: "Analytes are categorized by their main lipid class (when lipids) or their main class according to the RefMet database (for polar metabolites). You can choose to 'Select all' groups at once."
- [readme] the app requires that the file contains the input data in tabular form, with sample names in the rows and analyte names in the columns: "Despite being displayed in a transposed way, the app requires that the file contains the input data in tabular form, with sample names in the rows and analyte names in the columns."
