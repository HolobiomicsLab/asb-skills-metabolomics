---
name: python-batch-job-orchestration-with-skip-existing-reruns
description: Use when you need to run a batch search of many MS/MS spectra (an .mgf
  file from MZmine or a .csv/.tsv USI list) against multiple indexed public repositories
  and the API intermittently drops entries, requiring sequent re-runs until no new
  output is generated.
when_to_use_negative:
- When the input is a single spectrum to search one at a time — the standalone web
  apps (microbeMASST, plantMASST, etc., served from GNPS_MASST) handle one-spectrum-at-a-time
  searches instead of a batch jobs.py run.
- When a domainMASST (e.g. plantMASST) has no strain-level coverage and strain-level
  taxonomic resolution is required — the lineages folder shows plantMASST covers kingdom
  through species (1, 1, 11, 81, 319, 1796, 3712) with strain as NA, so strain-level
  questions cannot be answered there.
- When the search parameters (minimum cosine score, mz tolerance, minimum matching
  peaks) are left unadjusted — jobs.py defaults must be tuned to the research question,
  as the README instructs 'Check and adjust the different parameters for the search'.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: microbe_masst (jobs.py)
  role: Python 3.10 driver script that assembles the files list and invokes the Fast
    Search API to batch-search spectra against all domainMASSTs simultaneously, with
    skip_existing=True for idempotent reruns.
  repo: https://github.com/robinschmid/microbe_masst
