---
name: csv-output-formatting
description: Use when a generated lipid spectral library must feed instrument acquisition
  or targeted-analysis software — i.e. you need an Excalibur precursor list for Orbitrap
  DDA or a Skyline transition list rather than a '.MSP' library.
when_to_use_negative:
- A searchable in silico spectral library is the goal — export with the '.MSP' file
  extension instead of '.CSV'
- The downstream instrument workflow is neither Orbitrap DDA acquisition (Excalibur)
  nor Skyline targeted analysis, so neither '.CSV' export variant applies
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: Lipid Spectrum Generator
  role: Generates the on-demand lipid (mass)spectral library and performs the '.CSV'
    export producing the Excalibur-compatible precursor list or Skyline-compatible
    transition list
  repo: https://github.com/98104781/LSG
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.2c04518
    title: 'Lipid Spectrum Generator: A Simple Script for the Generation of Accurate
      In Silico Lipid Fragmentation Spectra'
  source_content_hash: sha256:af6511c04d2cb7220cd8237075167148a5e580335170d81498ae3a1183b6a4bd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/98104781/LSG
  tools:
  - Lipid Spectrum Generator
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.631578947368421
n_claims: 19
---

# csv-output-formatting

## Summary

Export Lipid Spectrum Generator output as a '.CSV' file containing either an Excalibur-compatible precursor list for DDA analysis on an Orbitrap or a Skyline-compatible transition list, instead of the default '.MSP' spectral library.

## When to use

Use when a generated lipid spectral library must feed instrument acquisition or targeted-analysis software — i.e. you need an Excalibur precursor list for Orbitrap DDA or a Skyline transition list rather than a '.MSP' library.

## When NOT to use

- A searchable in silico spectral library is the goal — export with the '.MSP' file extension instead of '.CSV'
- The downstream instrument workflow is neither Orbitrap DDA acquisition (Excalibur) nor Skyline targeted analysis, so neither '.CSV' export variant applies

## Inputs

- Generated lipid/precursor library output from a Lipid Spectrum Generator run (lipids by class and fatty acid composition, spectra by adduct)
- User selection of entries intended for DDA precursor targeting

## Outputs

- Excalibur-compatible precursor list (.CSV) for DDA analysis via Orbitrap
- Skyline-compatible transition list (.CSV)
- Exported comma-separated table, e.g. precursor_list.csv

## How to apply

Run the Lipid Spectrum Generator to build lipids by class and fatty acid composition and spectra by adduct, then use the tool's export mechanism with the '.CSV' file extension selected (rather than '.MSP') to emit an Excalibur-compatible precursor list (for DDA analysis via Orbitrap) or a Skyline-compatible transition list. Write the result to a comma-separated file such as precursor_list.csv following the header/field layout implied by the Excalibur precursor-list convention. The decision point is the downstream consumer: Orbitrap DDA acquisition calls for the Excalibur precursor list, whereas targeted monitoring in Skyline calls for the transition-list variant of the same '.CSV' export.

## Related tools

- **Lipid Spectrum Generator** (Generates the on-demand lipid (mass)spectral library and performs the '.CSV' export producing the Excalibur-compatible precursor list or Skyline-compatible transition list) — https://github.com/98104781/LSG

## Evaluation signals

- The exported file exists and carries the .CSV extension
- The file parses as a comma-separated precursor/transition table (file_format_is / contains_substring checks)
- Exported entries match the lipid/precursor entries selected for DDA targeting (class, fatty acid composition, adduct-derived masses)
- Content-level correctness of the precursor entries is confirmed by expert_review against the Excalibur precursor-list convention

## Limitations

- The README provides no field-level schema for the '.CSV' export; the required header/field layout is only implied by the Excalibur precursor-list convention
- Content correctness of the precursor entries is not validated by automated file checks and must be assessed by expert_review
- The README typo 'Skylike compatible transition list' indicates documentation is informal; confirm the Skyline variant's exact fields empirically against the v1.3.0 release notes

## Evidence

- [intro] An Excalibur compatible precursor list (for DDA analysis via orbitrap) or Skylike compatible transition list may be exported by selecting '.CSV': "an Excalibur compatible precursor list (for DDA analysis via orbitrap) or Skylike compatible transition list may be exported by selecting '.CSV'"
- [readme] Spectral libraries can be exported with the file extension '.MSP' selected: "Spectral libraries can be exported with the file extension '.MSP' selected."
