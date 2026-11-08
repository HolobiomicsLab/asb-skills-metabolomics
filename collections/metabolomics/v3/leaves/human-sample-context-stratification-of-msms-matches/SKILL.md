---
name: human-sample-context-stratification-of-msms-matches
description: Use when a MASST/microbeMASST batch search has returned per-spectrum
  match tables (_matches.tsv, _datasets.tsv, _count_domain.tsv) and you need to determine
  which matched public datasets originate from human specimens and stratify those
  matches by body site and disease cohort.
when_to_use_negative:
- The spectra being matched originate directly from human samples — microbeMASST and
  the metadataMASST joins are designed to trace spectra found in non-human or model-organism
  data into public human datasets; if the input is already human-derived, stratification
  by human sample context is redundant
- No ReDU sample metadata is available or linkable for the matched datasets — without
  metadata terms describing sample context (body site, disease state) the stratification
  cannot be performed and only raw match counts are available
- The search has not yet completed all re-runs of jobs.py with skip_existing=True
  — matches missing because the Fast Search API failed on some entries would produce
  an undercount of human-context matches
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2640
tools:
- name: metadataMASST
  role: Web app used to generate and visualize aggregated search outputs from the
    domain-specific MASSTs, which is the basis for stratifying matched datasets by
    sample metadata such as human body site and disease cohort
  repo: https://masst.gnps2.org/metadatamasst/
- name: Fast Search API
  role: Batch-search API used by jobs.py to query the searched spectra against indexed
    public data (GNPS/MassIVE, Metabolomics Workbench, Metabolights, NORMAN), producing
    the _matches.tsv, _datasets.tsv, and _count_domain.tsv outputs that are then joined
    to ReDU metadata
- name: Python 3.10
  role: Runtime for running jobs.py and for performing the join of match tables to
    ReDU metadata and stratification aggregation in the reproduction workflow
- name: MZmine
  role: Used to generate the single .mgf file of MS/MS spectra that constitutes the
    searched input spectra list
  repo: https://github.com/mzmine/mzmine
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:9bcd2364bf6d55b15e7ad7914e992b51ccde294ddde9b5c7903bae0944a4a14d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fast Search API
  - MZmine
  - Metabolights
  - NORMAN
  - Python 3.10
  - metadataMASST
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# human-sample-context-stratification-of-msms-matches

## Summary

Stratifies MS/MS spectral matches returned by a MASST batch search by human sample context (e.g. stool, blood, oral cavity, breast milk) and disease cohort (T2D, IBD, AD) by joining _matches.tsv and _datasets.tsv outputs to ReDU sample metadata. Used to trace microbe-derived spectra found in animal models into public human datasets and identify in which body sites and disease states they are detected.

## When to use

Use when a MASST/microbeMASST batch search has returned per-spectrum match tables (_matches.tsv, _datasets.tsv, _count_domain.tsv) and you need to determine which matched public datasets originate from human specimens and stratify those matches by body site and disease cohort.

## When NOT to use

- The spectra being matched originate directly from human samples — microbeMASST and the metadataMASST joins are designed to trace spectra found in non-human or model-organism data into public human datasets; if the input is already human-derived, stratification by human sample context is redundant
- No ReDU sample metadata is available or linkable for the matched datasets — without metadata terms describing sample context (body site, disease state) the stratification cannot be performed and only raw match counts are available
- The search has not yet completed all re-runs of jobs.py with skip_existing=True — matches missing because the Fast Search API failed on some entries would produce an undercount of human-context matches

## Inputs

- _matches.tsv output file from a MASST batch search (all scans matched to the searched spectra in indexed datasets)
- _datasets.tsv output file (number of unique matching samples per currently indexed dataset)
- _count_domain.tsv output files (per-domain match counts)
- ReDU sample metadata for the matched public datasets
- Searched spectra list (512 antibiotic-exclusive MS/MS spectra as .mgf from MZmine or USI list in .csv/.tsv)

## Outputs

