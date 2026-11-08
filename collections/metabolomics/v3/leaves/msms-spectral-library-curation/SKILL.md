---
name: msms-spectral-library-curation
description: Use when batch QC inspection of LC-MS DDA data (e.g., TripleTOF or QE
  HF converted Aird files) has produced checked analysis overviews and you need to
  build a homemade MS/MS spectral library from inspected-and-passed compounds.
when_to_use_negative:
- Input is raw vendor instrument data (e.g., .raw, .wiff2, .d) that still needs conversion
  to Aird format via AirdPro and targeted search — curation requires already-inspected,
  user-approved spectra from batch QC.
- The goal is comparing spectra against existing curated databases such as HMDB or
  METLIN rather than building a homemade library.
- QC overview records are still unchecked or manually unreviewed — curation should
  only collect inspected-and-passed (Success) compounds, so starting from unverified
  peaks defeats the review-based curation rationale.
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaPro
  role: 'Web-based application providing the library curation module: automatically
    saves user-approved spectra grouped by metabolite, records instrument/collision-energy/ionization-mode/source
    metadata, and exports the curated library.'
  repo: https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- name: MongoDB
  role: Database into which MetaPro writes the curated spectra records and their per-spectrum
    metadata.
- name: AirdPro
  role: Upstream GUI client that converts vendor LC-MS files into the Aird computation-oriented
    format consumed by MetaPro's analysis pipeline feeding the curation module.
  repo: https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-023-02018-6
    title: 'MetaPro: a web-based metabolomics application for LC-MS data batch inspection
      and library curation'
  source_content_hash: sha256:03ba67a11e59c938d27982e0df031b3d0f71bd4babea420ce617a01ff5b74af7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - MetaPro
  - MongoDB
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# msms-spectral-library-curation

## Summary

Curate a homemade MS/MS spectral library by accumulating user-approved spectra grouped by metabolite, with recorded instrument type, collision energy, ionization mode, and source file metadata, using MetaPro's library curation module backed by MongoDB.

## When to use

Use when batch QC inspection of LC-MS DDA data (e.g., TripleTOF or QE HF converted Aird files) has produced checked analysis overviews and you need to build a homemade MS/MS spectral library from inspected-and-passed compounds.

## When NOT to use

- Input is raw vendor instrument data (e.g., .raw, .wiff2, .d) that still needs conversion to Aird format via AirdPro and targeted search — curation requires already-inspected, user-approved spectra from batch QC.
- The goal is comparing spectra against existing curated databases such as HMDB or METLIN rather than building a homemade library.
- QC overview records are still unchecked or manually unreviewed — curation should only collect inspected-and-passed (Success) compounds, so starting from unverified peaks defeats the review-based curation rationale.

## Inputs

- Checked analysis overview results from MetaPro batch QC inspection of a specific project (source of inspected compounds and spectra)
- User-approved MS/MS spectra from inspected compounds with check status Success
- Per-spectrum metadata: instrument type, collision energy, ionization mode, source experiment file

## Outputs

- Curated spectral library records grouped by compound and stored in MongoDB, each spectrum carrying instrument/collision-energy/ionization-mode/source metadata
- curated_spectral_library.txt — a txt-format file with the first column representing m/z and the second column representing intensity, spectra grouped by compound

## How to apply

Start from the checked analysis overview results of a batch QC inspection in a specific MetaPro project, which supply the inspected compounds and their MS/MS spectra. Filter the overview records to retain only compounds whose check status is Success, mirroring the BUILD_LIBRARY behavior that automatically collects inspected-and-passed spectra. Group the passed spectra by compound (name/ID, m/z, RT) so each metabolite accumulates all of its user-approved MS/MS spectra, and attach per-spectrum metadata (instrument type, collision energy, ionization mode, source experiment file) that MetaPro records into the MongoDB database. At the library QC stage, use multi-selection deletion to remove low-quality spectra so only reviewed spectra remain, then export the curated spectra grouped by compound to curated_spectral_library.txt, a txt-format file whose first column is m/z and second column is intensity.

## Related tools

- **MetaPro** (Web-based application providing the library curation module: automatically saves user-approved spectra grouped by metabolite, records instrument/collision-energy/ionization-mode/source metadata, and exports the curated library.) — https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- **MongoDB** (Database into which MetaPro writes the curated spectra records and their per-spectrum metadata.)
- **AirdPro** (Upstream GUI client that converts vendor LC-MS files into the Aird computation-oriented format consumed by MetaPro's analysis pipeline feeding the curation module.) — https://github.com/CSi-Studio/AirdPro/releases/tag/1.1

## Examples

```
bash MetaproStart.sh  # then browse localhost:8080 -> Library page -> Create Library, and start BUILD_LIBRARY from the checked batch QC overview
```

## Evaluation signals

- curated_spectral_library.txt exists and is txt format.
- The output file contains numeric m/z-intensity pairs per compound (first column m/z, second column intensity).
- Each curated spectrum carries the recorded instrument type, collision energy, ionization mode, and source experiment file metadata fields.
- Spectra in the library are grouped so each metabolite (name/ID, m/z, RT) accumulates all of its approved MS/MS spectra.
- Only compounds whose batch QC check status is Success contribute spectra; low-quality spectra removed via multi-selection deletion are absent from the export.

## Limitations

- Curation depends on prior batch QC inspection with human judgment: MetaPro's semi-targeted workflow combines artificial judgment with algorithm inference, so libraries are only as reliable as the manual inspection step.
- The workflow assumes DDA-acquired MS data; MetaPro supports MS data acquired with the data-dependent acquisition (DDA) method.
- The exported curated_spectral_library.txt is limited to m/z-intensity pairs (two columns), so full metadata must be retrieved from the database rather than the flat file.

## Evidence

- [abstract] MetaPro supports semi-targeted analysis workflow and functions for fast QC inspection and self-made spectral library curation with easy-to-use interfaces.: "MetaPro supports semi-targeted analysis workflow and functions for fast QC inspection and self-made spectral library curation with easy-to-use interfaces."
- [abstract] A spectral curation function is provided to promote the prosperity of homemade MS/MS spectral libraries in the metabolomics community.: "A spectral curation function is provided to promote the prosperity of homemade MS/MS spectral libraries in the metabolomics community."
- [intro] the metabolite identification accuracy of this method is highly dependent on the quality of the spectral library: "the metabolite identification accuracy of this method is highly dependent on the quality of the spectral library"
- [methods] MetaPro supports MS data acquired with the data-dependent acquisition (DDA) method.: "MetaPro supports MS data acquired with the data-dependent acquisition (DDA) method."
- [methods] This is the main inspection page on samples in a batch.: "This is the main inspection page on samples in a batch."
