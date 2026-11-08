---
name: lipidomics-internal-standard-quantification
description: Use when a raw lipidomics matrix contains peak area/height intensities
  plus an internal standard table with known concentrations, and the goal is a concentration
  matrix per molecular species.
when_to_use_negative:
- Input is already a concentration matrix (peak-area-to-concentration conversion would
  double-transform the data)
- No internal standards are present or standard.txt lacks a Conc. value for a class,
  so no class-specific conversion factor can be built
- The analysis goal is annotation or pathway interpretation at class/molecular-species/building-block
  level rather than producing quantified concentrations
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: LipidOne 2.0
  role: 'Reference implementation: its Lipid Data Quantification function in the Utility
    section converts peak areas to concentrations via internal standards'
  repo: https://lipidone.eu
- name: R
  role: Used to implement the back-end quantification logic and to load the peak matrix
    and standard.txt as data frames
- name: LipidOne
  role: Standalone Perl predecessor whose LipidOne lineage this quantification utility
    extends
  repo: https://github.com/matteogiulietti/LipidOne
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:599822096f138f5195fcb3e490ce14e3b8025918a9d4e076ffdccae251a699d4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidOne
  - LipidOne 2.0
  - R
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# lipidomics-internal-standard-quantification

## Summary

Reconstructs the Lipid Data Quantification utility of LipidOne 2.0, converting peak-area or peak-height values into concentrations using class-specific internal standards (e.g., SPLASH Lipidomix species), then removes standard rows and consolidates duplicate molecular species entries.

## When to use

Use when a raw lipidomics matrix contains peak area/height intensities plus an internal standard table with known concentrations, and the goal is a concentration matrix per molecular species.

## When NOT to use

- Input is already a concentration matrix (peak-area-to-concentration conversion would double-transform the data)
- No internal standards are present or standard.txt lacks a Conc. value for a class, so no class-specific conversion factor can be built
- The analysis goal is annotation or pathway interpretation at class/molecular-species/building-block level rather than producing quantified concentrations

## Inputs

- Peak-area or peak-height data matrix (CSV), including internal standard rows such as SPLASH Lipidomix species
- standard.txt table with columns Standard/Class/Conc. mapping each internal standard to its lipid class and known concentration

## Outputs

- quantified_matrix.csv: concentration matrix with one row per molecular species, internal standards removed and duplicate species consolidated

## How to apply

Load the peak-area/peak-height matrix (which includes internal standard rows such as SPLASH Lipidomix species) and the 'standard.txt' table with columns Standard/Class/Conc. into pandas or R data frames. For each internal standard, use its known concentration (Conc. column) and its lipid-class assignment (Class column) to build a class-specific conversion factor, and apply it to the peak areas of all species belonging to the matching class. After quantification, exclude the internal standard rows from the data matrix so they do not appear as analytes, then detect molecular species appearing in multiple rows and consolidate them into a single entry per species. Write the resulting concentration matrix to quantified_matrix.csv. The rationale mirrors the Utility section of LipidOne 2.0, whose back-end interacts with R scripts.

## Related tools

- **LipidOne 2.0** (Reference implementation: its Lipid Data Quantification function in the Utility section converts peak areas to concentrations via internal standards) — https://lipidone.eu
- **R** (Used to implement the back-end quantification logic and to load the peak matrix and standard.txt as data frames)
- **LipidOne** (Standalone Perl predecessor whose LipidOne lineage this quantification utility extends) — https://github.com/matteogiulietti/LipidOne

## Examples

```
Rscript lipid_quantify.R --peak-matrix peaks.csv --standards standard.txt --out quantified_matrix.csv
```

## Evaluation signals

- No internal standard rows (e.g., SPLASH Lipidomix species) remain in the output matrix
- Every molecular species in the output is mapped to a lipid class via standard.txt
- No duplicate molecular species labels persist in the output matrix (all duplicates consolidated into single rows)
- Concentrations are finite, non-negative values consistent with the class-specific conversion factors derived from the Conc. column
- Output file quantified_matrix.csv exists with the expected species x sample dimensions relative to the input matrix

## Limitations

- Species whose class has no matching internal standard in standard.txt cannot be converted to concentration
- The standalone LipidOne README documents only a Microsoft Windows 64-bit build requiring GNUPLOT, so the Perl-based desktop code is not directly portable; the quantification logic should be reimplemented in R/pandas as described
- LipidOne 2.0 is web-based (lipidone.eu) and no installability/deployment information (Docker, conda, source install) is given, so reproducibility depends on reimplementing the steps

## Evidence

- [abstract] LipidOne 2.0 is described as a web bioinformatic tool for lipidomic data analysis: "LipidOne 2.0 (https://lipidone.eu) is a new web bioinformatic tool for the analysis of lipidomic data."
- [discussion] Back-end implemented with R scripts: "coding its back-end interac- tions with the R scripts"
- [abstract] Dataset preparation is a basic protocol of the workflow: "Basic Protocol 1: Dataset preparation for LipidOne 2.0"
- [readme] LipidOne is a standalone Perl lipidomic analysis tool: "LipidOne is a stand alone software entirely written in Perl and implemented for the Microsoft Windows operating system."
