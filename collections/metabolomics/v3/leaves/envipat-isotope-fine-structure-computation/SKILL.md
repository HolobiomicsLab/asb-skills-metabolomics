---
name: envipat-isotope-fine-structure-computation
description: Use when building a validation/reference peak list for metal-assisted
  (e.g. Ag-sputtered nanolayer) LDI MSI, where expected Ag-cluster and Ag-adduct masses
  must be derived from isotope abundances before peak classification.
when_to_use_negative:
- The aim is classifying peaks in datasets acquired with conventional organic MALDI
  matrices rather than a silver nanolayer — the possible/not-possible peak list is
  specific to AgLDI
- The input is already a classified peak matrix or a finished validation list (the
  computation would be redundant)
- The LDI-promoting material is an element without a distinctive multi-isotope pattern
  usable for peak-list construction
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3314
tools:
- name: enviPat
  role: Computes theoretical isotope patterns and fine-structure masses for the silver
    clusters and silver adducts in the validation list
- name: rMSIcleanup
  role: 'Downstream consumer: validates its m/z peak classification against the possible/not-possible
    silver-peak list built with enviPat'
  repo: https://github.com/gbaquer/rMSIcleanup
- name: R
  role: Language in which enviPat and rMSIcleanup are run to generate the validation
    list
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-020-00449-0
    title: 'rMSIcleanup: an open-source tool for matrix-related peak annotation in
      mass spectrometry imaging and its application to silver-assisted laser desorption/ionization'
  source_content_hash: sha256:909ee7836cea0b3de2e10cad6f8655966371377f8c1d127f8e95300074825788
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gbaquer/rMSIcleanup
  tools:
  - R
  - enviPat
  - rMSIcleanup
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# envipat-isotope-fine-structure-computation

## Summary

Compute theoretical monoisotopic masses and isotopic fine-structure patterns with the enviPat R package to build the silver-cluster validation list (possible vs not-possible Ag-related peaks) used by rMSIcleanup to classify matrix-related signals in AgLDI MSI experiments.

## When to use

Use when building a validation/reference peak list for metal-assisted (e.g. Ag-sputtered nanolayer) LDI MSI, where expected Ag-cluster and Ag-adduct masses must be derived from isotope abundances before peak classification.

## When NOT to use

- The aim is classifying peaks in datasets acquired with conventional organic MALDI matrices rather than a silver nanolayer — the possible/not-possible peak list is specific to AgLDI
- The input is already a classified peak matrix or a finished validation list (the computation would be redundant)
- The LDI-promoting material is an element without a distinctive multi-isotope pattern usable for peak-list construction

## Inputs

- Element isotopic abundances for Ag (107Ag 51.84%, 109Ag 48.16%) from the methods section
- List of positive-class silver cluster formulas Ag_n+ (n=1..10)
- List of negative-class formulas: silver salts/synthetics ([AgF]n, [AgCl]n, [AgBr]n, [AgI]n, [AgH]n, [AgH2]n, [AgHe]n, [AgNO3]n, [AgTh2]n, [AgF2]n, [AgBF4]n) and Ag adducts ([C27H56+Ag]+, [C29H60+Ag]+, [C31H64+Ag]+, [C26H54O+Ag]+, [C28H58O+Ag]+, [C30H62O+Ag]+, [C26H52O2+Ag]+, [C30H60O2+Ag]+)
- Mass range of the typical AgLDI MSI acquisition

## Outputs

- validation_list.csv table with columns: chemical formula, class (possible/not-possible), type, monoisotopic mass, mirroring Table 2
- Theoretical isotopic pattern intensities for each entry

## How to apply

Load the silver isotopic abundances from the methods (107Ag 51.84%, 109Ag 48.16%) and use enviPat's isotope fine-structure routines to compute theoretical monoisotopic masses and isotopic pattern intensities for positive-class silver clusters Ag_n+ (n from 1 up to the acquired mass-range limit, e.g. n=1 to n=10). Compute the same theoretical masses/patterns for the negative class: silver salts and synthetic compounds ([AgF]n, [AgCl]n, [AgBr]n, [AgI]n, [AgH]n, [AgH2]n, [AgHe]n, [AgNO3]n, [AgTh2]n, [AgF2]n, [AgBF4]n) and Ag adducts of plant/wax/insect compounds such as [C27H56+Ag]+, [C29H60+Ag]+ and [C30H60O2+Ag]+. Compile chemical formula, class, type and monoisotopic mass for each entry into a validation list mirroring Table 2 of the paper and save it as validation_list.csv. The rationale: the characteristic isotopic pattern of silver is what allows a list of possible and not-possible silver-related peaks to be defined as ground truth for validating the peak classification algorithm.

## Related tools

- **enviPat** (Computes theoretical isotope patterns and fine-structure masses for the silver clusters and silver adducts in the validation list)
- **rMSIcleanup** (Downstream consumer: validates its m/z peak classification against the possible/not-possible silver-peak list built with enviPat) — https://github.com/gbaquer/rMSIcleanup
- **R** (Language in which enviPat and rMSIcleanup are run to generate the validation list)

## Examples

```
install.packages("enviPat"); library(enviPat); data(isotopes); pattern <- isopattern(isotopes, c("Ag1","Ag2","AgF"), threshold=0.1, plot=FALSE)
```

## Evaluation signals

- Each entry's monoisotopic mass matches the reported Table 2 value (e.g. [Ag]n+ 106.9051, [AgF]n 125.9036, [AgBr]n 185.8229) within isotopic-calculation tolerance
- Isotopic pattern intensities reflect the silver abundances 107Ag 51.84% and 109Ag 48.16% (near-equal doublets separated by ~2 Da)
- The compiled list spans the full mass range of a typical AgLDI dataset
- The list contains both positive class (all silver clusters in range) and negative class (clusters/adducts not expected in animal or plant samples) with formula, class, type and mass columns

## Limitations

- The list is defined for a typical AgLDI MSI experiment; entries in the negative class are compounds not expected in animal or plant samples and may not transfer to other sample types
- Cluster enumeration is capped by the mass-range limit (e.g. n=1 to n=10 for Ag_n+), so clusters beyond the acquired range are not covered
- enviPat computes theoretical patterns only; experimental agreement still depends on prior mass re-calibration with Ag reference peaks

## Evidence

- [methods] are calculated using the open-source package enviPat [23], a fast and memory-efficient algorithm to compute theoretical isotope patterns.: "are calculated using the open-source package enviPat [23], a fast and memory-efficient algorithm to compute theoretical isotope patterns."
- [intro] 14 MSI datasets ... exploiting the characteristic isotopic pattern of silver to define a list of possible and not-possible silver-related peaks as a validation list.: [evidence span withheld — anchor status: not_found]
- [abstract] The algorithm was able to correctly classify m/z signals related to silver clusters.: "The algorithm was able to correctly classify m/z signals related to silver clusters."
