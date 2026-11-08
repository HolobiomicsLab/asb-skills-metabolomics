---
name: cross-database-metabolomics-dataset-retrieval
description: Use when a study requires searching a query MS/MS spectrum (or an entire
  .
when_to_use_negative:
- When the goal is to search a single already-known dataset (not the full public corpus),
  e.g., when you have a specific reprocessed experiment in hand rather than a discovery
  search across repositories.
- When the query spectra are not in .mgf or USI list format — e.g., raw vendor files
  (Thermo .RAW, Bruker .d) that have not been centroided and deconvoluted first.
- When the analysis targets a single domain-specific MASST (e.g., only microbeMASST)
  rather than a cross-database aggregation; then use the domain-specific standalone
  web app directly.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0121
tools:
- name: microbe_masst (domainMASSTs / jobs.py)
  role: Provides the batch search driver (jobs.py) that calls the GNPS Fast Search
    API to run the query spectra against the aggregated indexed corpus from GNPS/MassIVE,
    Metabolomics Workbench, Metabolights, and NORMAN and generates the tree, match,
    library, and dataset count outputs.
  repo: https://github.com/robinschmid/microbe_masst
- name: GNPS Fast Search API
  role: Backend spectral-search service (fasst.gnps2.org/fastsearch) that jobs.py
    queries to match spectra against the currently indexed public metabolomics data.
- name: GNPS (Global Natural Product Social Molecular Networking)
  role: Hosts the public untargeted metabolomics datasets and library spectra that
    make up the indexed search corpus; also hosts the molecular networking workflow
    that generates the input .mgf.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:c50c5c8cb2baa9bb8ceaa685d54928f196418759d8b3b0ab061c013bc6ae62ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/robinschmid/microbe_masst
  tools:
  - GNPS (Global Natural Product Social Molecular Networking)
  - GNPS Fast Search API
  - microbe_masst (domainMASSTs / jobs.py)
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# cross-database-metabolomics-dataset-retrieval

## Summary

Aggregate untargeted LC-MS/MS metabolomics datasets from multiple public repositories (GNPS/MassIVE, MetaboLights, Metabolomics Workbench, NORMAN) into a single indexed corpus that can be batch-searched via the GNPS Fast Search API to find spectral matches of a query spectrum of interest.

## When to use

Use when a study requires searching a query MS/MS spectrum (or an entire .mgf peak list) against the totality of public untargeted metabolomics data spread across GNPS/MassIVE, MetaboLights, Metabolomics Workbench, or NORMAN to find where that spectrum — or analogs of it — has been observed, rather than searching one dataset at a time.

## When NOT to use

- When the goal is to search a single already-known dataset (not the full public corpus), e.g., when you have a specific reprocessed experiment in hand rather than a discovery search across repositories.
- When the query spectra are not in .mgf or USI list format — e.g., raw vendor files (Thermo .RAW, Bruker .d) that have not been centroided and deconvoluted first.
- When the analysis targets a single domain-specific MASST (e.g., only microbeMASST) rather than a cross-database aggregation; then use the domain-specific standalone web app directly.

## Inputs

- A single .mgf file of deconvoluted MS/MS spectra (e.g., from MZmine / GNPS molecular networking)
- A .csv or .tsv file containing a list of Universal Spectrum Identifiers (USIs)
- Search parameter settings: minimum cosine score, m/z tolerance, minimum matching peaks

## Outputs

- Interactive HTML trees per domain MASST (e.g., _microbe.html, _plant.html)
- JSON tree files (e.g., _microbe.json)
- _matches.tsv containing all scans found to match the searched spectrum in the indexed data, including samples outside curated domain-specific MASSTs
- _library.tsv listing GNPS library spectra matching the searched spectrum (enables Level 2 annotation per Metabolomics Standards Initiative)
- _datasets.tsv with the number of unique matching samples per currently indexed dataset
- _count_domain.tsv files with match counts per specific domain MASST

## How to apply

1. Prepare the query spectra either as a single .mgf file (e.g., generated via MZmine from the molecular networking in GNPS workflow) or as a list of Universal Spectrum Identifiers (USIs) in a .csv/.tsv. 2. Add entries to the jobs.py files list as ("input_directory/input_file", "output_directory/output_prefix"). 3. Adjust search parameters — minimum cosine score, m/z tolerance, and minimum number of matching peaks — according to the research question; for drug-analog propagation the paper used a modified cosine score of at least 0.8 with at least 6 matched ions. 4. Execute jobs.py (Python 3.10), with skip_existing=True, re-running a couple of times until no new outputs are generated because the Fast Search API can intermittently fail some entries. 5. The corpus being searched is the currently indexed data in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN, so no per-dataset accession handling is needed. 6. Collect the generated _matches.tsv, _datasets.tsv, _library.tsv, JSON trees, and _count_domain.tsv outputs.

## Related tools

- **microbe_masst (domainMASSTs / jobs.py)** (Provides the batch search driver (jobs.py) that calls the GNPS Fast Search API to run the query spectra against the aggregated indexed corpus from GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN and generates the tree, match, library, and dataset count outputs.) — https://github.com/robinschmid/microbe_masst
- **GNPS Fast Search API** (Backend spectral-search service (fasst.gnps2.org/fastsearch) that jobs.py queries to match spectra against the currently indexed public metabolomics data.)
- **GNPS (Global Natural Product Social Molecular Networking)** (Hosts the public untargeted metabolomics datasets and library spectra that make up the indexed search corpus; also hosts the molecular networking workflow that generates the input .mgf.)

## Examples

```
python code/jobs.py  # after adding ("input_directory/input_file", "output_directory/output_prefix") to the files list
```

## Evaluation signals

- Re-run jobs.py with skip_existing=True a couple of times until no new output files are generated; if a final run still produces new files, the search is incomplete (Fast Search API intermittently drops entries).
- Verify that every expected output type is present for each input entry: an interactive HTML tree ending in _domain.html, a matching _domain.json, a _matches.tsv, a _library.tsv, a _datasets.tsv, and a _count_domain.tsv per domain.
- Confirm that the _matches.tsv rows satisfy the configured thresholds (minimum cosine score, m/z tolerance, and minimum matching peaks) as specified in jobs.py before the run.
- Confirm that the Python environment is exactly Python 3.10; the README explicitly requires this version.
- Check that the _library.tsv matches can be used to assign at least a Level 2 annotation under the Metabolomics Standards Initiative.

## Limitations

- The Fast Search API intermittently fails some entries on a single pass, so jobs.py must be re-run a couple of times with skip_existing=True until no new output is generated (the README notes this should not be an issue anymore, but the requirement remains).
- The search is limited to the data currently indexed in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN; newly deposited or unindexed datasets are not covered.
- The search is restricted to spectra provided in .mgf or as USI lists; raw vendor formats must be preprocessed first.
- The tool requires Python 3.10 specifically.
- The corpus includes samples beyond curated domain-specific MASSTs, so _matches.tsv can contain hits from samples that do not belong to the domain of interest.

## Evidence

- [readme] Batch search of multiple spectra against all domainMASSTs: [evidence span withheld — anchor status: external_doc]
- [readme] Fast Search API searches the indexed corpus: [evidence span withheld — anchor status: external_doc]
- [readme] Input formats accepted by jobs.py: [evidence span withheld — anchor status: external_doc]
- [readme] Re-run requirement: [evidence span withheld — anchor status: external_doc]
- [methods] Thresholds used for drug-analog propagation: [evidence span withheld — anchor status: not_found]
- [readme] Python version requirement: [evidence span withheld — anchor status: external_doc]
