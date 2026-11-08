---
name: spectra-quality-filtering
description: Use when building or cleaning a homemade MS/MS spectral library from
  MetaPro batch QC inspection results and you need to keep only inspected-and-passed
  spectra grouped per compound with full acquisition metadata.
when_to_use_negative:
- The compounds or spectra have not been through batch QC manual inspection and have
  no check status (no inspected-and-passed records to curate)
- The goal is untargeted feature-table generation or peak quantification rather than
  MS/MS library curation
- Input is already a curated external spectral library (e.g. HMDB or METLIN) rather
  than user-inspected homemade spectra
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaPro
  role: Web application providing batch QC inspection overview and self-made spectral
    library curation; its BUILD_LIBRARY behavior automatically collects user-approved
    inspected-and-passed spectra grouped by metabolite and exports curated_spectral_library.txt
  repo: https://github.com/CSi-Studio/MetaPro
- name: MongoDB
  role: Database in which MetaPro saves user-approved spectra gathered by metabolite
    along with instrument type, collision energy, ionization mode, and file-source
    metadata
- name: AirdPro
  role: GUI client used upstream to convert vendor MS files (DDA data) to Aird format
    so they can be loaded and inspected in MetaPro before curation
  repo: https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-023-02018-6
    title: 'MetaPro: a web-based metabolomics application for LC-MS data batch inspection
      and library curation'
  source_content_hash: sha256:2402d52fe34dbd2c8aff2b235ba0cf96d8020ac727ac6d60393633feab6a74e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/MetaPro
  tools:
  - AirdPro
  - MetaPro
  - MongoDB
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# spectra-quality-filtering

## Summary

Curates a homemade MS/MS spectral library in MetaPro by loading checked batch QC analysis overviews, retaining only compounds whose inspection check status is Success, grouping user-approved spectra by metabolite, and recording per-spectrum metadata (instrument type, collision energy, ionization mode, source file) so only reviewed spectra remain in the curated library.

## When to use

Use when building or cleaning a homemade MS/MS spectral library from MetaPro batch QC inspection results and you need to keep only inspected-and-passed spectra grouped per compound with full acquisition metadata.

## When NOT to use

- The compounds or spectra have not been through batch QC manual inspection and have no check status (no inspected-and-passed records to curate)
- The goal is untargeted feature-table generation or peak quantification rather than MS/MS library curation
- Input is already a curated external spectral library (e.g. HMDB or METLIN) rather than user-inspected homemade spectra

## Inputs

- Checked batch QC inspection overview results from a MetaPro project (inspected compounds and MS/MS spectra)
- Per-spectrum acquisition metadata: instrument type, collision energy, ionization mode, source experiment file
- MetaPro spectral library database entries (MongoDB-backed)

## Outputs

- curated_spectral_library.txt — txt-format file with the first column representing m/z and the second column representing intensity, spectra grouped by compound
- Curated spectral library records in MetaPro/MongoDB, each approved spectrum annotated with instrument/collision-energy/ionization-mode metadata

## How to apply

Start from the checked analysis overview produced by MetaPro's batch QC inspection of a specific project, since this is the source of inspected compounds and spectra. Filter the overview records to retain only compounds whose check status is Success, mirroring BUILD_LIBRARY behavior that automatically collects inspected-and-passed spectra. Group the passed spectra by compound (name/ID, m/z, RT) so each metabolite accumulates all of its user-approved MS/MS spectra, attaching per-spectrum metadata (instrument type, collision energy, ionization mode, source experiment file) to each grouped record. At the library QC stage, use multi-selection deletion to remove low-quality spectra so only reviewed spectra remain; curated spectra are persisted by MetaPro into MongoDB and exported to curated_spectral_library.txt as m/z–intensity pairs.

## Related tools

- **MetaPro** (Web application providing batch QC inspection overview and self-made spectral library curation; its BUILD_LIBRARY behavior automatically collects user-approved inspected-and-passed spectra grouped by metabolite and exports curated_spectral_library.txt) — https://github.com/CSi-Studio/MetaPro
- **MongoDB** (Database in which MetaPro saves user-approved spectra gathered by metabolite along with instrument type, collision energy, ionization mode, and file-source metadata)
- **AirdPro** (GUI client used upstream to convert vendor MS files (DDA data) to Aird format so they can be loaded and inspected in MetaPro before curation) — https://github.com/CSi-Studio/AirdPro/releases/tag/1.1

## Evaluation signals

- curated_spectral_library.txt exists and is txt format
- The output file contains m/z–intensity numeric pairs per compound (first column m/z, second column intensity)
- Each curated spectrum carries recorded instrument type, collision energy, ionization mode, and source experiment file metadata fields
- Every spectrum retained in the library belongs to a compound whose batch QC check status was Success
- Low-quality spectra marked for deletion at the library QC stage are absent from the curated export (only reviewed spectra remain)

## Limitations

- Curation depends entirely on prior manual batch QC inspection: only compounds with check status Success enter the library, so uninspected or failed compounds are excluded
- MetaPro supports MS data acquired with the data-dependent acquisition (DDA) method, so curation is scoped to DDA spectra
- This version of MetaPro is for academic use only; commercial use requires contacting the developers

## Evidence

- [methods] Spectral library curation from checked overviews: [evidence span withheld — anchor status: not_found]
- [abstract] Self-made spectral library curation purpose: "A spectral curation function is provided to promote the prosperity of homemade MS/MS spectral libraries in the metabolomics community."
- [intro] Identification accuracy depends on library quality: "the metabolite identification accuracy of this method is highly dependent on the quality of the spectral library"
- [readme] Installation stack including MongoDB: "********* Start MongoDB *********"
