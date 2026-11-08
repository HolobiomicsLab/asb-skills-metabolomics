---
name: adduct-based-fragmentation-spectra-simulation
description: Use when building a reference spectral library for lipid identification
  in LC-MS/MS and you need in silico MS/MS spectra for lipid species enumerated by
  class and fatty acid composition across defined adducts.
when_to_use_negative:
- When an experimentally acquired (measured) spectral library already covers the target
  lipid classes and adducts, since LSG produces in silico rather than empirical spectra.
- When the target analytes are not lipids — the generator enumerates lipid species
  by class and fatty acid composition and is not a general small-molecule spectra
  predictor.
- When the needed export format is neither '.MSP' nor the '.CSV' precursor/transition
  list formats supported by LSG.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Lipid Spectrum Generator
  role: Core script that enumerates lipids by class and fatty acid composition, generates
    fragmentation spectra per adduct, and exports .MSP or .CSV libraries
  repo: https://github.com/98104781/LSG
- name: Python
  role: Runtime environment in which the Lipid Spectrum Generator script is executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.2c04518
    title: 'Lipid Spectrum Generator: A Simple Script for the Generation of Accurate
      In Silico Lipid Fragmentation Spectra'
  source_content_hash: sha256:2986a2c8b9487ffead06f746b21101647d3b0f01accd7bab8fb62e65726bfb12
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/98104781/LSG
  tools:
  - Lipid Spectrum Generator
  - Python
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# adduct-based-fragmentation-spectra-simulation

## Summary

Generates in silico lipid fragmentation spectra and on-demand (mass)spectral libraries using the open-source Lipid Spectrum Generator (LSG), which enumerates lipids by lipid class and fatty acid composition and then computes spectra for each lipid with respect to its adducts.

## When to use

Use when building a reference spectral library for lipid identification in LC-MS/MS and you need in silico MS/MS spectra for lipid species enumerated by class and fatty acid composition across defined adducts.

## When NOT to use

- When an experimentally acquired (measured) spectral library already covers the target lipid classes and adducts, since LSG produces in silico rather than empirical spectra.
- When the target analytes are not lipids — the generator enumerates lipid species by class and fatty acid composition and is not a general small-molecule spectra predictor.
- When the needed export format is neither '.MSP' nor the '.CSV' precursor/transition list formats supported by LSG.

## Inputs

- Built-in lipid classes of the Lipid Spectrum Generator (lipid class definitions)
- Fatty acid composition space to enumerate lipid species
- Adduct set to simulate per lipid species

## Outputs

- Spectral library file with the '.MSP' extension containing spectra for enumerated lipids by class, fatty acid composition, and adduct
- Optional Excalibur-compatible precursor list (.CSV) for Orbitrap DDA analysis
- Optional Skyline-compatible transition list (.CSV)

## How to apply

Obtain the Lipid Spectrum Generator (v1.3.0) from its GitHub release (https://github.com/98104781/LSG/releases/tag/v1.3.0). Invoke the generator over its built-in lipid classes so that lipids are enumerated with respect to class and fatty acid composition. For each enumerated lipid, generate fragmentation spectra with respect to its adducts; the fragmentation patterns used are based on peer-reviewed studies. Export the accumulated spectra as a spectral library by selecting the '.MSP' file extension; alternatively export an Excalibur-compatible precursor list (for DDA analysis on an Orbitrap) or a Skyline-compatible transition list by selecting '.CSV'. Validate the exported .MSP artifact by confirming it contains entries for all enumerated lipid classes and their adducts.

## Related tools

- **Lipid Spectrum Generator** (Core script that enumerates lipids by class and fatty acid composition, generates fragmentation spectra per adduct, and exports .MSP or .CSV libraries) — https://github.com/98104781/LSG
- **Python** (Runtime environment in which the Lipid Spectrum Generator script is executed)

## Examples

```
python LSG.py --lipid-classes all --adducts all --export .MSP
```

## Evaluation signals

- The exported .MSP file contains entries for every enumerated lipid class and adduct (per the task_001 workflow validation step).
- Spectra reflect fragmentation patterns consistent with peer-reviewed fragmentation studies, as the README states the patterns are based on peer-reviewed studies.
- Spot-check precursor/fragment masses in the .MSP against expected m/z values for known lipid species and adducts.
- If a .CSV precursor list for Orbitrap DDA was exported, confirm it is Excalibur-compatible; if a transition list was exported, confirm it is Skyline-compatible.
- Number of spectra entries matches the lipid count implied by the class × fatty-acid-composition × adduct enumeration.

## Limitations

- Spectra are in silico and depend on the fragmentation patterns encoded from peer-reviewed studies; accuracy is bounded by those patterns, not by instrument-specific behavior.
- The paper's discussion section notes missing signals such as no public repository or code release mentioned in the discussion, exact hyperparameters not specified, and statistical test details omitted — so downstream statistical claims around validation (e.g., TCGA cohort) lack documented tuning and test details in that section.
- Export formats are limited to '.MSP' spectral libraries or '.CSV' precursor/transition lists.

## Evidence

- [intro] Lipid generation loop: "Lipids are generated with respect to class and fatty acid composition, spectra are then generated with respect to their adducts"
- [intro] MSP export: "Spectral libraries can be exported with the file extension '.MSP' selected"
- [intro] CSV export options: "an Excalibur compatible precursor list (for DDA analysis via orbitrap) or Skylike compatible transition list may be exported by selecting '.CSV'"
- [abstract] Purpose of the tool: "an open-source in silico spectral library generator specifically designed to aid in the identification of lipids in liquid chromatography-tandem mass spectrometry analysis"
- [readme] README purpose statement: "This program is intended to enable the user to generate on-demand (mass)spectral libraries for the identification of lipid species."
- [readme] Fragmentation pattern basis: "The fragmentation patterns used to generate the spectra are based on peer-reviewed studies."
