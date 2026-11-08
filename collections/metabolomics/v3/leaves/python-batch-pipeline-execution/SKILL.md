---
name: python-batch-pipeline-execution
description: Use when you have a whole .mgf file (or a USI .csv/.tsv list) of consensus
  MS/MS spectra that must be searched in batch against a domain-specific MASST, and
  single-spectrum web-app queries would be too slow.
when_to_use_negative:
- When searching only one spectrum at a time — the standalone microbeMASST web app
  (https://masst.gnps2.org/microbemasst/) is the intended route for single queries
- When the input is raw vendor LC-MS data or a feature table rather than a processed
  .mgf of consensus MS/MS spectra or a USI list
- When the target reference space is plants/tissues/food rather than microbial taxa
  — a different domainMASST entry point may be more appropriate, though jobs.py covers
  all listed domainMASSTs simultaneously
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: microbeMASST
  role: Domain-specific target database searched by the batch job; standalone web
    app at https://masst.gnps2.org/microbemasst/
  repo: https://github.com/mwang87/GNPS_MASST
- name: jobs.py
  role: Python batch driver script that calls the Fast Search API for every spectrum
    in the input .mgf/USI file and writes all domainMASST outputs
  repo: https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py
- name: Fast Search API
  role: HTTP search endpoint (https://fasst.gnps2.org/fastsearch/) queried by jobs.py
    for each spectrum against indexed data
- name: MZmine
  role: Pre-processing tool used to produce the single .mgf file of consensus MS/MS
    spectra that serves as the batch-search input
  repo: https://github.com/mzmine/mzmine
- name: Python 3.10
  role: Required interpreter version for running jobs.py
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:0ddca970519019e69f06a167d3d55e31bd2db4a92a22bf75b9a78a22639ee8e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mwang87/GNPS_MASST
  tools:
  - Fast Search API
  - MZmine
  - Python 3.10
  - jobs.py
  - microbeMASST
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# python-batch-pipeline-execution

## Summary

Run jobs.py from robinschmid/microbe_masst to batch-search many MS/MS spectra via the GNPS Fast Search API against indexed GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN data, generating per-domain outputs for microbeMASST and related domainMASSTs.

## When to use

Use when you have a whole .mgf file (or a USI .csv/.tsv list) of consensus MS/MS spectra that must be searched in batch against a domain-specific MASST, and single-spectrum web-app queries would be too slow.

## When NOT to use

- When searching only one spectrum at a time — the standalone microbeMASST web app (https://masst.gnps2.org/microbemasst/) is the intended route for single queries
- When the input is raw vendor LC-MS data or a feature table rather than a processed .mgf of consensus MS/MS spectra or a USI list
- When the target reference space is plants/tissues/food rather than microbial taxa — a different domainMASST entry point may be more appropriate, though jobs.py covers all listed domainMASSTs simultaneously

## Inputs

- Single .mgf file generated via MZmine from the GNPS molecular networking workflow
- List of USIs provided via a .csv or .tsv file
- Search parameters: minimum cosine score, m/z tolerance, number of minimum matching peaks

## Outputs

- _matches.tsv (all scans matching each searched spectrum in the indexed data)
- _library.tsv (matching GNPS library spectra for Level 2 MSI annotations)
- _datasets.tsv (number of unique matching samples per indexed dataset)
- _count_domain.tsv files (match counts per domain MASST)
- Interactive HTML trees per domainMASST (e.g., _microbe.html) and corresponding JSON tree files (e.g., _microbe.json)

## How to apply

Navigate to code/jobs.py in the robinschmid/microbe_masst repository and add entries to the files list as ("input_directory/input_file", "output_directory/output_prefix"). Check and adjust the search parameters — minimum cosine score, m/z tolerance, and number of minimum matching peaks — to match the settings reported by the study before running. Run jobs.py under Python 3.10 with skip_existing=True, re-running it a couple of times until no new output is generated, because some Fast Search API entries fail intermittently. Each run produces _matches.tsv, _library.tsv, _datasets.tsv, _count_domain.tsv files, and interactive HTML/JSON trees (e.g., _microbe.html) per domainMASST, which downstream filtering (e.g., removing human-cell-line matches to keep putative microbial spectra) consumes.

## Related tools

- **microbeMASST** (Domain-specific target database searched by the batch job; standalone web app at https://masst.gnps2.org/microbemasst/) — https://github.com/mwang87/GNPS_MASST
- **jobs.py** (Python batch driver script that calls the Fast Search API for every spectrum in the input .mgf/USI file and writes all domainMASST outputs) — https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py
- **Fast Search API** (HTTP search endpoint (https://fasst.gnps2.org/fastsearch/) queried by jobs.py for each spectrum against indexed data)
- **MZmine** (Pre-processing tool used to produce the single .mgf file of consensus MS/MS spectra that serves as the batch-search input) — https://github.com/mzmine/mzmine
- **Python 3.10** (Required interpreter version for running jobs.py)

## Examples

```
python code/jobs.py  # files list: ("input/input.mgf", "output/prefix"), skip_existing=True
```

## Evaluation signals

- Re-run jobs.py with skip_existing=True until no new output files or rows appear; convergence on identical outputs indicates the batch completed despite intermittent API failures
- All four output families are present for each input: _matches.tsv, _library.tsv, _datasets.tsv, and _count_domain.tsv, plus _domain.html/_domain.json trees
- _count_domain.tsv totals reconcile with downstream filtering cascade counts (e.g., in the source task: 3,262 microbial matches → 837 human-cell-line matches removed → 2,425 remaining, split 1,673 bacteria-only / 95 fungi-only / 657 both)
- The aggregated output .mgf contains exactly the expected number of spectra (e.g., 2,425 putative microbial spectra)
- Parameter settings in jobs.py (minimum cosine score, m/z tolerance, minimum matching peaks) match the values reported by the study

## Limitations

- Some Fast Search API entries fail on a single pass, so jobs.py must be re-run a couple of times with skip_existing=True until no new output is generated
- The README notes the API failure issue 'should not be an issue anymore', but intermittent failures remain a historical failure mode
- _matches.tsv includes matches to samples that are not part of the curated domain-specific MASSTs, so raw matches can overcount domain associations
- Python version is pinned: users are instructed to use Python 3.10

## Evidence

- [readme] Running jobs.py allows users to leverage the Fast Search API and execute a batch search of multiple MS/MS spectra: "Running [jobs.py](https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py) allows users to leverage the [Fast Search API](https://fasst.gnps2.org/fastsearch/) and execute a batch search"
- [readme] Make sure to run jobs.py a couple of times, until no new output is generated by having the option skip_existing=True: "Make sure to run [jobs.py] **_a couple of times_**, until no new output is generated by having the option: `skip_existing=True`. Due to the Fast Search API some of the entries will fail."
- [methods] Check and adjust the different parameters for the search, such as minimum cosine score, mz tolerance, and number of minimum matching peaks: "Check and adjust the different parameters for the search, such as minimum cosine score, mz tolerance, and number of minimum matching peaks based on your research question."
- [readme] Please make user to use Python 3.10: "3. Please make user to use **_Python 3.10_**"
- [readme] A series of _count_domain.tsv files will be generated, containing information on matches found for each specific domain MASST: "A series of _count_domain.tsv files will be generated, containing information on matches found for each specific domain MASST."
