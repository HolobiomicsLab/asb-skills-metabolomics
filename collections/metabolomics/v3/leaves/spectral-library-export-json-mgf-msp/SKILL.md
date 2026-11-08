---
name: spectral-library-export-json-mgf-msp
description: Use when a curated MSn tree collection (e.g. consensus SAME_ENERGY /
  ALL_ENERGIES / ALL_MSN_TO_PSEUDO_MS2 spectra) has passed filtering and must be persisted
  as a spectral library in .json, .mgf or .msp for downstream searching or ML training.
when_to_use_negative:
- Input is a single unprocessed raw file that has not yet been denoised, background-removed
  or merged into MSn trees — the export step expects filtered consensus spectra
- Only MS2 peak lists without MSn tree context are needed for a simple fragmentation
  search that does not require multi-stage library entries
- The target format is a proprietary vendor library rather than the open .json/.mgf/.msp
  outputs supported here
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0157
tools:
- name: mzmine
  role: open-source processing software hosting the MSn tree, merging, filtering and
    spectral-library export modules that write .json, .mgf and .msp files
  repo: https://github.com/mzmine/mzmine
- name: ThermoRawFileParser
  role: converts Thermo Orbitrap ID-X raw files into mzML, the input format consumed
    by the mzmine library-building workflow before export
  repo: https://github.com/CompOmics/ThermoRawFileParser
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:610bfb455ab303c6dd835a0fd8cfcac4321d84d4fc534cf18d38ef8b89667737
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - Orbitrap ID-X
  - ThermoRawFileParser
  - mzmine
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# spectral-library-export-json-mgf-msp

## Summary

Export processed MSn spectra from the mzmine MSn tree pipeline as an open spectral library in .json, .mgf or .msp formats so that the >2 million spectra of MSnLib (30,008 unique small molecules) become machine learning-ready and interoperable with community spectral-library infrastructure.

## When to use

Use when a curated MSn tree collection (e.g. consensus SAME_ENERGY / ALL_ENERGIES / ALL_MSN_TO_PSEUDO_MS2 spectra) has passed filtering and must be persisted as a spectral library in .json, .mgf or .msp for downstream searching or ML training.

## When NOT to use

- Input is a single unprocessed raw file that has not yet been denoised, background-removed or merged into MSn trees — the export step expects filtered consensus spectra
- Only MS2 peak lists without MSn tree context are needed for a simple fragmentation search that does not require multi-stage library entries
- The target format is a proprietary vendor library rather than the open .json/.mgf/.msp outputs supported here

## Inputs

- mzmine project containing merged consensus MSn spectra (SAME_ENERGY, ALL_ENERGIES, ALL_MSN_TO_PSEUDO_MS2) that passed the at-least-two-signals-above-noise filter
- mzML files converted from Thermo Orbitrap ID-X raw with ThermoRawFileParser
- metadata sheet with unique sample id, plate_id and well/vial location (e.g. the shared template from corinnabrungs/msn_tree_library)

## Outputs

- Spectral library file in .json format (machine learning-ready MSn tree representation)
- Spectral library file in .mgf format
- Spectral library file in .msp format
- MSn tree structure per precursor (as in MSnLib: >2 million spectra, 30,008 unique small molecules)

## How to apply

In mzmine, finish the pipeline that imports vendor or open-format data (Orbitrap ID-X raw converted via ThermoRawFileParser to mzML), denoises MSn levels, removes background artifacts, builds MSn trees, annotates compounds by exact mass with expected adducts/in-source fragments matched via unique sample ID substring, and scores precursor isolation purity to flag chimeric spectra. Merge spectra with the SAME_ENERGY, ALL_ENERGIES and ALL_MSN_TO_PSEUDO_MS2 strategies to create consensus spectra at each stage, then apply the quality filter retaining only spectra with at least two signals above noise. Finally use mzmine's spectral-library export modules to write the validated library to .json, .mgf and .msp files; MSnLib demonstrates the scale of the result: >2 million spectra in MSn trees of 30,008 unique small molecules.

## Related tools

- **mzmine** (open-source processing software hosting the MSn tree, merging, filtering and spectral-library export modules that write .json, .mgf and .msp files) — https://github.com/mzmine/mzmine
- **ThermoRawFileParser** (converts Thermo Orbitrap ID-X raw files into mzML, the input format consumed by the mzmine library-building workflow before export) — https://github.com/CompOmics/ThermoRawFileParser

## Examples

```
ThermoRawFileParser --input=raw/*.RAW --output=mzml && mzmine --task=mzmine_library_export --input=project.mzmine --format=msp,mgf,json
```

## Evaluation signals

- Exported files exist in all three formats (.json, .mgf, .msp) and are non-empty
- .json output parses cleanly and contains MSn tree structure (MS2 children under MS3/MS4 precursors)
- Spectra count matches library scale comparable to MSnLib (>2 million spectra across 30,008 unique small molecules)
- Each spectrum has a linked metadata record matched by unique sample ID and annotated with expected adduct/in-source fragment
- Every retained spectrum shows at least two signals above noise and chimeric (low isolation purity) entries are excluded

## Limitations

- Only spectra passing the at-least-two-signals-above-noise filter and precursor isolation purity checks are exportable; chimeric spectra are flagged and removed
- Community lacked open MSn reference data for diverse natural products and other chemicals — MSnLib is one of the first open resources, so external library cross-checks may be limited
- Export depends on preceding metadata cleanup; missing unique sample id or PubChem name lookup failures will leave entries unannotated

## Evidence

- [methods] export formats: "built with a high-throughput data acquisition and processing pipeline in the open-source software mzmine"
- [readme] metadata template: "Please use the [template]...for your metadata for having same column names and minimum needed information for the query."
- [readme] structure query fallback: "If no structure information is provided, it is queried from PubChem by Name search."
- [abstract] scale of library: "MS n Lib, a machine learning-ready open resource of >2 million spectra in MS n trees of 30,008 unique small molecules"
