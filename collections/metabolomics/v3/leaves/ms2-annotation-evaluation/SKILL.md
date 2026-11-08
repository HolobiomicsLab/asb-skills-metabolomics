---
name: ms2-annotation-evaluation
description: Use when a new spectral library's practical value must be quantified
  as added annotations (unique matches, total count deltas) on a public LC-MS/MS dataset,
  e.g. drug-incubated bacterial cultures.
when_to_use_negative:
- Input is already an mzmine feature table or FBMN annotation output — the skill assumes
  raw MS2 acquisition data is the starting point.
- The goal is library construction or spectral tree generation rather than benchmarking
  annotation coverage of an existing library against open alternatives.
edam_operation: http://edamontology.org/operation_3647
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: ThermoRawFileParser
  role: Convert Thermo RAW MS2 files from the public dataset to open formats (mzML/MGF)
    for downstream mzmine processing
  repo: https://github.com/CompOmics/ThermoRawFileParser
- name: mzmine
  role: Process MS2 data files into a feature table and deconvoluted MS2 spectra for
    molecular networking input
  repo: https://github.com/mzmine/mzmine
- name: MassBank-data
  role: One of the open spectral libraries queried during FBMN spectral library searches
    to establish the baseline annotation count
  repo: https://github.com/MassBank/MassBank-data
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:6adea53b34ccec6a0d12c06e227936957be07a3ecf0f21b5ed09688da2fd778f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CompOmics/ThermoRawFileParser
  tools:
  - MassBank-data
  - MassBank/MassBank-data
  - ThermoRawFileParser
  - mzmine
  edam_operation: http://edamontology.org/operation_3647
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ms2-annotation-evaluation

## Summary

Evaluate the annotation gain of a new MS2/MSn spectral library by comparing feature-based molecular networking and spectral matching counts against established open libraries (GNPS, MoNA, MassBank EU) on a shared public dataset.

## When to use

Use when a new spectral library's practical value must be quantified as added annotations (unique matches, total count deltas) on a public LC-MS/MS dataset, e.g. drug-incubated bacterial cultures.

## When NOT to use

- Input is already an mzmine feature table or FBMN annotation output — the skill assumes raw MS2 acquisition data is the starting point.
- The goal is library construction or spectral tree generation rather than benchmarking annotation coverage of an existing library against open alternatives.

## Inputs

- Public MassIVE dataset MS2 raw files (e.g. group1_B* Thermo RAW files from MSV000096589)
- Converted mzML/MGF files for mzmine processing
- Feature table and MS2 spectra files exported from mzmine
- Spectral library reference entries for the library under evaluation (e.g. MSnLib)

## Outputs

- Per-library annotation counts from the FBMN spectral matching output (e.g. 129 open, 80 MSnLib, 150 total)
- List of annotations unique to the evaluated library (e.g. 21 unique MSnLib matches)
- Subset of unique matches corresponding to known added compounds (e.g. 14 added drugs)

## How to apply

Retrieve the target MS2 raw data files from the public MassIVE dataset (e.g. the group1_B* files of MSV000096589, drug-incubated bacterial cultures) and convert them with ThermoRawFileParser. Process the converted files in mzmine to produce a feature table and deconvoluted MS2 spectra for molecular networking. Submit the resulting files to Feature-Based Molecular Networking on GNPS with spectral library searches configured against the open libraries (GNPS, MoNA, MassBank EU) and the library under evaluation (e.g. MSnLib reference entries) in the same run. Extract per-library annotation counts and the set of library matches unique to the new library, then compare against reported baselines — for the MSV000096589 evaluation the expected values were 129 open-library annotations, 80 MSnLib annotations, 150 total, and 21 unique MSnLib matches, of which 14 corresponded to added drugs. Rationale: running all libraries against the same mzmine feature table removes feature-detection variance so count deltas reflect only library coverage differences.

## Related tools

- **ThermoRawFileParser** (Convert Thermo RAW MS2 files from the public dataset to open formats (mzML/MGF) for downstream mzmine processing) — https://github.com/CompOmics/ThermoRawFileParser
- **mzmine** (Process MS2 data files into a feature table and deconvoluted MS2 spectra for molecular networking input) — https://github.com/mzmine/mzmine
- **MassBank-data** (One of the open spectral libraries queried during FBMN spectral library searches to establish the baseline annotation count) — https://github.com/MassBank/MassBank-data

## Examples

```
Feature-Based Molecular Networking job on GNPS with the mzmine feature table and MS2 spectra from group1_B* files of MSV000096589, library search enabled against GNPS, MoNA, MassBank EU, and MSnLib reference entries.
```

## Evaluation signals

- Per-library annotation counts from the FBMN output must match reported baselines: 129 (open libraries), 80 (MSnLib), 150 total
- Unique MSnLib match count must equal 21
- Of the 21 unique MSnLib matches, 14 must correspond to drugs added in the bacterial incubation experiment
- All libraries must be searched against the identical mzmine feature table and MS2 spectra files — any difference in feature detection invalidates the comparison

## Limitations

- Count deltas reflect library coverage only if the same feature table is used for all library searches; re-processing raw data per library confounds the comparison.
- The evaluation is dataset-specific: the reported counts (129/80/150/21) hold for the group1_B* subset of MSV000096589 and are not transferable to other sample types without re-running the workflow.

## Evidence

- [results] Open libraries yielded 129 annotations and MSnLib 80; MSnLib contributed 21 unique annotations, increasing the total to 150, of which 14 corresponded to added drugs.: [evidence span withheld — anchor status: not_found]
- [supplementary] Workflow step 1: retrieve MS2 files from MSV000096589: [evidence span withheld — anchor status: not_found]
- [supplementary] Workflow step 2: mzmine processing: [evidence span withheld — anchor status: not_found]
- [supplementary] Workflow step 3: FBMN library search: [evidence span withheld — anchor status: not_found]
- [supplementary] Validation criteria for annotation counts: [evidence span withheld — anchor status: not_found]
- [abstract] mzmine used for high-throughput data acquisition and processing: "built with a high-throughput data acquisition and processing pipeline in the open-source software mzmine"
