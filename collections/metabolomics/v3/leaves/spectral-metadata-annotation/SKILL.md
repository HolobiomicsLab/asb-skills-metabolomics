---
name: spectral-metadata-annotation
description: Use when building or curating an in-house LC-MS/MS spectral library from
  batch QC-inspected compounds and you need per-spectrum metadata (instrument, collision
  energy, ionization mode, file source) preserved.
when_to_use_negative:
- You need to search an existing public spectral database (e.g. HMDB, METLIN) rather
  than build a homemade library
- The compounds have not passed batch QC inspection — unchecked/failed overview records
  must not enter the curated library
- The task is raw vendor-file conversion or batch QC inspection itself rather than
  library curation (use AirdPro/MetaPro Target Search workflow instead)
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaPro
  role: Web application providing the library curation module (BUILD_LIBRARY) that
    accumulates approved spectra per metabolite, records metadata into the database,
    supports multi-selection deletion at library QC, and exports curated_spectral_library.txt
  repo: https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- name: MongoDB
  role: Database backend in which MetaPro stores the user-approved spectra grouped
    by metabolite together with instrument/collision/ionization/source metadata
- name: AirdPro
  role: GUI client used upstream to convert vendor LC-MS files to the Aird format
    that MetaPro analyzes; AirdPro v1.1 is bundled with the MetaPro download
  repo: https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-023-02018-6
    title: 'MetaPro: a web-based metabolomics application for LC-MS data batch inspection
      and library curation'
  source_content_hash: sha256:6f85402ca6984b8afa9efec3a1cff9ec723b33d7e952097ef830afd2311a77e8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - MetaPro
  - MongoDB
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# spectral-metadata-annotation

## Summary

Curates a homemade MS/MS spectral library in MetaPro by accumulating user-approved spectra grouped per metabolite and annotating each spectrum with instrument type, collision energy, ionization mode, and source experiment file metadata stored in MongoDB.

## When to use

Use when building or curating an in-house LC-MS/MS spectral library from batch QC-inspected compounds and you need per-spectrum metadata (instrument, collision energy, ionization mode, file source) preserved.

## When NOT to use

- You need to search an existing public spectral database (e.g. HMDB, METLIN) rather than build a homemade library
- The compounds have not passed batch QC inspection — unchecked/failed overview records must not enter the curated library
- The task is raw vendor-file conversion or batch QC inspection itself rather than library curation (use AirdPro/MetaPro Target Search workflow instead)

## Inputs

- Checked analysis overview results from batch QC inspection of a MetaPro project (inspected compounds and spectra)
- Per-spectrum acquisition metadata: instrument type, collision energy, ionization mode, source experiment file
- MetaPro project backed by MongoDB

## Outputs

- curated_spectral_library.txt (txt format; first column m/z, second column intensity, spectra grouped by compound)
- Metadata-annotated spectra records in the MetaPro/MongoDB database (instrument type, collision energy, ionization mode, file source)

## How to apply

Start from the checked analysis overview results of a batch QC inspection in a MetaPro project, which supply the inspected compounds and their MS/MS spectra. Filter the overview records to retain only compounds whose check status is Success, mirroring the BUILD_LIBRARY behavior that automatically collects inspected-and-passed spectra. Group the passed spectra by compound identity (name/ID, m/z, RT) so each metabolite accumulates all of its approved MS/MS spectra, and attach per-spectrum metadata fields — instrument type, collision energy, ionization mode, and source experiment file — to each grouped record, persisted via MongoDB. At the library QC stage, use multi-selection deletion to remove low-quality spectra so only reviewed spectra remain. Finally export the curated, compound-grouped spectra to curated_spectral_library.txt, where the first column is m/z and the second column is intensity. Because identification accuracy depends heavily on spectral library quality, curating only reviewed, metadata-annotated spectra directly improves downstream peak identification.

## Related tools

- **MetaPro** (Web application providing the library curation module (BUILD_LIBRARY) that accumulates approved spectra per metabolite, records metadata into the database, supports multi-selection deletion at library QC, and exports curated_spectral_library.txt) — https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- **MongoDB** (Database backend in which MetaPro stores the user-approved spectra grouped by metabolite together with instrument/collision/ionization/source metadata)
- **AirdPro** (GUI client used upstream to convert vendor LC-MS files to the Aird format that MetaPro analyzes; AirdPro v1.1 is bundled with the MetaPro download) — https://github.com/CSi-Studio/AirdPro/releases/tag/1.1

## Examples

```
passed = overview[overview['check_status'] == 'Success']; grouped = passed.groupby(['compound_name', 'mz', 'rt']); grouped.to_csv('curated_spectral_library.txt', sep='\t', index=False, columns=['mz', 'intensity'])
```

## Evaluation signals

- curated_spectral_library.txt exists, is txt format, and contains numeric m/z–intensity pairs per compound (first column m/z, second column intensity)
- Every curated spectrum record carries the recorded instrument type, collision energy, ionization mode, and source experiment file metadata fields
- Spectra in the output are grouped by compound (name/ID, m/z, RT) with each metabolite accumulating all of its approved MS/MS spectra
- No compounds with check status other than Success appear in the curated library
- Multi-selection deletion at the library QC stage leaves only reviewed spectra in the database

## Limitations

- Only compounds whose batch QC check status is Success can feed the curated library; failed inspections are excluded by design
- MetaPro v1.1.8 is for academic use only; commercial use requires contacting the authors
- The skill depends on MetaPro's DDA-acquired, Aird-format instrument data (converted via AirdPro) — MRM/SRM-only or non-Aird inputs are outside this curation path

## Evidence

- [abstract] MetaPro supports semi-targeted analysis workflow and functions for fast QC inspection and self-made spectral library curation with easy-to-use interfaces.: "MetaPro supports semi-targeted analysis workflow and functions for fast QC inspection and self-made spectral library curation with easy-to-use interfaces."
- [intro] Identification accuracy depends on spectral library quality: "the metabolite identification accuracy of this method is highly dependent on the quality of the spectral library"
- [methods] Library page creation step in the user manual: "#### 4.2.2 Load library Enter Library page, click `Create Library` at the top."
- [readme] MetaPro backend uses MongoDB, started automatically on installation: "********* Start MongoDB *********"
