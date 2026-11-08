---
name: mgf-spectrum-batch-search
description: Use when you have a single .mgf file of consensus MS/MS spectra (e.g.,
  from MZmine molecular networking) and need to determine which spectra match microbial
  metabolomes in GNPS/MassIVE-indexed data.
when_to_use_negative:
- You need to search one spectrum at a time interactively — use the standalone microbeMASST
  web app instead of the jobs.py batch workflow
- Your input is raw LC-MS data or a feature table rather than a pre-clustered .mgf
  of consensus MS/MS spectra or a USI list
- Your target reference space is plants, tissues, or food rather than microbial metabolomes
  and the domainMASSTs
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: microbeMASST
  role: Taxonomically informed reference database that the .mgf spectra are batch-searched
    against
  repo: https://masst.gnps2.org/microbemasst/
- name: jobs.py
  role: Batch-search driver script that calls the Fast Search API for all spectra
    and writes the per-domain output files
  repo: https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py
- name: Fast Search API
  role: Backend search service at https://fasst.gnps2.org/fastsearch/ that jobs.py
    queries for each spectrum
- name: MZmine
  role: Used upstream to generate the single .mgf consensus-spectrum file that serves
    as the batch-search input
  repo: https://github.com/mzmine/mzmine
- name: Python 3.10
  role: Required runtime version for executing jobs.py
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:be61ecfc32b490f3a3cfc17ddbe59fdfa1cf96b7264921e2bc68e5122c0e9225
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fast Search API
  - MZmine
  - Python 3.10
  - jobs.py
  - microbeMASST
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# mgf-spectrum-batch-search

## Summary

Batch-search an .mgf file of consensus MS/MS spectra against microbeMASST via the Fast Search API using the jobs.py script, to identify which spectra match microbial reference data and taxonomically partition the matches.

## When to use

Use when you have a single .mgf file of consensus MS/MS spectra (e.g., from MZmine molecular networking) and need to determine which spectra match microbial metabolomes in GNPS/MassIVE-indexed data.

## When NOT to use

- You need to search one spectrum at a time interactively — use the standalone microbeMASST web app instead of the jobs.py batch workflow
- Your input is raw LC-MS data or a feature table rather than a pre-clustered .mgf of consensus MS/MS spectra or a USI list
- Your target reference space is plants, tissues, or food rather than microbial metabolomes and the domainMASSTs

## Inputs

- Single .mgf file of consensus MS/MS spectra generated via MZmine or the GNPS molecular networking workflow (alternatively a .csv/.tsv list of USIs)
- jobs.py script configured with input file list and search parameters

## Outputs

- _matches.tsv (all scans matching the searched spectrum of interest)
- _library.tsv (GNPS library spectra matches, enabling Level 2 MSI annotations)
- _datasets.tsv (number of unique matching samples per indexed dataset)
- _count_domain.tsv files (match counts per domain MASST, e.g. _count_microbe.tsv)
- Interactive HTML trees and JSON files per domain (e.g., _microbe.html, _microbe.json)

## How to apply

Add entries to the file list in jobs.py as ("input_directory/input_file", "output_directory/output_prefix"), then check and adjust search parameters — minimum cosine score, m/z tolerance, and number of minimum matching peaks — to the paper's reported settings. jobs.py leverages the Fast Search API (fasst.gnps2.org/fastsearch/) to batch-search all spectra in the .mgf against the current indexed data in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN, generating outputs for all domainMASSTs simultaneously. Because some Fast Search API calls fail, re-run jobs.py with skip_existing=True a couple of times until no new output is generated. Use Python 3.10 as required by the repository. Parse the _count_domain.tsv outputs to quantify matches per domain (e.g., total microbial matches vs. human cell line matches) and the _matches.tsv, _library.tsv, and _datasets.tsv files for per-scan match details, then filter the matches (e.g., removing spectra that also match human cell lines) and split by taxonomic origin.

## Related tools

- **microbeMASST** (Taxonomically informed reference database that the .mgf spectra are batch-searched against) — https://masst.gnps2.org/microbemasst/
- **jobs.py** (Batch-search driver script that calls the Fast Search API for all spectra and writes the per-domain output files) — https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py
- **Fast Search API** (Backend search service at https://fasst.gnps2.org/fastsearch/ that jobs.py queries for each spectrum)
- **MZmine** (Used upstream to generate the single .mgf consensus-spectrum file that serves as the batch-search input) — https://github.com/mzmine/mzmine
- **Python 3.10** (Required runtime version for executing jobs.py)

## Examples

```
python jobs.py  # files list: ("input_directory/spectra.mgf", "output_directory/prefix"); re-run until no new output with skip_existing=True
```

## Evaluation signals

- Re-run jobs.py with skip_existing=True until a run produces no new output, confirming all Fast Search API calls succeeded
- All six output families are present: _matches.tsv, _library.tsv, _datasets.tsv, _count_domain.tsv files, _domain.html trees, and domain JSON files
- Total microbial match counts parsed from _count_domain.tsv reproduce the expected figures (e.g., 3,262 microbial matches for 10,047 SPF-unique spectra)
- After removing spectra that also match human cell lines (e.g., 837), the filtering cascade leaves exactly the reported count (e.g., 2,425 putative microbial spectra split 1,673 bacteria-only / 95 fungi-only / 657 both)
- Python version check confirms 3.10 was used, as the repository explicitly requires it

## Limitations

- Due to the Fast Search API, some entries fail on a single pass; sequential re-runs with skip_existing=True are needed to catch all possible matches
- _matches.tsv includes samples that are not part of the curated domain-specific MASSTs, so matches may need extra curation
- Search parameters (minimum cosine score, m/z tolerance, minimum matching peaks) must be chosen based on the research question; results are sensitive to these settings

## Evidence

- [readme] Running jobs.py allows users to leverage the Fast Search API and execute a batch search of multiple MS/MS spectra against the current indexed data: "Running [jobs.py](https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py) allows users to leverage the [Fast Search API](https://fasst.gnps2.org/fastsearch/) and execute a batch search"
- [readme] Re-run jobs.py a couple of times with skip_existing=True until no new output is generated because some Fast Search API entries fail: "Make sure to run [jobs.py] **_a couple of times_**, until no new output is generated by having the option: `skip_existing=True`. Due to the Fast Search API some of the entries will fail."
- [methods] Check and adjust the search parameters such as minimum cosine score, mz tolerance, and number of minimum matching peaks: "Check and adjust the different parameters for the search, such as minimum cosine score, mz tolerance, and number of minimum matching peaks"
- [readme] The batch search supports a single .mgf file generated via MZmine or a list of USIs in .csv/.tsv form: "You can run either a single .mgf file generated via [MZmine](https://github.com/mzmine/mzmine), from the molecular networking in GNPS workflow, or a list of [USIs]"
- [methods] A _count_domain.tsv series is generated containing match information for each specific domain MASST: "A series of _count_domain.tsv files will be generated, containing information on matches found for each specific domain MASST"
- [readme] The _matches.tsv output may include samples that are not part of the curated domain-specific MASSTs: "This includes also samples that are not part of the curated domain-specific MASSTs."
