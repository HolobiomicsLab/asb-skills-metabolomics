---
name: mass-spectrometry-batch-search
description: Use when you have a set of MS/MS spectra (a .mgf file from MZmine or
  a list of USIs in .csv/.tsv) and need to find matching datasets and taxa in public
  metabolomics repositories.
when_to_use_negative:
- Only a single spectrum needs searching — the standalone microbeMASST web app (https://masst.gnps2.org/microbemasst/)
  searches one spectrum at a time
- Input is already a feature table or aggregated counts rather than raw MS/MS spectra/USIs
- The target domain is not microbial (e.g., plant metabolites) — use plantMASST or
  another domain-specific MASST
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: microbeMASST
  role: Taxonomically informed domain-specific MASST searched in batch mode to match
    MS/MS spectra to microbial taxa
  repo: https://github.com/robinschmid/microbe_masst
- name: Fast Search API
  role: Backend API invoked by jobs.py to execute the batch spectrum search against
    indexed repositories
  repo: https://fasst.gnps2.org/fastsearch/
- name: Python 3.10
  role: Required runtime for running jobs.py batch searches
  repo: https://github.com/robinschmid/microbe_masst
- name: MZmine
  role: Generates the single .mgf query file from LC-MS/MS feature data
  repo: https://github.com/mzmine/mzmine
- name: GNPS_MASST
  role: Standalone web applications for searching one spectrum at a time (microbeMASST
    UI)
  repo: https://github.com/mwang87/GNPS_MASST
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:08adbd4132c9a6d44477b65a81d34b9813fbf89b8ec6191ed0d812ef3cda2212
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/robinschmid/microbe_masst
  tools:
  - Fast Search API
  - GNPS_MASST
  - MZmine
  - Python 3.10
  - microbeMASST
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# mass-spectrometry-batch-search

## Summary

Batch-search multiple MS/MS spectra (e.g., GNPS library spectra of microbial metabolites) against indexed public datasets using the microbeMASST/Fast Search API via jobs.py, to trace which taxa match each spectrum. Used to taxonomically validate molecular annotations across GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN.

## When to use

Use when you have a set of MS/MS spectra (a .mgf file from MZmine or a list of USIs in .csv/.tsv) and need to find matching datasets and taxa in public metabolomics repositories.

## When NOT to use

- Only a single spectrum needs searching — the standalone microbeMASST web app (https://masst.gnps2.org/microbemasst/) searches one spectrum at a time
- Input is already a feature table or aggregated counts rather than raw MS/MS spectra/USIs
- The target domain is not microbial (e.g., plant metabolites) — use plantMASST or another domain-specific MASST

## Inputs

- .mgf file of MS/MS spectra generated via MZmine or the GNPS molecular networking workflow
- List of USIs provided as a .csv or .tsv file
- jobs.py entry list of (input file, output prefix) pairs
- Search parameters: minimum cosine score, mz tolerance, minimum matching peaks

## Outputs

- _matches.tsv with all scans matching the searched spectrum, including samples outside curated domain-specific MASSTs
- _library.tsv listing GNPS library spectra matching the query (supports MSI Level 2 annotation)
- _datasets.tsv with unique matching sample counts per indexed dataset
- _count_domain.tsv files per domain MASST
- Interactive _domain.html tree files and JSON tree files (e.g., _microbe.json)

## How to apply

Collect the query spectra as .mgf entries (generated via MZmine or the GNPS molecular networking workflow) or as USIs in a .csv/.tsv file. Add entries to the files list in jobs.py as ("input_directory/input_file", "output_directory/output_prefix") and run jobs.py under Python 3.10, which calls the Fast Search API (fasst.gnps2.org/fastsearch/) to search against currently indexed data in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN. Adjust search parameters — minimum cosine score, mz tolerance, and number of minimum matching peaks — to fit the research question; run jobs.py a couple of times with skip_existing=True until no new output is generated, since some Fast Search API entries can fail. Parse the resulting _matches.tsv (matching scans, including samples outside curated domain MASSTs), _library.tsv (GNPS library matches enabling MSI Level 2 annotation), _datasets.tsv (unique matching samples per dataset), and _count_domain.tsv files, plus interactive _domain.html trees and JSON tree files.

## Related tools

- **microbeMASST** (Taxonomically informed domain-specific MASST searched in batch mode to match MS/MS spectra to microbial taxa) — https://github.com/robinschmid/microbe_masst
- **Fast Search API** (Backend API invoked by jobs.py to execute the batch spectrum search against indexed repositories) — https://fasst.gnps2.org/fastsearch/
- **Python 3.10** (Required runtime for running jobs.py batch searches) — https://github.com/robinschmid/microbe_masst
- **MZmine** (Generates the single .mgf query file from LC-MS/MS feature data) — https://github.com/mzmine/mzmine
- **GNPS_MASST** (Standalone web applications for searching one spectrum at a time (microbeMASST UI)) — https://github.com/mwang87/GNPS_MASST

## Examples

```
python code/jobs.py  # with entries ("input_directory/input_file", "output_directory/output_prefix") and skip_existing=True
```

## Evaluation signals

- A _matches.tsv file is generated containing all scans found to match each searched spectrum
- A _datasets.tsv reports unique matching sample counts per currently indexed dataset
- jobs.py has been re-run with skip_existing=True until no new output is generated (no API-failure gaps remain)
- Domain-specific outputs (_count_microbe.tsv, _microbe.html, _microbe.json) are present for every input prefix
- Library matches in _library.tsv support MSI Level 2 annotations consistent with the query spectra

## Limitations

- Some Fast Search API entries can fail on a single run; sequential re-runs with skip_existing=True are needed to catch all possible matches
- The batch search is restricted to currently indexed data; newly deposited or unindexed datasets are missed
- Python 3.10 is required; other versions may not work
- _matches.tsv includes samples that are not part of the curated domain-specific MASSTs, so results must be filtered to curated taxa

## Evidence

- [methods] Running jobs.py allows users to leverage the Fast Search API and execute a batch search of multiple MS/MS spectra against the current indexed data in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN: "Running [jobs.py](https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py) allows users to leverage the [Fast Search API]"
- [readme] Check and adjust the different parameters for the search, such as minimum cosine score, mz tolerance, and number of minimum matching peaks based on your research question.: "Check and adjust the different parameters for the search, such as minimum cosine score, mz tolerance, and number of minimum matching peaks based on your research question."
- [readme] Make sure to run jobs.py a couple of times, until no new output is generated by having the option: skip_existing=True: "run [jobs.py] **_a couple of times_**, until no new output is generated by having the option: `skip_existing=True`"
- [readme] A _matches.tsv file will be generated. This contains all the scans found to match your searched spectrum of interest in the data that have been currently indexed.: "A _matches.tsv file will be generated. This contains all the scans found to match your searched spectrum of interest in the data that have been currently indexed."
- [readme] You can run either a single .mgf file generated via MZmine, from the molecular networking in GNPS workflow, or a list of USIs provided either via a .csv or .tsv file.: "a single .mgf file generated via [MZmine](https://github.com/mzmine/mzmine), from the molecular networking in GNPS workflow, or a list of [USIs]"
