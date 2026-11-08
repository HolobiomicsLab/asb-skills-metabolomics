---
name: targeted-metabolomics-qc-workflow
description: Use when DDA-acquired LC-MS data has been converted to Aird format and
  quantified in MetaPro, and you need to batch-inspect compound integration QC and/or
  curate user-approved MS/MS spectra into a homemade spectral library.
when_to_use_negative:
- Data has not yet been converted to Aird format — run AirdPro conversion first; MetaPro
  cannot scan raw vendor files directly
- The analysis goal is untargeted feature detection rather than semi-targeted metabolite
  quantification — MetaPro targets DDA-acquired, semi-targeted workflows
- The input is already a curated spectral library — this skill builds libraries from
  freshly inspected MetaPro analysis overviews, not from importing external ones
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaPro v1.1.8
  role: Web-based application running the semi-targeted Target Search, batch QC inspection
    interface, and spectral library curation module; uses in-memory caching (Ehcache)
    to speed up MS data analysis
  repo: https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- name: AirdPro v1.1
  role: GUI client that converts vendor files (DDA/DIA/PRM) into the computation-oriented
    Aird format that MetaPro consumes
  repo: https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- name: MongoDB
  role: Database where MetaPro stores user-approved spectra gathered by metabolite
    together with instrument/collision-energy/ionization-mode/file-source metadata
- name: Redis
  role: In-memory service started alongside MetaPro; also used by AirdPro for distributed
    batch conversion tasks (ConvertJob on Redis channel Database0)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-023-02018-6
    title: 'MetaPro: a web-based metabolomics application for LC-MS data batch inspection
      and library curation'
  source_content_hash: sha256:37e615a2c12611f5eeb963cf85e0084657fcf190b2baf92db4c3403bb90830e4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro v1.1
  - MetaPro
  - MetaPro v1.1.8
  - MongoDB
  - Redis
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# targeted-metabolomics-qc-workflow

## Summary

A semi-targeted LC-MS metabolomics workflow in MetaPro that couples fast batch QC inspection of integrated peaks with homemade MS/MS spectral library curation, storing user-approved spectra and instrument metadata to improve metabolite identification accuracy.

## When to use

Use when DDA-acquired LC-MS data has been converted to Aird format and quantified in MetaPro, and you need to batch-inspect compound integration QC and/or curate user-approved MS/MS spectra into a homemade spectral library.

## When NOT to use

- Data has not yet been converted to Aird format — run AirdPro conversion first; MetaPro cannot scan raw vendor files directly
- The analysis goal is untargeted feature detection rather than semi-targeted metabolite quantification — MetaPro targets DDA-acquired, semi-targeted workflows
- The input is already a curated spectral library — this skill builds libraries from freshly inspected MetaPro analysis overviews, not from importing external ones

## Inputs

- Vendor LC-MS DDA files converted to Aird format (.aird data file + . index file with matching filenames)
- MetaPro project with imported batch folder structure, library, and Method parameters (Target Search configuration)
- Checked analysis overview results from batch QC inspection of a specific project (source of inspected compounds and spectra)
- Per-spectrum metadata: instrument type, collision energy, ionization mode, source experiment file

## Outputs

- curated_spectral_library.txt — txt-format file with first column = m/z and second column = intensity, spectra grouped by compound
- MongoDB records of user-approved MS/MS spectra grouped by metabolite (name/ID, m/z, RT) with instrument/collision-energy/ionization-mode/source metadata
- Manually re-integrated compound quantification results (via Manual Integration, Reset TargetRT)

## How to apply

Convert vendor files with AirdPro (v1.1, output .aird plus . index files sharing the same filename so AirdScanUtil can scan both), import multi-batch folder structures into a MetaPro project, then run Target Search to start the targeted analysis. On the Batch Quality Control inspection interface, review compound integrations per batch, using 'Reset integration params', 'Reset TargetRT', and 'Manual Integration' (choosing two data points) to fix poor peak integration. Once analysis overviews are checked, start library curation from the checked overviews: MetaPro's curation mechanism automatically saves user-approved spectra grouped by metabolite and records metadata (instrument type, collision energy, ionization mode, file source) into MongoDB. The rationale is combining artificial judgment with algorithm inference, since identification accuracy depends directly on spectral library quality. Finally, curated spectra grouped by compound are exported as curated_spectral_library.txt with m/z in the first column and intensity in the second.

## Related tools

- **MetaPro v1.1.8** (Web-based application running the semi-targeted Target Search, batch QC inspection interface, and spectral library curation module; uses in-memory caching (Ehcache) to speed up MS data analysis) — https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- **AirdPro v1.1** (GUI client that converts vendor files (DDA/DIA/PRM) into the computation-oriented Aird format that MetaPro consumes) — https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- **MongoDB** (Database where MetaPro stores user-approved spectra gathered by metabolite together with instrument/collision-energy/ionization-mode/file-source metadata)
- **Redis** (In-memory service started alongside MetaPro; also used by AirdPro for distributed batch conversion tasks (ConvertJob on Redis channel Database0))

## Examples

```
import pandas as pd; passed = overview[overview['check_status']=='Success']; passed.groupby(['name','mz','rt']).apply(collect_spectra)
```

## Evaluation signals

- curated_spectral_library.txt exists, is txt format, and contains m/z–intensity numeric pairs per compound (first column m/z, second column intensity)
- Every curated spectrum record in the database carries instrument type, collision energy, ionization mode, and source experiment file metadata fields
- Only compounds whose batch QC check status is 'Success' contribute spectra to the curated library, mirroring the BUILD_LIBRARY behavior of collecting inspected-and-passed spectra
- Spectra are grouped so each metabolite (name/ID, m/z, RT) accumulates all of its user-approved MS/MS spectra
- Aird conversion produced matching .aird and . files with identical filenames in the same directory, enabling AirdScanUtil to scan both

## Limitations

- MetaPro supports only MS data acquired with the data-dependent acquisition (DDA) method for its semi-targeted workflow
- The distributed/downloaded version is for academic use only; commercial use requires contacting the developers
- Aird index (.) and data (.aird) files must share the same name and directory, otherwise AirdScanUtil cannot scan the pair
- Manual inspection remains indispensable even with automated tooling, since many software packages output quantification results without manual modification functions

## Evidence

- [abstract] abstract: "MetaPro supports semi-targeted analysis workflow and functions for fast QC inspection and self-made spectral library curation with easy-to-use interfaces."
- [methods] methods: "#### 4.3.2 Batch Quality Control inspection interface This is the main inspection page on samples in a batch."
- [readme] readme: [evidence span withheld — anchor status: external_doc]
- [intro] intro: "the metabolite identification accuracy of this method is highly dependent on the quality of the spectral library"
