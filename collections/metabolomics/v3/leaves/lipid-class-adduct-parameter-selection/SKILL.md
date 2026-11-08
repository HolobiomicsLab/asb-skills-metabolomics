---
name: lipid-class-adduct-parameter-selection
description: Use when building an in silico lipid spectral library with LSG and you
  must decide which lipid class(es), fatty-acid compositions, and adduct(s) to include
  for LC-MS/MS identification.
when_to_use_negative:
- The input is already an acquired LC-MS/MS dataset needing spectral matching against
  an existing library rather than generation of a new one
- The target lipid class, fatty-acid composition, or adduct is outside what LSG supports
  (its fragmentation patterns are derived from peer-reviewed studies of specific species)
- A spectral library with fragmentation patterns outside LSG's peer-reviewed basis
  is required
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: Lipid Spectrum Generator
  role: Open-source in silico spectral library generator whose user parameters (lipid
    class, fatty-acid composition, adduct) define the generated library; also performs
    the '.MSP' or '.CSV' export
  repo: https://github.com/98104781/LSG
- name: Python
  role: Environment in which the LSG script is run; downstream analysis of generated
    libraries was performed in Python using libraries such as pandas and scikit-learn
- name: pandas
  role: Python library used for data analysis alongside LSG's generated spectral outputs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.2c04518
    title: 'Lipid Spectrum Generator: A Simple Script for the Generation of Accurate
      In Silico Lipid Fragmentation Spectra'
  source_content_hash: sha256:7ec8675a7b11990c11257b5b405fe9fe34553732b0d93b10a25db97c10338f06
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/98104781/LSG
  tools:
  - Lipid Spectrum Generator
  - Python
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# lipid-class-adduct-parameter-selection

## Summary

Guides selection of lipid class, fatty-acid composition, and adduct parameters in the Lipid Spectrum Generator (LSG) so that in silico fragmentation spectra match the target analytes of an LC-MS/MS experiment. LSG builds lipids per class and fatty-acid composition, then generates spectra per adduct, so these choices determine the library's coverage.

## When to use

Use when building an in silico lipid spectral library with LSG and you must decide which lipid class(es), fatty-acid compositions, and adduct(s) to include for LC-MS/MS identification.

## When NOT to use

- The input is already an acquired LC-MS/MS dataset needing spectral matching against an existing library rather than generation of a new one
- The target lipid class, fatty-acid composition, or adduct is outside what LSG supports (its fragmentation patterns are derived from peer-reviewed studies of specific species)
- A spectral library with fragmentation patterns outside LSG's peer-reviewed basis is required

## Inputs

- Target lipid class(es) (e.g., chosen by the user for the sample under study)
- Fatty-acid composition specification for the chosen class
- Adduct selection (ion species expected in the LC-MS/MS run)

## Outputs

- In silico (mass)spectral library of generated lipid species and their fragmentation spectra
- Exported library file with '.MSP' extension (spectral library), or '.CSV' (Excalibur-compatible precursor list or Skyline-compatible transition list)

## How to apply

In LSG, lipids are generated with respect to class and fatty-acid composition, and spectra are then generated with respect to their adducts, so parameter selection proceeds in that order: first choose the lipid class and fatty-acid composition that match the biological sample, then choose the adduct(s) expected under your ionization conditions. The fragmentation patterns used to generate spectra are based on peer-reviewed studies, so selections should be constrained to classes/adducts the tool supports. After generation, export the library with the '.MSP' extension for spectral-library use, or select '.CSV' if an Excalibur-compatible precursor list (for DDA analysis via orbitrap) or a Skyline-compatible transition list is needed. An overview of the features and a brief how-to guide is available on the v1.3.0 release page (https://github.com/98104781/LSG/releases/tag/v1.3.0).

## Related tools

- **Lipid Spectrum Generator** (Open-source in silico spectral library generator whose user parameters (lipid class, fatty-acid composition, adduct) define the generated library; also performs the '.MSP' or '.CSV' export) — https://github.com/98104781/LSG
- **Python** (Environment in which the LSG script is run; downstream analysis of generated libraries was performed in Python using libraries such as pandas and scikit-learn)
- **pandas** (Python library used for data analysis alongside LSG's generated spectral outputs)

## Evaluation signals

- The generated library contains lipids only for the selected class(es) and fatty-acid composition(s) and spectra only for the selected adduct(s)
- Exported file exists with the expected extension: '.MSP' for a spectral library, '.CSV' for a Skyline-compatible transition list or Excalibur-compatible precursor list
- A '.CSV' export is loadable as a Skyline transition list (or Excalibur precursor list for orbitrap DDA), confirming format compatibility
- Fragmentation patterns in the generated spectra trace back to the peer-reviewed studies underlying LSG
- Repository version used is the documented release (v1.3.0) whose release page documents the features used

## Limitations

- Spectral accuracy depends on fragmentation patterns derived from peer-reviewed studies; unsupported classes/adducts cannot be generated
- Existing in silico lipid spectral libraries (the problem LSG addresses) can be limited in usability or capacity to represent lipid species, so coverage must be checked for your target class
- Selection of a wrong adduct yields spectra for ion species not observed in the LC-MS/MS run, wasting the export

## Evidence
