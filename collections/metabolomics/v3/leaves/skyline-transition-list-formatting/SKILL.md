---
name: skyline-transition-list-formatting
description: Use when a targeted LC-MS/MS workflow (e.g. Skyline) needs an in silico
  lipid transition list, and LSG's default '.MSP' spectral-library export does not
  match the acquisition software's required input format.
when_to_use_negative:
- The downstream tool consumes spectral-library files directly — LSG's default '.MSP'
  export is the appropriate choice, not the CSV transition list.
- The workflow targets DDA acquisition on an Orbitrap — the same '.CSV' option should
  instead produce an Excalibur-compatible precursor list, not a Skyline transition
  list.
- Transitions must be derived from empirically measured fragmentation spectra — LSG
  spectra are generated in silico from fragmentation patterns based on peer-reviewed
  studies, not from experimental data.
edam_operation: http://edamontology.org/operation_0335
tools:
- name: Lipid Spectrum Generator
  role: Python tool that generates the in silico lipid library and whose export step,
    when '.CSV' is selected, writes the Skyline-compatible transition list.
  repo: https://github.com/98104781/LSG
- name: Python
  role: Runtime environment in which LSG is executed to generate lipids/spectra and
    perform the export.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.2c04518
    title: 'Lipid Spectrum Generator: A Simple Script for the Generation of Accurate
      In Silico Lipid Fragmentation Spectra'
  source_content_hash: sha256:0df3a6a11351f681773a1e21e48522fb271ad0740b0a0ce093122086711455d5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/98104781/LSG
  tools:
  - Lipid Spectrum Generator
  - Python
  - pandas
  edam_operation: http://edamontology.org/operation_0335
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Skyline transition list formatting

## Summary

Exports an in silico lipid spectral library from the Lipid Spectrum Generator (LSG) as a Skyline-compatible transition list by choosing the '.CSV' output option instead of the default '.MSP' spectral-library file. The same option also yields an Excalibur-compatible precursor list for DDA analysis on an Orbitrap.

## When to use

Use when a targeted LC-MS/MS workflow (e.g. Skyline) needs an in silico lipid transition list, and LSG's default '.MSP' spectral-library export does not match the acquisition software's required input format.

## When NOT to use

- The downstream tool consumes spectral-library files directly — LSG's default '.MSP' export is the appropriate choice, not the CSV transition list.
- The workflow targets DDA acquisition on an Orbitrap — the same '.CSV' option should instead produce an Excalibur-compatible precursor list, not a Skyline transition list.
- Transitions must be derived from empirically measured fragmentation spectra — LSG spectra are generated in silico from fragmentation patterns based on peer-reviewed studies, not from experimental data.

## Inputs

- LSG in silico lipid library (lipid class, fatty-acid composition, and adduct selections)
- Export-format choice ('.CSV' vs '.MSP') in the LSG export dialog

## Outputs

- Skyline-compatible transition list exported as a .CSV file
- (alternative) Excalibur-compatible precursor list for Orbitrap DDA, exported as a .CSV file

## How to apply

Generate lipids and spectra in the LSG Python tool by specifying the lipid class, fatty-acid composition, and adduct; LSG generates lipids with respect to class and fatty-acid composition and spectra with respect to their adducts. Then, at the export step, deliberately select the '.CSV' option rather than the default '.MSP' selection, since '.CSV' switches the output from a spectral-library file to either an Excalibur-compatible precursor list (for DDA analysis via Orbitrap) or a Skyline-compatible transition list. Decide between the two '.CSV' flavors based on the downstream instrument/workflow: Skyline targeted analysis needs the transition list, while Orbitrap DDA acquisition software needs the precursor list. Write the exported transition list to disk as the final artifact and verify the file carries the CSV extension and imports cleanly as a transition list into Skyline.

## Related tools

- **Lipid Spectrum Generator** (Python tool that generates the in silico lipid library and whose export step, when '.CSV' is selected, writes the Skyline-compatible transition list.) — https://github.com/98104781/LSG
- **Python** (Runtime environment in which LSG is executed to generate lipids/spectra and perform the export.)

## Evaluation signals

- The exported artifact exists on disk with a .CSV file extension (not .MSP).
- The file is accepted by Skyline as a transition list when imported for targeted lipid analysis.
- The export corresponds to the requested lipid class, fatty-acid composition, and adduct as configured in LSG.
- If the intended target was Orbitrap DDA, the file instead validates as an Excalibur-compatible precursor list — format flavor matches the acquisition workflow.
- The README release page (v1.3.0) how-to guide is consulted to confirm the correct export selection was used.

## Limitations

- The export format is an either/or choice: selecting '.MSP' yields a spectral library, while '.CSV' yields a transition list or precursor list; a single export produces only one format.
- Fragmentation patterns underlying the spectra are in silico and 'based on peer-reviewed studies', so transition intensities may not reflect instrument-specific behavior.
- The exact column schema of the Skyline-compatible transition list is not specified in the article or README; refer to the v1.3.0 release page how-to guide for details.

## Evidence
