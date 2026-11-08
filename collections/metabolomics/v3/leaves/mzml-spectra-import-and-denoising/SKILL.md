---
name: mzml-spectra-import-and-denoising
description: Use when acquired Orbitrap ID-X MSn raw files (or mzML) must be converted
  into clean spectra before building an MSn spectral library, e.g. signals at each
  MSn level need to be denoised and background artifacts removed.
when_to_use_negative:
- Input data is already an mzmine project with spectra processed — skip re-import
  and go directly to MSn tree building.
- Raw files are not from a Thermo instrument and no mzML is available, since ThermoRawFileParser
  only converts Thermo RAW files.
- Only MS2-level spectra are needed without MSn tree structure, in which case the
  denoising/merge pipeline is overkill.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mzmine
  role: performs the import, denoising, background removal and MSn tree pipeline on
    the imported spectra
  repo: https://github.com/mzmine/mzmine
- name: CompOmics/ThermoRawFileParser
  role: converts Thermo Orbitrap ID-X .raw vendor files into the mzML open format
    that mzmine can import
  repo: https://github.com/CompOmics/ThermoRawFileParser
- name: Prefect 2
  role: orchestrates the metadata cleanup and processing flow, deploying cleanup_file
    to the local-work pool that processes the imported spectra
  repo: https://github.com/corinnabrungs/msn_tree_library
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:7dc4d738337e7b97016fa04e240f1fc772c6b196b36feff75289b73fbf72e70f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - CompOmics/ThermoRawFileParser
  - Orbitrap ID-X
  - Prefect 2
  - mzmine
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mzml-spectra-import-and-denoising

## Summary

Import MSn raw data (vendor .raw or mzML) into mzmine and apply denoising plus background signal removal so that low-noise spectra can be merged into MSn tree consensus spectra for spectral library building.

## When to use

Use when acquired Orbitrap ID-X MSn raw files (or mzML) must be converted into clean spectra before building an MSn spectral library, e.g. signals at each MSn level need to be denoised and background artifacts removed.

## When NOT to use

- Input data is already an mzmine project with spectra processed — skip re-import and go directly to MSn tree building.
- Raw files are not from a Thermo instrument and no mzML is available, since ThermoRawFileParser only converts Thermo RAW files.
- Only MS2-level spectra are needed without MSn tree structure, in which case the denoising/merge pipeline is overkill.

## Inputs

- Thermo Orbitrap ID-X .raw files (converted to mzML via ThermoRawFileParser)
- already-open .mzML files
- metadata sheet with unique sample id, plate_id and vial/well location

## Outputs

- denoised, background-removed spectra loaded into an mzmine project, ready for MSn tree building and consensus-spectrum merging

## How to apply

Import vendor .raw files (converted via ThermoRawFileParser to mzML when needed) or already-open mzML files into mzmine and run its high-throughput acquisition-and-processing pipeline that denoises spectra at each MSn level and removes background artifacts. Match each imported file to its cleaned metadata record using the unique sample id substring, then annotate adducts and in-source fragments before downstream MSn tree building. Because the pipeline is driven by Prefect, start the server, deploy the cleanup_file flow to the local-work pool, and start a worker with concurrency-limit 5 before submitting jobs via jobs.py. Retain only spectra that pass quality filtering later in the pipeline — specifically spectra with at least two signals above the noise level — so the denoised import feeds a library of machine-learning-ready spectra rather than chimeric or sparse entries.

## Related tools

- **mzmine** (performs the import, denoising, background removal and MSn tree pipeline on the imported spectra) — https://github.com/mzmine/mzmine
- **CompOmics/ThermoRawFileParser** (converts Thermo Orbitrap ID-X .raw vendor files into the mzML open format that mzmine can import) — https://github.com/CompOmics/ThermoRawFileParser
- **Prefect 2** (orchestrates the metadata cleanup and processing flow, deploying cleanup_file to the local-work pool that processes the imported spectra) — https://github.com/corinnabrungs/msn_tree_library

## Examples

```
prefect work-pool update --concurrency-limit 5 local-work && prefect worker start --pool local-work
```

## Evaluation signals

- Imported spectra show no unmatched background artifact peaks after the background removal step.
- Every imported file is matched to a metadata record via the unique sample id substring before adduct/in-source-fragment annotation.
- Spectra retained after filtering have at least two signals above the noise level.
- Chimeric spectra flagged by low precursor isolation purity scores are excluded from downstream consensus merging.

## Limitations

- ThermoRawFileParser is required to convert vendor .raw files into mzML; older versions (<2.0.0) need Mono on Linux and Mac.
- Some external databases need local files and/or special access, otherwise they must be set to False in jobs.py, limiting which annotations can be applied during import.
- The pipeline is tuned for Thermo Orbitrap ID-X sequences (Xcalibur) with the required metadata sheet columns (unique sample id, plate_id, vial/well location).

## Evidence

- [abstract] MSnLib pipeline built on mzmine: "built with a high-throughput data acquisition and processing pipeline in the open-source software mzmine"
- [methods] Orbitrap ID-X sequence setup: "The sequence creation is setup for the Orbitrap ID-X (Xcalibur)."
- [methods] Prefect worker pool setup: "prefect work-pool update --concurrency-limit 5 local-work"
- [readme] ThermoRawFileParser conversion formats: [evidence span withheld — anchor status: external_doc]
- [readme] mzmine general scope: [evidence span withheld — anchor status: external_doc]
