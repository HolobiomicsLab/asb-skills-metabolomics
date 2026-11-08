---
name: compound-coverage-calculation
description: Use when a spectral library release (e.g. the MSnLib Zenodo 11163380
  deposit) must be audited for which of its constituent compounds actually have spectra,
  to reproduce headline coverage/total counts reported in a paper.
when_to_use_negative:
- When the input is a raw Orbitrap ID-X (.raw) acquisition file rather than the already
  merged/deduplicated Zenodo library deposit — coverage should be computed on the
  released library, not on unprocessed raw files.
- When the task is generating new MSn trees or metadata cleanup (jobs.py / metadata_cleanup_prefect.py)
  rather than auditing coverage of an existing library.
- When only single-stage MS2 library matching (e.g. MassBank-record comparison) is
  needed and no multi-stage tree or compound-coverage metric is required.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzmine
  role: Open-source mass-spectrometry data processing software in which the MSnLib
    high-throughput acquisition and processing pipeline was built; provides the context
    for the spectral library records being counted and joined to compound metadata.
  repo: https://github.com/mzmine/mzmine
- name: pip
  role: Installs the Python requirements needed to parse the Zenodo library files
    and compute counts/coverage (per the article's 'pip install requirements' step).
- name: Prefect 2
  role: Orchestrates the metadata-cleanup flow (metadata_cleanup_prefect.py) that
    produces the compound metadata table against which spectral counts are joined
    to compute coverage.
  repo: https://github.com/corinnabrungs/msn_tree_library
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:9502fa0a507f1ab364d5508a78bfd2395174b54918196f6c295cc0226d5bf84b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - Prefect 2
  - mzmine
  - pip
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# compound-coverage-calculation

## Summary

Computes the fraction of unique small molecules in the MSnLib library (30,008 compounds) that are represented by at least one deposited MS2 or MSn spectrum, reproducing the reported 87% compound coverage and validating that spectral counts (357,065 MS2; >2.3 million MSn after merge/deduplication) match Extended Data Tables 2 and 3.

## When to use

Use when a spectral library release (e.g. the MSnLib Zenodo 11163380 deposit) must be audited for which of its constituent compounds actually have spectra, to reproduce headline coverage/total counts reported in a paper.

## When NOT to use

- When the input is a raw Orbitrap ID-X (.raw) acquisition file rather than the already merged/deduplicated Zenodo library deposit — coverage should be computed on the released library, not on unprocessed raw files.
- When the task is generating new MSn trees or metadata cleanup (jobs.py / metadata_cleanup_prefect.py) rather than auditing coverage of an existing library.
- When only single-stage MS2 library matching (e.g. MassBank-record comparison) is needed and no multi-stage tree or compound-coverage metric is required.

## Inputs

- MSnLib spectral library files from Zenodo record 11163380 (merged/deduplicated MS2 and MSn spectra)
- compound metadata table for the 30,008 unique small molecules (per the msn_tree_library metadata template)

## Outputs

- library_totals.csv with MS2 spectrum count (357,065), merged/deduplicated MSn spectrum count (>2.3 million), and compound coverage percentage (87%)
- per-compound spectral coverage table linking compound identifiers to available MS2/MSn spectra

## How to apply

Download the MSnLib spectral files from Zenodo record 11163380 and load them with Python (pandas or pyteomics-equivalent parsers). Parse spectral records and count all MS2-level spectra, verifying the total equals 357,065 before deduplication. Merge duplicate spectra across library files and deduplicate to aggregate multi-stage (MSn) tree records, reproducing the >2.3 million MSn-spectra total. Join the deduplicated spectral counts against the compound metadata table (built via the metadata-cleanup flow from the msn_tree_library repo) to compute coverage as the fraction of the 30,008 unique small molecules represented by at least one spectrum; reproduce the reported 87%. Write the reproduced MS2 count, merged MSn count and coverage percentage into a summary table (library_totals.csv) for direct comparison with Extended Data Tables 2 and 3.

## Related tools

- **mzmine** (Open-source mass-spectrometry data processing software in which the MSnLib high-throughput acquisition and processing pipeline was built; provides the context for the spectral library records being counted and joined to compound metadata.) — https://github.com/mzmine/mzmine
- **pip** (Installs the Python requirements needed to parse the Zenodo library files and compute counts/coverage (per the article's 'pip install requirements' step).)
- **Prefect 2** (Orchestrates the metadata-cleanup flow (metadata_cleanup_prefect.py) that produces the compound metadata table against which spectral counts are joined to compute coverage.) — https://github.com/corinnabrungs/msn_tree_library

## Examples

```
python -c "import pandas as pd; df = pd.read_parquet('msnlib_11163380.parquet'); ms2 = df[df.ms_level == 2]; print(len(ms2)); merged = df.drop_duplicates(subset=['compound_id', 'spectrum']); print(f'coverage: {merged.compound_id.nunique() / 30008:.0%}')"
```

## Evaluation signals

- Reproduced MS2 spectrum count must equal exactly 357,065.
- Merged and deduplicated MSn spectrum count must exceed 2.3 million.
- Computed compound coverage must equal 87% over the 30,008 unique small molecules.
- Resulting library_totals.csv values must match Extended Data Tables 2 and 3 of the MSnLib paper.
- All spectra referenced in the coverage calculation should map to compounds present in the 30,008-compound metadata table.

## Limitations

- Counts and coverage are computed on the already merged/deduplicated Zenodo deposit; re-counting raw or pre-merge files will not reproduce the reported totals.
- Some source databases (Dictionary of Natural Products, DrugBank, LOTUS) require local files or special access; without them, compound metadata must be set to False in jobs.py, potentially affecting coverage completeness.
- Coverage depends on the metadata template being used, so inconsistent column names or missing structure information (which triggers PubChem Name search fallback) can shift the 30,008-compound denominator.

## Evidence

- [abstract] MSnLib library totals reproduce from Zenodo 11163380: "MS n Lib, a machine learning-ready open resource of >2 million spectra in MS n trees of 30,008 unique small molecules"
- [results] Zenodo deposit contains merged/deduplicated spectra for counting: [evidence span withheld — anchor status: not_found]
- [methods] Coverage targets from Extended Data Tables 2 and 3: [evidence span withheld — anchor status: not_found]
- [intro] Metadata template is required for consistent compound denominator: "Please use the [template]...for your metadata for having same column names and minimum needed information for the query."
- [intro] Some databases need local files or access, otherwise disabled: "some need a local file and/or special access otherwise set it to False in the jobs.py"
- [methods] Publication uses 'pip install requirements' for dependency setup: "## Requirements for running:

```bash
pip install requirements
```"
- [methods] Prefect flow produces metadata used for coverage joins: "Start serving the flow locally by running `metadata_cleanup_prefect.py`, the service will then run and wait for jobs to be submitted (jobs.py)"
- [abstract] Coverage denominator grounded in 30,008 unique compounds: "of 30,008 unique small molecules"
