---
name: redu-metadata-aggregation-for-spectral-matches
description: Use when you have a set of MS/MS spectra that have already been batch-searched
  against public GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN data
  via the Fast Search API, and you need to determine in which body sites (stool, blood,
  oral cavity, breast milk, urine, brain) and.
when_to_use_negative:
- Before any batch search has been run — this skill aggregates match outputs, it does
  not execute the Fast Search itself
- When matches are confined to microbeMASST/plantMASST domain outputs and no public
  dataset sample metadata is needed
- When sample context is already annotated in your own study metadata rather than
  public ReDU metadata
tools:
- name: Fast Search API
  role: Produces the _matches.tsv, _datasets.tsv, and _count_domain.tsv outputs that
    this skill joins to ReDU metadata
  repo: https://fasst.gnps2.org/fastsearch/
- name: metadataMASST
  role: Aggregates and visualizes MASST search outputs across domain-specific tools
  repo: https://masst.gnps2.org/metadatamasst/
- name: Python 3.10
  role: Runs jobs.py and performs the tabular joins/aggregations against ReDU metadata
  repo: https://github.com/robinschmid/microbe_masst
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:41b4c4d5e0cf120920651ddbe90c17154a1e549988e59aa67a4efe83d984ebc5
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# redu-metadata-aggregation-for-spectral-matches

## Summary

Join batch MASST/Fast Search match outputs (_matches.tsv, _datasets.tsv, _count_domain.tsv) to ReDU public-sample metadata to characterize where searched MS/MS spectra occur across human body sites and disease cohorts, as done to trace microbial metabolite signals into public human datasets.

## When to use

Use when you have a set of MS/MS spectra that have already been batch-searched against public GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN data via the Fast Search API, and you need to determine in which body sites (stool, blood, oral cavity, breast milk, urine, brain) and disease cohorts (T2D, IBD, AD) the matched spectra occur by attaching ReDU sample metadata.

## When NOT to use

- Before any batch search has been run — this skill aggregates match outputs, it does not execute the Fast Search itself
- When matches are confined to microbeMASST/plantMASST domain outputs and no public dataset sample metadata is needed
- When sample context is already annotated in your own study metadata rather than public ReDU metadata

## Inputs

- _matches.tsv per searched spectrum (all scans matched in indexed public data)
- _datasets.tsv per searched spectrum (number of unique matching samples per dataset)
- _count_domain.tsv outputs (matches per domain-specific MASST)
- ReDU public-sample metadata terms attached to matched datasets

## Outputs

- Per-spectrum table of matched human samples with body-site and disease-cohort annotations
- Aggregated counts: number of spectra with ≥1 human match out of total searched
- Fecal subset counts: spectra matched to stool samples and total stool MS/MS matches
- Machine-readable distribution table stratified by sample context and disease cohort

## How to apply

After running jobs.py through the Fast Search API (fasst.gnps2.org/fastsearch) until re-runs with skip_existing=True produce no new outputs, collect the per-spectrum _matches.tsv, _datasets.tsv, and _count_domain.tsv files, which contain all matched scans, unique-sample counts, and per-domain match counts for each of your searched spectra. Join these tables to ReDU sample metadata terms attached to the matched datasets, using the dataset identifiers in _datasets.tsv to pull the corresponding sample context (human subject, tissue source, disease status). Aggregate per spectrum: count spectra with at least one human match (e.g. 455/512 antibiotic-exclusive microbial spectra), and for the fecal subset count spectra matched to stool samples and sum the individual MS/MS matches attributed to stool (e.g. 411 spectra with 110,973 stool matches). Stratify the aggregated matches by sample context (blood, breast milk, oral cavity, brain, urine) and by disease cohort (T2D, IBD, AD) using the ReDU metadata terms, then export a machine-readable distribution table and validate it against the expected counts before interpreting body-site or disease associations.

## Related tools

- **Fast Search API** (Produces the _matches.tsv, _datasets.tsv, and _count_domain.tsv outputs that this skill joins to ReDU metadata) — https://fasst.gnps2.org/fastsearch/
- **metadataMASST** (Aggregates and visualizes MASST search outputs across domain-specific tools) — https://masst.gnps2.org/metadatamasst/
- **Python 3.10** (Runs jobs.py and performs the tabular joins/aggregations against ReDU metadata) — https://github.com/robinschmid/microbe_masst

## Examples

```
python code/jobs.py  # run repeatedly with skip_existing=True until no new outputs; then join _matches.tsv/_datasets.tsv to ReDU metadata
```

## Evaluation signals

- Reproduced totals match reported figures (e.g. 455/512 spectra with human matches; 411 stool-matched spectra with 110,973 MS/MS matches)
- Tissue stratification covers the expected contexts (blood, breast milk, oral cavity, brain, urine) in the final table
- Disease stratification includes T2D, IBD, and AD cohorts derived from ReDU metadata terms
- _datasets.tsv unique-sample counts reconcile with the per-spectrum _matches.tsv scan counts
- Re-running jobs.py with skip_existing=True yields no new outputs before aggregation is performed

## Limitations

- Matches include samples not part of the curated domain-specific MASSTs, so human context must be filtered explicitly via ReDU terms
- The Fast Search API can fail on some entries; repeated re-runs with skip_existing=True are required until no new outputs appear
- Must use Python 3.10 to run the jobs.py pipeline that generates these outputs

## Evidence

- [results] 455 of the 512 MS/MS spectra matched public human data, appearing in healthy individuals and disease cohorts (T2D, IBD, AD, etc.), and were most common in stool samples (n = 411 spectra; 110,973 matches): [evidence span withheld — anchor status: not_found]
- [readme] collect outputs before aggregation: "A _datasets.tsv file will be generated. This contains the number of unique samples found to be matching your searched spectrum"
- [readme] metadataMASST aggregates search outputs: "Aggregated search outputs can be generated and visualized using metadataMASST."
- [methods] re-run jobs.py until no new outputs: "Make sure to run [jobs.py] **_a couple of times_**, until no new output is generated by having the option: `skip_existing=True`"
- [results] join matches to metadata and stratify: [evidence span withheld — anchor status: not_found]
