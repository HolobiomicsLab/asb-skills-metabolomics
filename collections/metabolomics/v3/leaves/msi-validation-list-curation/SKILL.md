---
name: msi-validation-list-curation
description: Use when a matrix-related peak-annotation method for AgLDI/silver-nanolayer
  MSI needs a ground-truth list to validate classification performance, or when Table
  2-style theoretical Ag peak masses must be reproduced.
when_to_use_negative:
- The goal is to annotate/remove matrix peaks in an already-processed peak matrix
  — that is rMSIcleanup's annotate_matrix/remove_matrix workflow, not list curation
- The MSI dataset was acquired without a silver-sputtered nanolayer, so silver-cluster
  and silver-adduct peaks are not the relevant chemical space
- A different LDI-promoting material (not silver) is used; the positive/negative class
  definitions are silver-specific
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: enviPat
  role: Computes theoretical monoisotopic masses and isotopic pattern intensities
    for every positive- and negative-class entry of the validation list
- name: rMSIcleanup
  role: The peak-classification algorithm whose performance is assessed against the
    curated validation list
  repo: https://github.com/gbaquer/rMSIcleanup
- name: rMSIproc
  role: Generates the centroid-mode peak matrix of the 14 AgLDI datasets to which
    the validation list is matched
  repo: https://github.com/prafols/rMSIproc
- name: R
  role: Environment in which enviPat fine-structure calculations and compilation of
    validation_list.csv are carried out
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-020-00449-0
    title: 'rMSIcleanup: an open-source tool for matrix-related peak annotation in
      mass spectrometry imaging and its application to silver-assisted laser desorption/ionization'
  source_content_hash: sha256:e5117a831de8edc73cd291e4c0601b3448346405911bb1e21d40a3c52ee64203
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gbaquer/rMSIcleanup
  tools:
  - R
  - enviPat
  - rMSIcleanup
  - rMSIproc
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# msi-validation-list-curation

## Summary

Curates a ground-truth validation list of possible and not-possible silver-related peaks (positive class: Ag clusters; negative class: silver salts, diatomic salts and silver adducts of lipids/waxes) for benchmarking AgLDI MSI peak-classification algorithms such as rMSIcleanup.

## When to use

Use when a matrix-related peak-annotation method for AgLDI/silver-nanolayer MSI needs a ground-truth list to validate classification performance, or when Table 2-style theoretical Ag peak masses must be reproduced.

## When NOT to use

- The goal is to annotate/remove matrix peaks in an already-processed peak matrix — that is rMSIcleanup's annotate_matrix/remove_matrix workflow, not list curation
- The MSI dataset was acquired without a silver-sputtered nanolayer, so silver-cluster and silver-adduct peaks are not the relevant chemical space
- A different LDI-promoting material (not silver) is used; the positive/negative class definitions are silver-specific

## Inputs

- Silver isotopic abundances from the methods section (107Ag 51.84%, 109Ag 48.16%)
- Chemical formula list of positive-class Ag_n+ clusters within the acquired mass range
- Chemical formula list of negative-class silver salts ([AgF]n ... [AgBF4]n) and silver adducts ([C27H56+Ag]+ ... [C30H60O2+Ag]+)
- Mass range of the typical AgLDI MSI dataset to be validated

## Outputs

- validation_list.csv with columns chemical formula, class, type and monoisotopic mass (Table 2 mirror)
- Theoretical isotopic pattern intensities per entry from enviPat fine-structure calculations

## How to apply

Use enviPat's isotope fine-structure routines to compute theoretical monoisotopic masses and isotopic pattern intensities from the silver isotopic abundances given in the methods (107Ag 51.84%, 109Ag 48.16%). Enumerate the positive class as Ag_n+ clusters (n = 1 up to the mass-range limit, e.g. n = 1 to 10) covering the full mass range of a typical AgLDI dataset. Enumerate the negative class as neutral silver salts and synthetic compounds ([AgF]n, [AgCl]n, [AgBr]n, [AgI]n, [AgH]n, [AgH2]n, [AgHe]n, [AgNO3]n, [AgTh2]n, [AgF2]n, [AgBF4]n) plus silver adducts of plant/wax/insect compounds ([C27H56+Ag]+, [C29H60+Ag]+, [C31H64+Ag]+, [C26H54O+Ag]+, [C28H58O+Ag]+, [C30H62O+Ag]+, [C26H52O2+Ag]+, [C30H60O2+Ag]+) that are not expected in animal or plant samples. Compile both classes into a single table with columns chemical formula, class, type and monoisotopic mass, mirroring Table 2 of the paper, and save it as validation_list.csv. Validate each entry's monoisotopic mass against the reported values (e.g. [Ag]+ 106.9051, [AgF] 125.9036, [AgBr] 185.8229) within the isotopic-calculation tolerance.

## Related tools

- **enviPat** (Computes theoretical monoisotopic masses and isotopic pattern intensities for every positive- and negative-class entry of the validation list)
- **rMSIcleanup** (The peak-classification algorithm whose performance is assessed against the curated validation list) — https://github.com/gbaquer/rMSIcleanup
- **rMSIproc** (Generates the centroid-mode peak matrix of the 14 AgLDI datasets to which the validation list is matched) — https://github.com/prafols/rMSIproc
- **R** (Environment in which enviPat fine-structure calculations and compilation of validation_list.csv are carried out)

## Examples

```
library(enviPat); # compute fine structure for Ag species from the 107Ag 51.84% / 109Ag 48.16% abundances; write.csv(validation_list, file="validation_list.csv", row.names=FALSE)
```

## Evaluation signals

- Each entry's monoisotopic mass matches the reported Table 2 value (e.g. [Ag]+ 106.9051, [AgF] 125.9036, [AgBr] 185.8229) within the isotopic-calculation tolerance
- The positive class spans the full mass range of a typical AgLDI dataset (clusters n = 1 up to the mass-range limit)
- The negative class contains all negative-class types from the paper: diatomic salts, polyatomic salts and silver adducts of long-chain hydrocarbons, alcohols and fatty-acid-like compounds
- The list supports correct classification of silver-cluster signals — the paper benchmarked classifiers at AUC 0.97 (S1) and 0.91 (S2)
- validation_list.csv contains one row per formula with non-null class, type and monoisotopic mass fields

## Limitations

- The list is defined for a typical AgLDI experiment; it may not cover matrix-related peaks from other LDI-promoting materials
- Negative-class entries are compounds 'not expected in animal or plant samples'; if a sample type differs, class assignments must be re-evaluated
- Overlapping/isobaric peaks require the separate overlapping peak detection feature; the validation list alone does not resolve them

## Evidence

- [methods] enviPat isotope fine-structure for theoretical patterns: "are calculated using the open-source package enviPat [23], a fast and memory-efficient algorithm to compute theoretical isotope patterns."
- [intro] 14 AgLDI datasets and the validation list: "A total of 14 MSI datasets acquired with an Ag-sputtered nanolayer from three different laboratories, were used for validation."
- [abstract] classification correctness benchmark: "The algorithm was able to correctly classify m/z signals related to silver clusters."
- [intro] overlapping peak caveat: "The algorithm also incorporates an overlapping peak detection feature to prevent misclassification of overlapped or isobaric ions"