- name: Fast Search API
  role: Backend REST endpoint (https://fasst.gnps2.org/fastsearch/) that jobs.py calls
    to execute the batch search against the indexed data repositories.
  repo: https://fasst.gnps2.org/fastsearch/
- name: MZmine
  role: Used upstream to generate the single .mgf input file (from the molecular networking
    in GNPS workflow) that jobs.py consumes for batch searching.
  repo: https://github.com/mzmine/mzmine
- name: GNPS_MASST
  role: Repository containing the code for the standalone web applications that search
    one spectrum at a time; the batch orchestration skill is the batch analogue of
    those single-spectrum apps.
  repo: https://github.com/mwang87/GNPS_MASST
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:cefe9f10507390aa18c404862998fea7848b0f272267dce02a933e756f38eb1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/robinschmid/microbe_masst
  tools:
  - Fast Search API
  - GNPS_MASST
  - MZmine
  - Metabolights
  - NORMAN
  - Python 3.10
  - metadataMASST
  - microbe_masst (jobs.py)
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Python batch job orchestration with skip_existing reruns

## Summary

Orchestrate batch MS/MS spectrum searches against GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN via the Fast Search API using a Python 3.10 jobs.py driver, re-running with skip_existing=True until every input spectrum has produced outputs despite transient API failures.

## When to use

Use when you need to run a batch search of many MS/MS spectra (an .mgf file from MZmine or a .csv/.tsv USI list) against multiple indexed public repositories and the API intermittently drops entries, requiring sequent re-runs until no new output is generated.

## When NOT to use

- When the input is a single spectrum to search one at a time — the standalone web apps (microbeMASST, plantMASST, etc., served from GNPS_MASST) handle one-spectrum-at-a-time searches instead of a batch jobs.py run.
- When a domainMASST (e.g. plantMASST) has no strain-level coverage and strain-level taxonomic resolution is required — the lineages folder shows plantMASST covers kingdom through species (1, 1, 11, 81, 319, 1796, 3712) with strain as NA, so strain-level questions cannot be answered there.
- When the search parameters (minimum cosine score, mz tolerance, minimum matching peaks) are left unadjusted — jobs.py defaults must be tuned to the research question, as the README instructs 'Check and adjust the different parameters for the search'.

## Inputs

- .mgf file generated via MZmine (from the molecular networking in GNPS workflow)
- .csv or .tsv file containing a list of USIs (Universal Spectrum Identifiers)
- jobs.py script from robinschmid/microbe_masst with the files list configured

## Outputs

- Interactive HTML trees ending with _domain.html (e.g., _microbe.html) for each domain-specific MASST
- JSON files for the different trees (e.g., _microbe.json)
- _matches.tsv containing all scans found to match the searched spectra, including samples not part of curated domain MASSTs
- _library.tsv listing spectra from the GNPS libraries that matched, enabling Level 2 MSI annotations
- _datasets.tsv containing the number of unique samples found to match in each currently indexed dataset
- A series of _count_domain.tsv files with match information for each specific domain MASST

## How to apply

Navigate to jobs.py in the robinschmid/microbe_masst repository and add entries to the files list as ('input_directory/input_file', 'output_directory/output_prefix'), where the input is either a single .mgf file generated via MZmine from the GNPS molecular networking workflow or a .csv/.tsv list of USIs. Check and adjust the search parameters — minimum cosine score, mz tolerance, and number of minimum matching peaks — to match your research question. Run jobs.py under Python 3.10, which invokes the Fast Search API at https://fasst.gnps2.org/fastsearch/ to batch-search spectra against currently indexed GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN data. Because some entries will fail on the first pass due to the Fast Search API, run jobs.py a couple of times with the option skip_existing=True so already-completed jobs are not re-issued, and keep re-running until no new output is generated. The rationale is that each successive rerun catches the previously failed spectra until the batch is fully covered.

## Related tools

- **microbe_masst (jobs.py)** (Python 3.10 driver script that assembles the files list and invokes the Fast Search API to batch-search spectra against all domainMASSTs simultaneously, with skip_existing=True for idempotent reruns.) — https://github.com/robinschmid/microbe_masst
- **Fast Search API** (Backend REST endpoint (https://fasst.gnps2.org/fastsearch/) that jobs.py calls to execute the batch search against the indexed data repositories.) — https://fasst.gnps2.org/fastsearch/
- **MZmine** (Used upstream to generate the single .mgf input file (from the molecular networking in GNPS workflow) that jobs.py consumes for batch searching.) — https://github.com/mzmine/mzmine
- **GNPS_MASST** (Repository containing the code for the standalone web applications that search one spectrum at a time; the batch orchestration skill is the batch analogue of those single-spectrum apps.) — https://github.com/mwang87/GNPS_MASST

## Examples

```
python code/jobs.py; # run repeatedly with skip_existing=True until no new output appears
```

## Evaluation signals

- Repeated runs with skip_existing=True progressively reduce the number of failed/missing entries until a final run produces no new output files, confirming full batch coverage.
- Each of the six output types appears per search: at minimum _matches.tsv, _library.tsv, _datasets.tsv, and one _count_domain.tsv per domainMASST (e.g., _count_microbe.tsv).
- All scans reported in _matches.tsv correspond to spectra present in the input .mgf or USI list; no unrequested spectra are returned.
- _library.tsv entries are restricted to spectra from the GNPS libraries, supporting Level 2 Metabolomics Standards Initiative annotations.
- The environment is Python 3.10 — running under a different version violates the explicit README requirement 'Please make sure to use Python 3.10'.

## Limitations

- Due to the Fast Search API, some entries will fail on a single run; multiple re-runs with skip_existing=True are required to catch all possible matches.
- The README notes this repeated-failure issue 'should not be an issue anymore', so failure rates may vary over time.
- Python 3.10 is explicitly required; other versions are not supported.
- Only one .mgf file or one USI list (.csv/.tsv) can be configured per jobs.py entry; batch searching multiple independent files requires separate entries or runs.

## Evidence

- [readme] Running jobs.py allows users to leverage the Fast Search API and execute a batch search of multiple MS/MS spectra against the current indexed data in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN: "Running [jobs.py](https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py) allows users to leverage the [Fast Search API](https://fasst.gnps2.org/fastsearch/) and execute a batch search"
- [readme] Make sure to run jobs.py a couple of times, until no new output is generated by having the option: skip_existing=True. Due to the Fast Search API some of the entries will fail.: "Make sure to run [jobs.py] **_a couple of times_**, until no new output is generated by having the option: `skip_existing=True`. Due to the Fast Search API some of the entries will fail."
- [readme] Please make user to use Python 3.10: "Please make user to use **_Python 3.10_**"
- [readme] You can run either a single .mgf file generated via MZmine, from the molecular networking in GNPS workflow, or a list of USIs provided either via a .csv or .tsv file.: "You can run either a single .mgf file generated via [MZmine](https://github.com/mzmine/mzmine), from the molecular networking in GNPS workflow, or a list of"
- [readme] Check and adjust the different parameters for the search, such as minimum cosine score, mz tolerance, and number of minimum matching peaks based on your research question.: "Check and adjust the different parameters for the search, such as minimum cosine score, mz tolerance, and number of minimum matching peaks based on your research question."
- [methods] leverage the [Fast Search API](https://fasst.gnps2.org/fastsearch/) and execute a batch search of multiple MS/MS spectra: "leverage the [Fast Search API](https://fasst.gnps2.org/fastsearch/) and execute a batch search of multiple MS/MS spectra"
- [readme] A _matches.tsv file will be generated. This contains all the scans found to match your searched spectrum of interest: "A _matches.tsv file will be generated. This contains all the scans found to match your searched spectrum of interest in the data that have been currently indexed. This includes also samples that are"
