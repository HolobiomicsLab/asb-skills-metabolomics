---
name: disease-cohort-spectral-match-profiling
description: Use when you have MASST/Fast Search outputs for a spectrum list (e.g.,
  .mgf or USI .csv) and want to know which human body sites or disease cohorts (T2D,
  IBD, AD) the matching spectra appear in.
when_to_use_negative:
- When only single-spectrum annotation (Level 2 MSI via _library.tsv) is needed and
  no metadata-based cohort profiling is required
- When matched datasets have no ReDU sample metadata, making disease/body-site stratification
  impossible
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Fast Search API
  role: Executes the batch MS/MS spectrum search against indexed repositories to generate
    match outputs
  repo: https://fasst.gnps2.org/fastsearch/
- name: jobs.py
  role: Drives the batch search and output generation from the microbeMASST repository
  repo: https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py
- name: MZmine
  role: Produces the .mgf input file of MS/MS spectra for batch searching
  repo: https://github.com/mzmine/mzmine
- name: metadataMASST
  role: Standalone web app for aggregating and visualizing search outputs by metadata
  repo: https://masst.gnps2.org/metadatamasst/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:5401cae7a10cdb31dfea08869af5d15ad80b7ae75dfb45219b462a72b0ba3d4d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fast Search API
  - MZmine
  - Metabolights
  - NORMAN
  - jobs.py
  - metadataMASST
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# disease-cohort-spectral-match-profiling

## Summary

Reconstructs the human body-site and disease-cohort distribution of MS/MS spectra of interest by joining MASST batch-search match tables (e.g., _matches.tsv, _datasets.tsv) with ReDU sample metadata, enabling reproduction of stratified counts such as 455/512 spectra with human matches and 411 stool-matched spectra.

## When to use

Use when you have MASST/Fast Search outputs for a spectrum list (e.g., .mgf or USI .csv) and want to know which human body sites or disease cohorts (T2D, IBD, AD) the matching spectra appear in.

## When NOT to use

- When only single-spectrum annotation (Level 2 MSI via _library.tsv) is needed and no metadata-based cohort profiling is required
- When matched datasets have no ReDU sample metadata, making disease/body-site stratification impossible

## Inputs

- List of MS/MS spectra of interest as .mgf (e.g., from MZmine molecular networking) or USI list (.csv/.tsv)
- Fast Search API / domainMASST batch-search outputs (_matches.tsv, _datasets.tsv, _count_domain.tsv)
- ReDU sample metadata for matched datasets

## Outputs

- Reproduced human-context distribution table (machine-readable) stratified by body site and disease cohort
- Per-spectrum aggregated counts of unique human samples and total MS/MS matches
- Stool-subset counts (spectra with stool matches and total stool MS/MS matches)

## How to apply

Run the batch search via Fast Search API against indexed GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN data, re-running jobs.py with skip_existing=True until no new outputs appear. Collect the _matches.tsv, _datasets.tsv, and _count_domain.tsv outputs per spectrum. Join the per-spectrum match tables to ReDU sample metadata to identify which matched datasets/samples originate from human specimens, aggregating the number of spectra with at least one human match out of the queried list (e.g., 455/512). Compute the stool-sample subset by counting spectra with stool matches and summing individual MS/MS matches attributed to stool samples (e.g., 411 spectra; 110,973 matches). Stratify the human matches by sample context (blood, breast milk, oral cavity, brain) and disease cohort (T2D, IBD, AD) using the ReDU metadata terms attached to the matched datasets, then export the reproduced human-context distribution table.

## Related tools

- **Fast Search API** (Executes the batch MS/MS spectrum search against indexed repositories to generate match outputs) — https://fasst.gnps2.org/fastsearch/
- **jobs.py** (Drives the batch search and output generation from the microbeMASST repository) — https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py
- **MZmine** (Produces the .mgf input file of MS/MS spectra for batch searching) — https://github.com/mzmine/mzmine
- **metadataMASST** (Standalone web app for aggregating and visualizing search outputs by metadata) — https://masst.gnps2.org/metadatamasst/

## Examples

```
python code/jobs.py && python code/jobs.py --skip_existing=True
```

## Evaluation signals

- Verify that the reproduced count of spectra with at least one human match equals the reported value (e.g., 455/512).
- Verify the stool-sample subset reproduces the reported totals (e.g., 411 spectra with stool matches and 110,973 stool MS/MS matches).
- Check that body-site stratification includes blood, breast milk, oral cavity, brain, and stool in the final table.
- Check that disease cohorts (T2D, IBD, AD) appear in the stratified human-context table.
- Confirm all intermediate outputs (_matches.tsv, _datasets.tsv, _count_domain.tsv) exist for each queried spectrum after re-running jobs.py until no new outputs are generated.

## Limitations

- Some Fast Search API entries may fail on a single pass, requiring multiple re-runs of jobs.py with skip_existing=True to capture all matches.
- Coverage depends on the currently indexed data in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN; matches reflect only those repositories.
- Python 3.10 is required for running the batch jobs.

## Evidence

- [methods] Fast Search API batch search and ReDU metadata join: [evidence span withheld — anchor status: not_found]
- [readme] skip_existing re-run requirement: [evidence span withheld — anchor status: not_found]
- [readme] Python 3.10 requirement: [evidence span withheld — anchor status: not_found]
- [readme] Input format flexibility: [evidence span withheld — anchor status: not_found]
- [results] Reproduced stratification targets: [evidence span withheld — anchor status: not_found]
