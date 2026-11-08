---
name: adduct-mass-correction
description: Use when preparing raw tandem MS/MS spectra (from NIST20/23 SDF exports
  or MassSpecGym TSVs) for ICEBERG/SCARF training or evaluation, where peaks must
  be expressed as neutral fragment masses.
when_to_use_negative:
- When the downstream model or analysis operates directly on ion m/z values (e.g.
  comparing against an ion-mode spectral library) rather than neutral fragment masses
- When the adduct annotation is missing or unreliable for the spectrum, since subtracting
  an incorrect adduct mass systematically shifts all peaks
- When spectra are already stored as neutral fragment masses (e.g. pre-processed spec_datasets
  files that have already gone through the pipeline)
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ms-pred data scripts
  role: Implements the spectrum normalization pipeline (scan merging, peak merging,
    adduct subtraction, sqrt-intensity, top-50 peak selection) that contains this
    adduct correction step
  repo: https://github.com/samgoldman97/ms-pred
- name: ms-data-parser
  role: Parses raw NIST .SDF spectra into processed_data before normalization, preserving
    adduct metadata needed for mass correction
  repo: https://github.com/rogerwwww/ms-data-parser
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c04654
    title: Generating Molecular Fragmentation Graphs with Autoregressive Neural Networks
  source_content_hash: sha256:bbbbc83c2b8c13672383d3f7e445f2ea17f971658600e4475c441e65fc69b0f1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samgoldman97/ms-pred
  tools:
  - Python
  - ms-data-parser
  - ms-pred data scripts
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# adduct-mass-correction

## Summary

Subtract the adduct-ion mass from each MS2 peak (and the precursor m/z) during tandem MS/MS preprocessing so that peak m/z values reflect neutral fragment masses, aligning spectra with the ICEBERG normalization pipeline used to build its training data.

## When to use

Use when preparing raw tandem MS/MS spectra (from NIST20/23 SDF exports or MassSpecGym TSVs) for ICEBERG/SCARF training or evaluation, where peaks must be expressed as neutral fragment masses.

## When NOT to use

- When the downstream model or analysis operates directly on ion m/z values (e.g. comparing against an ion-mode spectral library) rather than neutral fragment masses
- When the adduct annotation is missing or unreliable for the spectrum, since subtracting an incorrect adduct mass systematically shifts all peaks
- When spectra are already stored as neutral fragment masses (e.g. pre-processed spec_datasets files that have already gone through the pipeline)

## Inputs

- Processed tandem MS/MS spectra with peaks (mz, intensity) and adduct annotation (e.g. from labels.tsv or the entry metadata of spec_files.hdf5)
- Adduct-to-mass mapping for the ion species present in the dataset (proton, sodium, deprotonation, etc.)

## Outputs

- Normalized spectrum entries whose mz values are adduct-corrected (neutral fragment masses), each with at most 50 peaks and square-rooted intensities in [0,1]

## How to apply

During the spectrum normalization workflow in the ms-pred repository, after merging scans across collision energies and merging peaks within a 1e-4 m/z tolerance, load the spectrum's adduct annotation (e.g. [M+H]+, [M+Na]+) from labels.tsv and compute the adduct-ion mass. Subtract this adduct mass from every retained MS2 peak's m/z so peaks represent neutral fragment masses rather than ion m/z. Perform this correction before renormalizing by maximum intensity, square-rooting intensities, and selecting the top 50 peaks with intensity above 0.003, so intensity statistics are computed on adduct-corrected peaks. The rationale is that fragmentation models such as ICEBERG operate on neutral fragment masses, so uncorrected ionized peaks would systematically shift every fragment by the adduct mass and corrupt both training targets and cosine-similarity scoring. Validate that each output spectrum entry still contains mz and intensity fields and that peak m/z values are consistent with subformula/fragment masses after correction.

## Related tools

- **ms-pred data scripts** (Implements the spectrum normalization pipeline (scan merging, peak merging, adduct subtraction, sqrt-intensity, top-50 peak selection) that contains this adduct correction step) — https://github.com/samgoldman97/ms-pred
- **ms-data-parser** (Parses raw NIST .SDF spectra into processed_data before normalization, preserving adduct metadata needed for mass correction) — https://github.com/rogerwwww/ms-data-parser

## Examples

```
python data_scripts/create_msg_simulation_dataset.py --strategy msg_all --audit-out data/spec_datasets/msg_all/audit.json
```

## Evaluation signals

- After correction, each output spectrum has at most 50 peaks with intensities in [0,1] after square-root transform
- Peak m/z values no longer include the adduct mass offset; the lowest retained peak should be consistent with a neutral fragment of the precursor formula
- Both mz and intensity fields are present in every exported spectrum entry
- Peaks below the 0.003 relative-intensity threshold are absent from the output

## Limitations

- The correction depends on correct adduct assignment; wrong adduct labels shift all peaks by a constant offset
- This is part of a normalization heuristic (top-50 peaks, 1e-4 m/z merge, sqrt intensity) that discards low-intensity peaks and may lose information
- ICEBERG's generated fragments are not guaranteed to be the correct physical explanations, so downstream interpretation of corrected spectra inherits that caveat

## Evidence

- [methods] adduct subtraction as part of normalization: [evidence span withheld — anchor status: not_found]
- [methods] full normalization procedure: [evidence span withheld — anchor status: not_found]
- [readme] readme description of model scope: "GLACIER, ICEBERG, and MARASON predict spectra at the level of molecular fragments"
