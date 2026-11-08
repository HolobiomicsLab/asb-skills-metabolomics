---
name: silver-cluster-mass-annotation
description: Use when building or validating a silver-cluster peak list for AgLDI
  MSI experiments — e.g. to benchmark rMSIcleanup's peak classification algorithm
  against a chemically defined set of positive and negative silver-related peaks.
when_to_use_negative:
- Input is already a processed peak matrix without accompanying chemical formula definitions
  for silver clusters/adducts — this skill builds the validation list, not the classification
  itself
- The LDI-promoting material is not silver (e.g. organic MALDI matrices) — the isotopic
  pattern logic and possible/not-possible peak definitions are silver-specific
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: enviPat
  role: Computes theoretical isotope fine-structure patterns and monoisotopic masses
    used to populate the validation list
- name: rMSIcleanup
  role: Downstream consumer of the validation list — classifies observed peaks in
    AgLDI MSI images against the possible/not-possible silver-related peak definitions
  repo: https://github.com/gbaquer/rMSIcleanup
- name: R
  role: Execution environment for enviPat and rMSIcleanup
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-020-00449-0
    title: 'rMSIcleanup: an open-source tool for matrix-related peak annotation in
      mass spectrometry imaging and its application to silver-assisted laser desorption/ionization'
  source_content_hash: sha256:e33e7b739837dc292a8035dcb9cab2be88ad79a41a72fc0e3f5e809db99654f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gbaquer/rMSIcleanup
  tools:
  - R
  - enviPat
  - rMSIcleanup
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# silver-cluster-mass-annotation

## Summary

Compile a validation list of possible and not-possible silver-related peaks by computing theoretical monoisotopic masses and isotopic patterns of Agn+ clusters and non-cluster silver adducts with enviPat, so that peak classification in AgLDI MSI data can be evaluated against known chemistry.

## When to use

Use when building or validating a silver-cluster peak list for AgLDI MSI experiments — e.g. to benchmark rMSIcleanup's peak classification algorithm against a chemically defined set of positive and negative silver-related peaks.

## When NOT to use

- Input is already a processed peak matrix without accompanying chemical formula definitions for silver clusters/adducts — this skill builds the validation list, not the classification itself
- The LDI-promoting material is not silver (e.g. organic MALDI matrices) — the isotopic pattern logic and possible/not-possible peak definitions are silver-specific

## Inputs

- Silver isotopic abundances (107Ag 51.84%, 109Ag 48.16%)
- Chemical formulas of silver clusters Agn+ within the acquired mass range (positive class)
- Chemical formulas of silver salts/synthetic compounds and silver adducts of plant/wax/insect compounds (negative class)

## Outputs

- validation_list.csv — table of chemical formula, class, type, and monoisotopic mass for each possible and not-possible silver-related peak, mirroring Table 2

## How to apply

Use enviPat to compute theoretical monoisotopic masses and isotopic pattern intensities for Agn+ clusters up to the mass range limit (e.g. n=1 to n=10) from silver isotopic abundances 107Ag 51.84% and 109Ag 48.16%. Separately compute the same quantities for the negative class: silver salts and synthetic compounds ([AgF]n, [AgCl]n, [AgBr]n, [AgI]n, [AgH]n, [AgH2]n, [AgHe]n, [AgNO3]n, [AgTh2]n, [AgF2]n, [AgBF4]n) and silver adducts of plant/wax/insect compounds (e.g. [C27H56+Ag]+, [C29H60+Ag]+, [C31H64+Ag]+, [C26H54O+Ag]+, [C28H58O+Ag]+, [C30H62O+Ag]+, [C26H52O2+Ag]+, [C30H60O2+Ag]+). Compile both classes into a single validation table with chemical formula, class, type, and monoisotopic mass, mirroring Table 2 of the paper, and save as validation_list.csv. Validate by checking that each entry's monoisotopic mass matches the reported value (e.g. [Ag]n+ 106.9051, [AgF]n 125.9036, [AgBr]n 185.8229) within the isotopic-calculation tolerance and that the list spans the full mass range of a typical AgLDI dataset.

## Related tools

- **enviPat** (Computes theoretical isotope fine-structure patterns and monoisotopic masses used to populate the validation list)
- **rMSIcleanup** (Downstream consumer of the validation list — classifies observed peaks in AgLDI MSI images against the possible/not-possible silver-related peak definitions) — https://github.com/gbaquer/rMSIcleanup
- **R** (Execution environment for enviPat and rMSIcleanup)

## Examples

```
results<-rMSIcleanup::annotate_matrix(pks,"Ag1",full)
```

## Evaluation signals

- Each entry's computed monoisotopic mass matches the reported Table 2 value (e.g. [Ag]n+ 106.9051, [AgF]n 125.9036, [AgBr]n 185.8229) within the isotopic-calculation tolerance
- Validation list spans the full mass range of a typical AgLDI MSI dataset
- Positive class covers all Agn+ clusters within the acquired mass range; negative class covers the specific salts/adducts enumerated in the paper
- Output table contains chemical formula, class, type, and monoisotopic mass columns for every entry

## Limitations

- The list is defined for a typical AgLDI MSI experiment; it does not generalize to other LDI-promoting materials
- Negative-class entries are clusters/adducts not expected in animal or plant samples; compositions outside this set are not covered

## Evidence

- [intro] Silver isotopic pattern defines the possible/not-possible peak list: "A total of 14 MSI datasets acquired with an Ag-sputtered nanolayer from three different laboratories, were used for validation."
- [methods] Spectral similarity score S1 and intra-cluster morphological similarity S2 as classifiers: [evidence span withheld — anchor status: not_found]
- [methods] enviPat computes theoretical isotope patterns: "are calculated using the open-source package enviPat [23], a fast and memory-efficient algorithm to compute theoretical isotope patterns."
- [readme] Basic usage of rMSIcleanup and data loading: "pks<-rMSIproc::LoadPeakMatrix("[Full Path to Peak Matrix (.zip)]")
full<-rMSI::LoadMsiData("[Full Path to Processed Data (.tar)]")"
