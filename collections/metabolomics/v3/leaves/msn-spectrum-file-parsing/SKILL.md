---
name: msn-spectrum-file-parsing
description: Use when the task requires reprocessing a deposited spectral library
  (e.g. MSnLib Zenodo 11163380) to reproduce reported totals such as 357,065 MS2 spectra,
  >2.3 million merged MSn spectra, and 87% compound coverage over 30,008 unique small
  molecules.
when_to_use_negative:
- Input is raw instrument data (.raw Thermo files) rather than an already-built spectral
  library — use ThermoRawFileParser to convert to mzML/MGF first
- The goal is acquisition or library building (mzmine pipeline, sequence setup for
  the Orbitrap ID-X) rather than reprocessing existing deposited library files
- Metadata cleanup or structure lookup is the task, not library-wide spectrum counting
edam_operation: http://edamontology.org/operation_0219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzmine
  role: Open-source mass spectrometry data processing software that underpins the
    high-throughput acquisition and processing pipeline generating the MSnLib library
    being parsed
  repo: https://github.com/mzmine/mzmine
- name: CompOmics/ThermoRawFileParser
  role: Converts Thermo RAW files to open formats (mzML, indexed mzML, MGF) upstream
    of spectral file parsing
  repo: https://github.com/CompOmics/ThermoRawFileParser
- name: Prefect
  role: Orchestrates the metadata cleanup and library preparation flows that feed
    the deposited library files
  repo: https://github.com/corinnabrungs/msn_tree_library
- name: pip
  role: Installs the Python requirements needed to load and parse the deposited spectral
    files
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:a6a7e0b34f0b78574ce807fb398839ed11620761365980d62dd55d252ff69c86
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - CompOmics/ThermoRawFileParser
  - Prefect
  - mzmine
  - pip
  edam_operation: http://edamontology.org/operation_0219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# msn-spectrum-file-parsing

## Summary

Parse the deposited MSnLib spectral library files (Zenodo 11163380) to count MS2 spectra, aggregate merged/deduplicated MSn tree records, and compute compound coverage so that library totals can be reproduced against Extended Data Tables 2 and 3 of the paper.

## When to use

Use when the task requires reprocessing a deposited spectral library (e.g. MSnLib Zenodo 11163380) to reproduce reported totals such as 357,065 MS2 spectra, >2.3 million merged MSn spectra, and 87% compound coverage over 30,008 unique small molecules.

## When NOT to use

- Input is raw instrument data (.raw Thermo files) rather than an already-built spectral library — use ThermoRawFileParser to convert to mzML/MGF first
- The goal is acquisition or library building (mzmine pipeline, sequence setup for the Orbitrap ID-X) rather than reprocessing existing deposited library files
- Metadata cleanup or structure lookup is the task, not library-wide spectrum counting

## Inputs

- MSnLib spectral library files from Zenodo record 11163380 (MS2 spectra and MSn tree records)
- Compound metadata sheet following the shared template (sample_id, plate_id, well location, structure/InChI or name for PubChem lookup)

## Outputs

- library_totals.csv summary table with reproduced MS2 spectrum count, merged/deduplicated MSn spectrum count, and compound coverage percentage
- Per-file spectral counts and deduplicated MSn tree records

## How to apply

Download the MSnLib spectral files from Zenodo record 11163380 and load them in Python (e.g. pandas/pyteomics). Parse each spectral record, distinguishing MS2-level spectra from the multi-stage (MSn) tree records, and count all MS2 spectra to reproduce the reported 357,065. Merge duplicate spectra across files and deduplicate to aggregate the MSn tree records, checking the merged count exceeds 2.3 million. Join spectral counts against the compound metadata (which follows the shared template with columns such as sample_id, plate_id, and well location) to compute compound coverage over the 30,008 unique small molecules, targeting the reported 87% from Extended Data Table 3. Write the reproduced totals as library_totals.csv and validate against the paper's Extended Data Tables 2 and 3.

## Related tools

- **mzmine** (Open-source mass spectrometry data processing software that underpins the high-throughput acquisition and processing pipeline generating the MSnLib library being parsed) — https://github.com/mzmine/mzmine
- **CompOmics/ThermoRawFileParser** (Converts Thermo RAW files to open formats (mzML, indexed mzML, MGF) upstream of spectral file parsing) — https://github.com/CompOmics/ThermoRawFileParser
- **Prefect** (Orchestrates the metadata cleanup and library preparation flows that feed the deposited library files) — https://github.com/corinnabrungs/msn_tree_library
- **pip** (Installs the Python requirements needed to load and parse the deposited spectral files)

## Examples

```
pip install -r requirements.txt; python parse_msnlib.py --zenodo 11163380 --output library_totals.csv
```

## Evaluation signals

- Reproduced MS2 spectrum count equals exactly 357,065
- Merged and deduplicated MSn spectrum count exceeds 2.3 million
- Computed compound coverage equals 87% over 30,008 unique small molecules
- Reproduced totals match Extended Data Tables 2 and 3 of the MSnLib paper
- Summary table library_totals.csv is produced with all three reported metrics

## Limitations

- Some spectral records may require merging across files and deduplication before totals are reproducible; skipping deduplication inflates MSn counts
- Coverage computation depends on complete compound metadata; external databases like DrugBank need local file access or special access, otherwise set them to False in jobs.py
- The skill assumes deposited library files are already built and merged — it does not generate or acquire new spectra

## Evidence

- [abstract] MSnLib deposited library totals to reproduce: "MS n Lib, a machine learning-ready open resource of >2 million spectra in MS n trees of 30,008 unique small molecules"
- [abstract] High-throughput pipeline in mzmine: "built with a high-throughput data acquisition and processing pipeline in the open-source software mzmine"
- [readme] Metadata template requirements: "Please use the [template]...for your metadata for having same column names and minimum needed information for the query."
- [readme] Structure lookup fallback: "If no structure information is provided, it is queried from PubChem by Name search."
- [methods] Python dependency installation: "## Requirements for running:

```bash
pip install requirements
```"
- [methods] Prefect deployment for library prep: "```bash
prefect deploy metadata_cleanup_prefect:cleanup_file --name local-deploy --pool local-work
```"
