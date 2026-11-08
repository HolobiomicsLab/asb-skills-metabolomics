---
name: msn-tree-construction
description: Use when converting multi-stage fragmentation raw data (e.g. Orbitrap
  ID-X runs exported as mzML) into tree-structured MSn spectra with precursor-child
  relationships, prior to library export.
when_to_use_negative:
- Input consists only of MS1 feature tables without staged MSn fragmentation data
  — no tree structure can be built.
- The goal is standalone peak picking on already-processed mzML files without precursor-child
  spectrum relationships.
- Single-stage MS2 library building without multi-stage (MS3+) fragmentation information.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzmine
  role: Open-source software that hosts the high-throughput MSn tree building, denoising,
    isolation-purity scoring, consensus merging, and .json/.mgf/.msp export workflow.
  repo: https://github.com/mzmine/mzmine
- name: ThermoRawFileParser
  role: Converts Thermo .raw files from Orbitrap ID-X acquisitions into open formats
    (mzML, indexed mzML, MGF, Parquet) that mzmine can import for MSn tree construction.
  repo: https://github.com/CompOmics/ThermoRawFileParser
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:f4bb1a7e5f5022fc730d3ce9d697fe738b38fe90d3de35519c146c95a005a836
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - Orbitrap ID-X
  - ThermoRawFileParser
  - mzmine
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# msn-tree-construction

## Summary

Builds MSn trees from acquired multi-stage fragmentation (MS2, MS3, ...) spectra by denoising, scoring precursor isolation purity to flag chimeric spectra, and merging consensus spectra across energy levels, as implemented in mzmine for the MSnLib spectral library.

## When to use

Use when converting multi-stage fragmentation raw data (e.g. Orbitrap ID-X runs exported as mzML) into tree-structured MSn spectra with precursor-child relationships, prior to library export.

## When NOT to use

- Input consists only of MS1 feature tables without staged MSn fragmentation data — no tree structure can be built.
- The goal is standalone peak picking on already-processed mzML files without precursor-child spectrum relationships.
- Single-stage MS2 library building without multi-stage (MS3+) fragmentation information.

## Inputs

- Vendor raw files or mzML files from multi-stage fragmentation acquisitions (e.g. Thermo .raw converted with ThermoRawFileParser)
- Metadata sheet with a unique sample id, plate name (plate_id) and well location for each acquisition

## Outputs

- MSn trees linking precursor and fragment spectra per compound
- Consensus merged spectra per tree stage (SAME_ENERGY, ALL_ENERGIES, ALL_MSN_TO_PSEUDO_MS2)
- Filtered spectral library entries exportable as .json, .mgf, or .msp

## How to apply

Import vendor or open-format data into mzmine, then denoise each MSn level and remove background artifacts so that only real fragment ions remain. Build MSn trees that link precursor and fragment stages, and for each precursor compute an isolation purity score to flag chimeric spectra where co-isolated species contaminate the fragmentation tree. Merge spectra within each tree using the SAME_ENERGY, ALL_ENERGIES, and ALL_MSN_TO_PSEUDO_MS2 strategies to generate consensus spectra at each stage, then filter the merged spectra to retain only entries with at least two signals above the noise threshold before export to .json, .mgf, or .msp.

## Related tools

- **mzmine** (Open-source software that hosts the high-throughput MSn tree building, denoising, isolation-purity scoring, consensus merging, and .json/.mgf/.msp export workflow.) — https://github.com/mzmine/mzmine
- **ThermoRawFileParser** (Converts Thermo .raw files from Orbitrap ID-X acquisitions into open formats (mzML, indexed mzML, MGF, Parquet) that mzmine can import for MSn tree construction.) — https://github.com/CompOmics/ThermoRawFileParser

## Examples

```
prefect deploy metadata_cleanup_prefect:cleanup_file --name local-deploy --pool local-work
```

## Evaluation signals

- Each output tree must link precursor and fragment stages consistent with the acquired MSn levels; isolated spectra without a parent should be flagged.
- Merged consensus spectra must exist for all three merge strategies (SAME_ENERGY, ALL_ENERGIES, ALL_MSN_TO_PSEUDO_MS2) for each tree.
- Every retained library spectrum must contain at least two signals above the noise threshold; entries below this cutoff are removed.
- Chimeric spectra identified by low precursor isolation purity must be flagged (or excluded) rather than silently merged into consensus trees.
- Exported files must parse as valid .json, .mgf, or .msp spectral library records with the MSn tree structure intact.

## Limitations

- Precursor isolation purity scoring flags chimeric spectra but co-isolated interference may still persist in low-purity trees.
- Spectra with fewer than two signals above noise are discarded, so very sparse fragmentations yield no library entry.
- Metadata queries against external databases (LOTUS, DrugBank, Dictionary of Natural Products) require local files or special access; missing entries set to False in jobs.py will skip annotation.

## Evidence

- [abstract] MSn trees link precursor and fragment stages and are built after denoising and background removal.: "a machine learning-ready open resource of >2 million spectra in MS n trees of 30,008 unique small molecules"
- [methods] Build MSn trees and compute precursor isolation purity scores for each precursor.: [evidence span withheld — anchor status: not_found]
- [methods] Merge spectra using the SAME_ENERGY, ALL_ENERGIES, and ALL_MSN_TO_PSEUDO_MS2 strategies to generate consensus spectra at each stage.: [evidence span withheld — anchor status: not_found]
- [methods] Quality filtering removes low-quality entries from the library.: [evidence span withheld — anchor status: not_found]
- [readme] mzmine hosts the high-throughput MSn data acquisition and processing pipeline.: "mzmine is an open-source software for mass spectrometry data processing. The goals of the project is
to provide a user-friendly, flexible and easily extendable software with a complete set of"
