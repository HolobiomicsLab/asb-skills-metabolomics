---
name: lipid-spectral-library-generation
description: Use when constructing a reference spectral library for lipid identification
  in liquid chromatography-tandem mass spectrometry (LC-MS/MS) and experimental library
  entries are missing or need custom lipid classes/adducts.
when_to_use_negative:
- An experimental, curated spectral library already covers the lipid classes and adducts
  of interest — enumerating additional in silico species is unnecessary and may introduce
  false identifications.
- The analysis goal is small-molecule (non-lipid) metabolite identification; LSG's
  fragmentation patterns are built specifically for lipid classes and their fatty
  acid compositions.
- Downstream software cannot read '.MSP' or '.CSV' precursor/transition lists — verify
  import compatibility before generating a large library.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3520
tools:
- name: Lipid Spectrum Generator
  role: Core tool that enumerates lipids by class and fatty acid composition, generates
    fragmentation spectra per adduct, and exports spectral libraries as .MSP or precursor/transition
    lists as .CSV
  repo: https://github.com/98104781/LSG
- name: Python
  role: Language in which the generator script is implemented and run
- name: pandas
  role: Data handling library used for data analysis in the underlying work
- name: scikit-learn
  role: Library referenced for downstream data analysis associated with the method
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.2c04518
    title: 'Lipid Spectrum Generator: A Simple Script for the Generation of Accurate
      In Silico Lipid Fragmentation Spectra'
  source_content_hash: sha256:84f045e2d3fa0c25250b569c109c7cc0134d95e7c30406f8b3098d4a7db0291a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/98104781/LSG
  tools:
  - Lipid Spectrum Generator
  - Python
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# lipid-spectral-library-generation

## Summary

Generate on-demand in silico lipid (mass)spectral libraries using the open-source Lipid Spectrum Generator (LSG), which enumerates lipid species by lipid class and fatty acid composition and then computes fragmentation spectra per lipid adduct, exporting them as .MSP libraries.

## When to use

Use when constructing a reference spectral library for lipid identification in liquid chromatography-tandem mass spectrometry (LC-MS/MS) and experimental library entries are missing or need custom lipid classes/adducts.

## When NOT to use

- An experimental, curated spectral library already covers the lipid classes and adducts of interest — enumerating additional in silico species is unnecessary and may introduce false identifications.
- The analysis goal is small-molecule (non-lipid) metabolite identification; LSG's fragmentation patterns are built specifically for lipid classes and their fatty acid compositions.
- Downstream software cannot read '.MSP' or '.CSV' precursor/transition lists — verify import compatibility before generating a large library.

## Inputs

- Lipid class and fatty acid composition definitions (built into LSG)
- Lipid adduct specifications for each enumerated lipid species
- Fragmentation pattern parameters derived from peer-reviewed studies

## Outputs

- In silico spectral library file with the '.MSP' extension containing lipid species spectra
- Optional Excalibur-compatible precursor list (Orbitrap DDA) exported as '.CSV'
- Optional Skyline-compatible transition list exported as '.CSV'

## How to apply

Obtain Lipid Spectrum Generator v1.3.0 from its GitHub release (https://github.com/98104781/LSG/releases/tag/v1.3.0); the release page documents the feature overview and how-to guide. Configure the generator over its built-in lipid classes, enumerating lipid species by class and fatty acid composition, then generate fragmentation spectra for each lipid with respect to its adducts — fragmentation patterns used are based on peer-reviewed studies. Export the accumulated spectra by selecting the '.MSP' file extension to produce a spectral library file; alternatively choose '.CSV' to emit an Excalibur-compatible precursor list (for DDA analysis via Orbitrap) or a Skyline-compatible transition list. Validate the exported library artifact by checking that entries exist for the enumerated lipid classes and their adducts before use in identification.

## Related tools

- **Lipid Spectrum Generator** (Core tool that enumerates lipids by class and fatty acid composition, generates fragmentation spectra per adduct, and exports spectral libraries as .MSP or precursor/transition lists as .CSV) — https://github.com/98104781/LSG
- **Python** (Language in which the generator script is implemented and run)
- **pandas** (Data handling library used for data analysis in the underlying work)
- **scikit-learn** (Library referenced for downstream data analysis associated with the method)

## Examples

```
python LSG.py --classes all --adducts all --export .MSP --output lipid_library.MSP
```

## Evaluation signals

- The exported artifact carries the '.MSP' extension (or '.CSV' for precursor/transition lists), matching the documented output types.
- Library entries exist for every enumerated lipid class and each lipid's specified adducts.
- Generated spectra reflect the fragmentation patterns derived from peer-reviewed studies rather than arbitrary guesses.
- Each lipid species is traceable to its class and fatty acid composition, and each spectrum to its adduct.
- When a '.CSV' export is chosen, the file parses as an Excalibur-compatible precursor list (for Orbitrap DDA) or a Skyline-compatible transition list.

## Limitations

- Fragmentation accuracy depends on the quality of the underlying peer-reviewed fragmentation patterns; gaps in literature coverage for a lipid class or adduct will limit library accuracy.
- Built-in lipid classes and fatty acid compositions constrain what can be enumerated — species outside the generator's class/adduct space cannot be produced.
- The '.CSV' export is format-specific: Excalibur compatibility targets Orbitrap DDA workflows and Skyline compatibility targets transition lists; other instrument platforms are not covered by the documented exports.

## Evidence
