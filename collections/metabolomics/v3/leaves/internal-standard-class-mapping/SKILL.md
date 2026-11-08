---
name: internal-standard-class-mapping
description: Use when a lipidomics data matrix contains raw peak areas or peak heights
  plus internal standard rows with known concentrations, and species-level concentrations
  per lipid class are needed.
when_to_use_negative:
- Input intensities are already concentration values (no internal-standard conversion
  is needed)
- The data matrix lacks internal standard rows and no standard.txt table with Standard/Class/Conc.
  columns is available, so class-specific conversion factors cannot be built
- The analysis targets LipidOne 2.0's statistical, pathway, or biomarker functions
  rather than the Utility-section Lipid Data Quantification step
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: LipidOne 2.0
  role: 'Source implementation being reconstructed: its Utility-section Lipid Data
    Quantification function converts peak areas to concentrations via internal standards
    and post-processes the matrix'
  repo: https://lipidone.eu
- name: R
  role: Implementation environment for loading the data matrix and standard.txt with
    R data frames and computing class-specific concentration conversion factors
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:9999f3371cd478b81daf9a365901d02b5b27b97a6a9afd901fb591e4df742cfc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidOne 2.0
  - R
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# internal-standard-class-mapping

## Summary

Converts lipid peak area/peak height intensities into concentration values by mapping each internal standard (e.g., SPLASH Lipidomix species) to its lipid class via a standards table, then cleaning the resulting quantified matrix.

## When to use

Use when a lipidomics data matrix contains raw peak areas or peak heights plus internal standard rows with known concentrations, and species-level concentrations per lipid class are needed.

## When NOT to use

- Input intensities are already concentration values (no internal-standard conversion is needed)
- The data matrix lacks internal standard rows and no standard.txt table with Standard/Class/Conc. columns is available, so class-specific conversion factors cannot be built
- The analysis targets LipidOne 2.0's statistical, pathway, or biomarker functions rather than the Utility-section Lipid Data Quantification step

## Inputs

- Peak area or peak height data matrix including internal standard rows (e.g., SPLASH Lipidomix species)
- standard.txt table with columns Standard/Class/Conc. giving each internal standard's identity, lipid class, and known concentration

## Outputs

- quantified_matrix.csv: concentration matrix with internal standard rows removed and duplicate molecular species consolidated into single rows

## How to apply

Load the peak-area/peak-height data matrix (including internal standard rows such as SPLASH Lipidomix species) and the 'standard.txt' table with columns Standard/Class/Conc. into pandas or R data frames. For each internal standard, use its known concentration (Conc. column) and its lipid-class assignment (Class column) to build a class-specific conversion factor, and transform the peak areas of all species belonging to the matching class into concentrations. After quantification runs, automatically exclude the internal standard rows from the data matrix so they do not appear as analytes, then detect molecular species present in multiple rows and consolidate them by summation into a single entry per species. Write the resulting concentration matrix to quantified_matrix.csv, mirroring the post-quantification behavior of the Lipid Data Quantification function in the Utility section of LipidOne 2.0.

## Related tools

- **LipidOne 2.0** (Source implementation being reconstructed: its Utility-section Lipid Data Quantification function converts peak areas to concentrations via internal standards and post-processes the matrix) — https://lipidone.eu
- **R** (Implementation environment for loading the data matrix and standard.txt with R data frames and computing class-specific concentration conversion factors)

## Examples

```
Rscript quantify_lipids.R --matrix peak_areas.csv --standards standard.txt --output quantified_matrix.csv
```

## Evaluation signals

- No internal standard rows (e.g., SPLASH Lipidomix species) remain in quantified_matrix.csv
- Every molecular species in the output matrix is mapped to a class via standard.txt
- No duplicate molecular species labels persist in the output matrix after consolidation
- Species concentrations are consistent with the class-specific conversion factor derived from the standard's Conc. column
- Duplicate species rows were merged by summation of their quantified amounts, so consolidated values equal the sum of the original rows

## Limitations

- Conversion depends entirely on the accuracy of the standard.txt Conc. and Class columns; a misassigned class propagates an incorrect conversion factor to all species of that class
- LipidOne 2.0 is a web-based tool and the described Utility function is accessed through the platform; the standalone desktop LipidOne repository README describes a Perl/Windows application, a different codebase

## Evidence

- [other] In the Lipid Data Quantification function, after the quantification process runs, rows containing standards are automatically excluded from the data matrix, and duplicate entries for molecular species are consolidated into a single row.: [evidence span withheld — anchor status: not_found]
- [abstract] Web tool description: "LipidOne 2.0 (https://lipidone.eu) is a new web bioinformatic tool for the analysis of lipidomic data."
- [discussion] Back-end implementation language: "coding its back-end interac- tions with the R scripts"
- [readme] README describing the original standalone LipidOne software: "LipidOne is a stand alone software entirely written in Perl and implemented for the Microsoft Windows operating system."
