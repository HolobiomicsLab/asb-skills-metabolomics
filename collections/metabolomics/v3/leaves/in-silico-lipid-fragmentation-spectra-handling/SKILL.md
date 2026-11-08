---
name: in-silico-lipid-fragmentation-spectra-handling
description: Use when building a targeted LC-MS/MS lipid identification workflow that
  needs a spectral library of lipid species (defined by class, fatty acid composition,
  and adduct) exported as '.MSP', or an accompanying precursor/transition list.
when_to_use_negative:
- You already possess an experimental or vendor lipid spectral library and only need
  format conversion rather than in silico spectrum generation
- You need spectra for non-lipid analytes (e.g., metabolites or peptides) — LSG enumerates
  lipids by class and fatty acid composition only
- You need precursor/transition lists for instruments/workflows other than Orbitrap
  DDA via Excalibur or Skyline transition lists
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Lipid Spectrum Generator
  role: Open-source in silico spectral library generator that enumerates lipids by
    class and fatty acid composition, builds spectra per adduct, and exports '.MSP'
    or '.CSV' outputs
  repo: https://github.com/98104781/LSG
- name: Python
  role: Language in which the LSG script is implemented and run to produce the spectral
    library exports
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.2c04518
    title: 'Lipid Spectrum Generator: A Simple Script for the Generation of Accurate
      In Silico Lipid Fragmentation Spectra'
  source_content_hash: sha256:f9ecb6d9503bffb1e739575eeaef2e104e80a63feb0b5700f13fdfb0b9edeb8f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/98104781/LSG
  tools:
  - Lipid Spectrum Generator
  - Python
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# in-silico-lipid-fragmentation-spectra-handling

## Summary

Generates on-demand in silico lipid (mass)spectral libraries with the open-source Lipid Spectrum Generator (LSG): lipids are enumerated by lipid class and fatty acid composition, fragmentation spectra are built per adduct from peer-reviewed fragmentation patterns, and libraries are exported as '.MSP' (or '.CSV' precursor/transition lists).

## When to use

Use when building a targeted LC-MS/MS lipid identification workflow that needs a spectral library of lipid species (defined by class, fatty acid composition, and adduct) exported as '.MSP', or an accompanying precursor/transition list.

## When NOT to use

- You already possess an experimental or vendor lipid spectral library and only need format conversion rather than in silico spectrum generation
- You need spectra for non-lipid analytes (e.g., metabolites or peptides) — LSG enumerates lipids by class and fatty acid composition only
- You need precursor/transition lists for instruments/workflows other than Orbitrap DDA via Excalibur or Skyline transition lists

## Inputs

- Lipid scope definition (lipid classes and fatty acid compositions to enumerate)
- Adduct set for which spectra should be generated
- Choice of export extension ('.MSP' for spectral library; '.CSV' for precursor/transition list)

## Outputs

- Spectral library file with the '.MSP' extension containing the generated lipid fragmentation spectra
- Optionally, an Excalibur-compatible precursor list (DDA, Orbitrap) or Skyline-compatible transition list as '.CSV'

## How to apply

Run the Lipid Spectrum Generator script (98104781/LSG, v1.3.0) to enumerate lipids with respect to lipid class and fatty acid composition, then generate each spectrum with respect to its adduct; the fragmentation patterns are drawn from peer-reviewed studies, so no training data are required. Decide the output format by extension: select '.MSP' to export the assembled spectral library in the MSP convention consumed by LC-MS/MS search engines, or select '.CSV' to export an Excalibur-compatible precursor list (for DDA analysis on an Orbitrap) or a Skyline-compatible transition list. Validate the '.MSP' file contains one formatted entry per generated lipid-class/fatty-acid/adduct combination before feeding it into downstream search or transition-list workflows.

## Related tools

- **Lipid Spectrum Generator** (Open-source in silico spectral library generator that enumerates lipids by class and fatty acid composition, builds spectra per adduct, and exports '.MSP' or '.CSV' outputs) — https://github.com/98104781/LSG
- **Python** (Language in which the LSG script is implemented and run to produce the spectral library exports)

## Examples

```
python LSG.py --lipid-classes <classes> --fatty-acids <compositions> --adducts <adducts> --export .MSP
```

## Evaluation signals

- Exported file carries the '.MSP' extension and parses as a valid MSP spectral library for LC-MS/MS search engines
- Each spectrum entry corresponds to a lipid generated with respect to its class, fatty acid composition, and adduct
- '.CSV' export (if requested) is accepted as an Excalibur precursor list or Skyline transition list without reformatting
- Number of spectra in the library matches the expected lipid-class × fatty-acid-composition × adduct enumeration
- Fragmentation patterns in the output are consistent with the peer-reviewed fragmentation studies the generator is based on

## Limitations

- Spectra are in silico and depend on peer-reviewed fragmentation patterns rather than measured reference spectra, so coverage and accuracy are bounded by those studies
- Alternative '.CSV' exports target only Excalibur (DDA via Orbitrap) precursor lists or Skyline transition lists; other instrument software may need reformatting
- A README typo ('Skylike compatible transition list') indicates limited documentation polish; feature overview must be checked on the v1.3.0 release page

## Evidence
