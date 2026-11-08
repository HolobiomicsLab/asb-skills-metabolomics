---
name: metabolomics-data-filtering
description: Use when putative microbial or taxon-linked MS/MS spectra (.mgf, e.g.
  from MZmine feature detection) need to be cross-validated against an independent
  dataset with a metadata contrast such as antibiotic-treated vs untreated samples.
when_to_use_negative:
- Input is already a processed feature/abundance table — the skill expects raw MS/MS
  spectra in .mgf or USI list form for spectral search
- The goal is library-based metabolite annotation via GNPS libraries rather than occurrence-pattern
  filtering across a dataset with a metadata contrast
- No independent validation dataset with contrasting metadata exists — exclusivity
  filtering (untreated-only retention) cannot be performed
edam_operation: http://edamontology.org/operation_3695
tools:
- name: microbeMASST
  role: Taxonomically informed search tool whose batch mode supplies matches of putative
    microbial spectra against indexed public MS/MS data
  repo: https://masst.gnps2.org/microbemasst/
- name: jobs.py
  role: Driver script that executes the batch Fast Search API calls for the .mgf spectra
    and emits the _matches/_datasets/_count_domain.tsv outputs that the filtering
    operates on
  repo: https://github.com/robinschmid/microbe_masst
- name: Fast Search API
  role: Backend endpoint (fasst.gnps2.org/fastsearch/) performing the per-spectrum
    search jobs whose failures require re-runs with skip_existing=True
- name: MZmine
  role: Upstream tool that generates the single .mgf file of spectra submitted to
    the batch search
  repo: https://github.com/mzmine/mzmine
- name: Python 3.10
  role: Runtime for jobs.py and the pandas-based filtering/aggregation of the TSV
    outputs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:bb40bd35875ed79d829ed1241a6984ef76c95cedca1e942fe889356b69f500c7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fast Search API
  - MZmine
  - Python 3.10
  - jobs.py
  - microbeMASST
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# metabolomics-data-filtering

## Summary

Filter and aggregate microbeMASST batch-search results (.mgf spectra searched via the Fast Search API through jobs.py) to isolate metabolite spectra whose occurrence pattern supports a biological origin — e.g. retaining only spectra present in untreated controls but absent from antibiotic-treated animals.

## When to use

Use when putative microbial or taxon-linked MS/MS spectra (.mgf, e.g. from MZmine feature detection) need to be cross-validated against an independent dataset with a metadata contrast such as antibiotic-treated vs untreated samples.

## When NOT to use

- Input is already a processed feature/abundance table — the skill expects raw MS/MS spectra in .mgf or USI list form for spectral search
- The goal is library-based metabolite annotation via GNPS libraries rather than occurrence-pattern filtering across a dataset with a metadata contrast
- No independent validation dataset with contrasting metadata exists — exclusivity filtering (untreated-only retention) cannot be performed

## Inputs

- Putative microbial MS/MS spectra as a single .mgf file (e.g. 2,425 spectra from MZmine)
- Metadata of an independent validation dataset (e.g. MSV000080918, antibiotic-treated vs untreated mouse groups)
- microbeMASST batch-search outputs: _matches.tsv, _datasets.tsv, _count_microbe.tsv

## Outputs

- Filtered matched spectra as a new .mgf file (e.g. 621 overlapping spectra)
- Summary .tsv of overlap counts per condition (e.g. 621 overlaps, 512 exclusive to untreated animals)

## How to apply

Load the putative spectra (.mgf, generated via MZmine) and run the microbeMASST batch search through jobs.py, which leverages the Fast Search API against indexed GNPS/MassIVE data; keep default search parameters (minimum cosine score, mz tolerance, number of minimum matching peaks) unless the question demands otherwise. Filter the *_matches.tsv output to retain only scans matching the target dataset (e.g. MSV000080918). Aggregate per-dataset and per-sample-group counts from *_datasets.tsv and *_count_microbe.tsv, splitting matches by metadata condition (antibiotic-treated vs untreated). Write the filtered spectra as a new .mgf and the overlap/exclusivity counts as a summary .tsv — re-run jobs.py several times with skip_existing=True because Fast Search API calls intermittently fail. Correctness is judged by whether the overlap counts reproduce expected values (here: 621 overlapping spectra, 512 exclusive to non-antibiotic-treated animals).

## Related tools

- **microbeMASST** (Taxonomically informed search tool whose batch mode supplies matches of putative microbial spectra against indexed public MS/MS data) — https://masst.gnps2.org/microbemasst/
- **jobs.py** (Driver script that executes the batch Fast Search API calls for the .mgf spectra and emits the _matches/_datasets/_count_domain.tsv outputs that the filtering operates on) — https://github.com/robinschmid/microbe_masst
- **Fast Search API** (Backend endpoint (fasst.gnps2.org/fastsearch/) performing the per-spectrum search jobs whose failures require re-runs with skip_existing=True)
- **MZmine** (Upstream tool that generates the single .mgf file of spectra submitted to the batch search) — https://github.com/mzmine/mzmine
- **Python 3.10** (Runtime for jobs.py and the pandas-based filtering/aggregation of the TSV outputs)

## Examples

```
python code/jobs.py  # entries ("input/putative_microbial.mgf","out/microbe") added; rerun with skip_existing=True, then filter _matches.tsv in pandas
```

## Evaluation signals

- The filtered summary table reproduces the expected overlap counts: 621 spectra matching MSV000080918, of which 512 are exclusive to untreated animals
- _matches.tsv is filtered to retain only scans with matches in the target dataset accession (MSV000080918)
- The output .mgf contains exactly the spectra retained after filtering, one MGF block per matched scan
- Re-running jobs.py with skip_existing=True produces no new output, confirming no spectra lost intermittent Fast Search API failures
- Per-condition counts from _count_microbe.tsv and _datasets.tsv agree on untreated-only exclusivity for the retained spectra

## Limitations

- Fast Search API calls intermittently fail; jobs.py must be run repeatedly with skip_existing=True until no new output is generated
- Python 3.10 is explicitly required for jobs.py
- _matches.tsv includes samples that are not part of the curated domain-specific MASSTs, so filtering by target dataset is mandatory
- Parameter choices (minimum cosine score, mz tolerance, minimum matching peaks) affect match sensitivity and must be checked against the research question

## Evidence

- [methods] leverage the [Fast Search API](https://fasst.gnps2.org/fastsearch/) and execute a batch search of multiple MS/MS spectra: "leverage the [Fast Search API](https://fasst.gnps2.org/fastsearch/) and execute a batch search of multiple MS/MS spectra"
- [readme] parameters must be adjusted per question: "Check and adjust the different parameters for the search, such as minimum cosine score, mz tolerance, and number of minimum matching peaks"
- [readme] matches.tsv content: "A _matches.tsv file will be generated. This contains all the scans found to match your searched spectrum of interest"
- [readme] datasets.tsv content: "A _datasets.tsv file will be generated. This contains the number of unique samples found to be matching your searched spectrum in each currently indexed dataset."
- [readme] re-run requirement: "Make sure to run [jobs.py] **_a couple of times_**, until no new output is generated by having the option: `skip_existing=True`."
