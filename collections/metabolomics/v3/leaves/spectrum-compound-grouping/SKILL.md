---
name: spectrum-compound-grouping
description: Use when curating a homemade MS/MS spectral library from checked MetaPro
  batch QC inspection overviews and reviewed spectra need to be accumulated per metabolite
  with acquisition metadata.
when_to_use_negative:
- The task is batch QC inspection itself (resetting integration params or TargetRT)
  rather than curation of already-approved spectra into a library
- Input spectra have not passed manual QC inspection — the grouping should only accumulate
  inspected-and-passed spectra
- The target is comparing against public curated databases such as HMDB or METLIN
  rather than building a self-made library
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaPro
  role: Web-based application providing the batch QC inspection and BUILD_LIBRARY-based
    spectral library curation that accumulates approved spectra per metabolite
  repo: https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- name: MongoDB
  role: Database backend storing the curated spectra grouped by compound with their
    instrument/collision-energy/ionization-mode metadata
- name: AirdPro
  role: Converts vendor instrument files (DDA) to the Aird format consumed downstream
    by MetaPro's analysis and curation workflow
  repo: https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-023-02018-6
    title: 'MetaPro: a web-based metabolomics application for LC-MS data batch inspection
      and library curation'
  source_content_hash: sha256:21259d6ee3b7244653e729f965aff4fbdc0de3dfc13dad3e62a76e06d3dd20bc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - MetaPro
  - MongoDB
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# spectrum-compound-grouping

## Summary

Groups user-approved MS/MS spectra by metabolite (name/ID, m/z, RT) during MetaPro's spectral library curation, accumulating all inspected-and-passed spectra per compound with instrument, collision energy, ionization mode, and source-file metadata. This underpins self-made spectral libraries that improve peak identification accuracy.

## When to use

Use when curating a homemade MS/MS spectral library from checked MetaPro batch QC inspection overviews and reviewed spectra need to be accumulated per metabolite with acquisition metadata.

## When NOT to use

- The task is batch QC inspection itself (resetting integration params or TargetRT) rather than curation of already-approved spectra into a library
- Input spectra have not passed manual QC inspection — the grouping should only accumulate inspected-and-passed spectra
- The target is comparing against public curated databases such as HMDB or METLIN rather than building a self-made library

## Inputs

- Checked analysis overview results from MetaPro batch QC inspection of a project
- User-approved MS/MS spectra from inspected compounds (check status = Success)
- Per-spectrum metadata: instrument type, collision energy, ionization mode, source experiment file
- Compound identifiers: name/ID, m/z, retention time (RT)

## Outputs

- Curated spectra grouped by metabolite and recorded in the MetaPro database (MongoDB)
- curated_spectral_library.txt with m/z (column 1) and intensity (column 2) per compound

## How to apply

Start from the checked analysis overview produced by MetaPro's batch QC inspection of a specific project, which is the source of inspected compounds and spectra. Filter the overview records (e.g., with pandas) to retain only compounds whose check status is Success, mirroring MetaPro's BUILD_LIBRARY behavior that automatically collects inspected-and-passed spectra. Group the passed spectra by compound using name/ID, m/z, and RT so each metabolite accumulates all its approved MS/MS spectra, and attach per-spectrum metadata (instrument type, collision energy, ionization mode, source experiment file). At the library QC stage, support multi-selection deletion of low-quality spectra so only reviewed spectra remain, then persist the curated records into the MongoDB database and export grouped spectra to curated_spectral_library.txt with m/z in the first column and intensity in the second.

## Related tools

- **MetaPro** (Web-based application providing the batch QC inspection and BUILD_LIBRARY-based spectral library curation that accumulates approved spectra per metabolite) — https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- **MongoDB** (Database backend storing the curated spectra grouped by compound with their instrument/collision-energy/ionization-mode metadata)
- **AirdPro** (Converts vendor instrument files (DDA) to the Aird format consumed downstream by MetaPro's analysis and curation workflow) — https://github.com/CSi-Studio/AirdPro/releases/tag/1.1

## Examples

```
df = pandas.read_csv('analysis_overview.txt', sep='\t'); passed = df[df['check_status'] == 'Success'].groupby(['compound', 'mz', 'rt'])
```

## Evaluation signals

- curated_spectral_library.txt exists and is txt format
- The output file contains numeric m/z-intensity pairs per compound (m/z in column 1, intensity in column 2)
- Each curated spectrum record carries the recorded instrument type, collision energy, ionization mode, and source experiment file metadata fields
- Every grouped spectrum in the library corresponds to a compound whose check status was Success in the QC overview
- Spectra flagged as low-quality were removed via multi-selection deletion so only reviewed spectra remain

## Limitations

- The curation module must be started from checked analysis overviews of a specific project; it cannot curate spectra without prior QC inspection
- The paper supports DDA-acquired MS data only ('MetaPro supports MS data acquired with the data-dependent acquisition (DDA) method')
- MetaPro v1.1.8 is for academic use only; commercial use requires contacting the authors

## Evidence

- [methods] BUILD_LIBRARY behavior: [evidence span withheld — anchor status: not_found]
- [methods] curated output: [evidence span withheld — anchor status: not_found]
- [intro] library quality motivation: "the metabolite identification accuracy of this method is highly dependent on the quality of the spectral library"
- [abstract] curation feature: "A spectral curation function is provided to promote the prosperity of homemade MS/MS spectral libraries in the metabolomics community."
