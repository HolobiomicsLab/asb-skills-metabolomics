---
name: lipid-class-and-fatty-acid-enumeration
description: Use when an LC-MS/MS lipid identification workflow needs a custom, on-demand
  in silico spectral library covering specified lipid classes, fatty acid compositions,
  and adducts, rather than a fixed pre-built library.
when_to_use_negative:
- When a suitable pre-built lipid spectral library already covers the classes, fatty
  acid compositions, and adducts of interest — on-demand generation is intended for
  cases where existing in silico libraries are limited in usability or capacity to
  represent lipid species.
- When the goal is downstream statistical modeling or data analysis of already-acquired
  lipidomics data (the paper used pandas and scikit-learn for that) rather than library
  construction.
- When identification of non-lipid small molecules is the target — LSG is specifically
  designed for lipid species identification in LC-MS/MS.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Lipid Spectrum Generator
  role: 'Primary generator: enumerates lipids by class and fatty acid composition,
    generates adduct-specific in silico fragmentation spectra, and exports spectral
    libraries (.MSP) or precursor/transition lists (.CSV)'
  repo: https://github.com/98104781/LSG
- name: Python
  role: Runtime environment for the LSG script and for downstream validation/data
    analysis of the generated library (e.g., with pandas)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.2c04518
    title: 'Lipid Spectrum Generator: A Simple Script for the Generation of Accurate
      In Silico Lipid Fragmentation Spectra'
  source_content_hash: sha256:b7c24f4bc3bee19c20c50e1f62141be5ece24d3f7e7068260bf995264e0b086e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/98104781/LSG
  tools:
  - Lipid Spectrum Generator
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# lipid-class-and-fatty-acid-enumeration

## Summary

Systematic enumeration of in silico lipid species by lipid class and fatty acid composition using the open-source Lipid Spectrum Generator (LSG, v1.3.0), followed by adduct-specific fragmentation spectrum generation for building on-demand lipid spectral libraries.

## When to use

Use when an LC-MS/MS lipid identification workflow needs a custom, on-demand in silico spectral library covering specified lipid classes, fatty acid compositions, and adducts, rather than a fixed pre-built library.

## When NOT to use

- When a suitable pre-built lipid spectral library already covers the classes, fatty acid compositions, and adducts of interest — on-demand generation is intended for cases where existing in silico libraries are limited in usability or capacity to represent lipid species.
- When the goal is downstream statistical modeling or data analysis of already-acquired lipidomics data (the paper used pandas and scikit-learn for that) rather than library construction.
- When identification of non-lipid small molecules is the target — LSG is specifically designed for lipid species identification in LC-MS/MS.

## Inputs

- Selection of target lipid classes (from the generator's built-in lipid classes)
- Fatty acid composition specification per lipid class
- Target adduct list per lipid species
- Choice of output format ('.MSP' or '.CSV')

## Outputs

- Spectral library file with the '.MSP' extension containing per-lipid, per-adduct fragmentation spectra
- Optionally, an Excalibur-compatible precursor list for DDA analysis via Orbitrap, or a Skyline-compatible transition list, exported as '.CSV'

## How to apply

Obtain the Lipid Spectrum Generator (v1.3.0) from its GitHub release page. Invoke the generator over its built-in lipid classes so that lipids are first enumerated with respect to class and fatty acid composition; the fragmentation patterns used to build the spectra are based on peer-reviewed studies. For each enumerated lipid, generate fragmentation spectra with respect to its adducts. Decide the export format at the end: choose '.MSP' to export a spectral library, or '.CSV' for an Excalibur-compatible precursor list (DDA analysis via Orbitrap) or a Skyline-compatible transition list. Validate the exported artifact by confirming it contains entries for all enumerated lipid classes and adducts.

## Related tools

- **Lipid Spectrum Generator** (Primary generator: enumerates lipids by class and fatty acid composition, generates adduct-specific in silico fragmentation spectra, and exports spectral libraries (.MSP) or precursor/transition lists (.CSV)) — https://github.com/98104781/LSG
- **Python** (Runtime environment for the LSG script and for downstream validation/data analysis of the generated library (e.g., with pandas))

## Evaluation signals

- The exported '.MSP' file exists and parses as a spectral library artifact.
- Every lipid class and fatty acid composition selected at enumeration time is represented by at least one library entry.
- Every adduct specified for an enumerated lipid has a corresponding generated spectrum entry.
- Entries match the expected lipid-class naming and fatty-acid composition scheme produced by LSG's generation loop.
- If a '.CSV' export was chosen, the file is compatible as an Excalibur precursor list (Orbitrap DDA) or a Skyline transition list.

## Limitations

- Spectral quality depends on the peer-reviewed fragmentation patterns LSG is built on; accuracy is bounded by those studies.
- Coverage is restricted to the generator's built-in lipid classes and their fatty acid compositions.
- The article's discussion does not specify model hyperparameters, statistical tests, or a public code release beyond the GitHub release, so reproducibility details beyond v1.3.0 may be incomplete.

## Evidence