- Per-spectrum human match counts (e.g. 455 of 512 spectra with at least one human match)
- Faecal-sample match table (411 stool-matched spectra; 110,973 individual MS/MS matches in stool)
- Stratified table of human matches by body site (stool, blood, oral cavity, breast milk, brain)
- Stratified table of human matches by disease cohort (T2D, IBD, AD)
- Machine-readable stratification table aggregating human-context distribution

## How to apply

After jobs.py completes the batch search of the spectra of interest (e.g. the 512 antibiotic-exclusive mouse spectra supplied as an MZmine-generated .mgf or a USI list .csv/.tsv) via the Fast Search API against GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN, collect the generated _matches.tsv, _datasets.tsv, and _count_domain.tsv outputs. Join the per-spectrum match tables to ReDU sample metadata to identify which matched datasets and samples originate from human specimens, then aggregate the number of spectra with at least one human match out of the total searched (e.g. 455 of 512). For the faecal subset, count spectra with stool-sample matches and sum the individual MS/MS matches attributed to stool samples (reproducing 411 spectra and 110,973 matches). Stratify human matches by sample context — stool, blood, oral cavity, breast milk, brain — and by disease cohort (T2D, IBD, AD) using the ReDU metadata terms attached to each matched dataset, then export the stratification table as a machine-readable output for downstream validation against reported figures.

## Related tools

- **metadataMASST** (Web app used to generate and visualize aggregated search outputs from the domain-specific MASSTs, which is the basis for stratifying matched datasets by sample metadata such as human body site and disease cohort) — https://masst.gnps2.org/metadatamasst/
- **Fast Search API** (Batch-search API used by jobs.py to query the searched spectra against indexed public data (GNPS/MassIVE, Metabolomics Workbench, Metabolights, NORMAN), producing the _matches.tsv, _datasets.tsv, and _count_domain.tsv outputs that are then joined to ReDU metadata)
- **Python 3.10** (Runtime for running jobs.py and for performing the join of match tables to ReDU metadata and stratification aggregation in the reproduction workflow)
- **MZmine** (Used to generate the single .mgf file of MS/MS spectra that constitutes the searched input spectra list) — https://github.com/mzmine/mzmine

## Examples

```
python code/jobs.py  # then join _matches.tsv/_datasets.tsv to ReDU metadata and export the human body-site/disease stratification table
```

## Evaluation signals

- The reproduced human-context table matches the reported figure that 455 of 512 spectra have at least one public human-data match
- The reproduced stool stratification yields 411 spectra with faecal-sample matches and 110,973 individual MS/MS matches attributed to stool samples
- The body-site distribution includes stool (n=411 spectra), followed by blood, oral cavity, breast milk, and urine as the most common contexts
- The disease-cohort stratification represents T2D, IBD, and AD among matched human cohorts
- No missing per-spectrum match outputs remain after re-running jobs.py until skip_existing=True generates no new outputs

## Limitations

- Fast Search API entries can fail on first run, requiring multiple re-runs of jobs.py with skip_existing=True until no new output is generated; incomplete re-runs produce undercounted match tables
- The _matches.tsv output also includes matches in samples that are not part of the curated domain-specific MASSTs, so raw match counts must be filtered to human contexts before stratification
- Domain-specific MASST taxonomic coverage differs by tool (microbeMASST covers 8 kingdoms, 1379 species, 542 strains), so spectra outside a tool's lineage coverage may not be recovered in its per-domain match counts

## Evidence

- [readme] Fast Search API batch search produces _matches.tsv, _datasets.tsv, and _count_domain.tsv used for stratification: "A _matches.tsv file will be generated. This contains all the scans found to match your searched spectrum of interest in the data that have been currently indexed."
- [readme] _datasets.tsv holds unique sample counts per indexed dataset: "A _datasets.tsv file will be generated. This contains the number of unique samples found to be matching your searched spectrum in each currently indexed dataset."
- [methods] search parameters for the batch run: "Check and adjust the different parameters for the search, such as minimum cosine score, mz tolerance, and number of minimum matching peaks based on your research question."
- [readme] MASST re-runs until no new outputs: "Make sure to run [jobs.py] **_a couple of times_**, until no new output is generated by having the option: `skip_existing=True`."
