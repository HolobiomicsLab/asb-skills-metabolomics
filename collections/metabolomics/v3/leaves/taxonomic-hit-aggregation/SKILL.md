---
name: taxonomic-hit-aggregation
description: Use when microbeMASST/jobs.py batch search output (_matches.tsv, _datasets.tsv,
  _count_microbe.tsv) must be parsed and condensed into a per-compound table of taxa
  and unique sample counts to answer which organisms match a molecule.
when_to_use_negative:
- The raw Fast Search API JSON or single-spectrum web app results have not yet been
  written to _matches.tsv/_datasets.tsv files by jobs.py
- The goal is spectral library annotation level assignment from _library.tsv rather
  than taxonomic attribution of matches
- The question requires interactive visualization rather than a flat aggregated table
  (use the generated _domain.html trees or metadataMASST instead)
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: microbeMASST
  role: Source of the taxonomically informed search hits (_matches.tsv, _datasets.tsv,
    _count_microbe.tsv) that the aggregation step consolidates
  repo: https://github.com/robinschmid/microbe_masst
- name: Fast Search API
  role: Batch search backend whose per-entry failures the repeated jobs.py runs with
    skip_existing=True must compensate for before aggregation
  repo: https://github.com/robinschmid/microbe_masst
- name: Python 3.10
  role: Runtime for running jobs.py and parsing the TSV outputs into the aggregated
    table
  repo: https://github.com/robinschmid/microbe_masst
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:18c866d8ae0d008c15ef88440308e4efac08b22b93c8ef3e052f40afc56c8223
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/robinschmid/microbe_masst
  tools:
  - Fast Search API
  - MZmine
  - Python 3.10
  - microbeMASST
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# taxonomic-hit-aggregation

## Summary

Aggregates per-scan MASST/microbeMASST search hits into a structured taxonomic table, listing for each searched compound the matched taxa and unique sample counts (e.g., lovastatin uniquely to Aspergillus, salinosporamide A only to Salinispora tropica).

## When to use

Use when microbeMASST/jobs.py batch search output (_matches.tsv, _datasets.tsv, _count_microbe.tsv) must be parsed and condensed into a per-compound table of taxa and unique sample counts to answer which organisms match a molecule.

## When NOT to use

- The raw Fast Search API JSON or single-spectrum web app results have not yet been written to _matches.tsv/_datasets.tsv files by jobs.py
- The goal is spectral library annotation level assignment from _library.tsv rather than taxonomic attribution of matches
- The question requires interactive visualization rather than a flat aggregated table (use the generated _domain.html trees or metadataMASST instead)

## Inputs

- _matches.tsv from microbeMASST batch search (all scans matching the searched spectrum)
- _datasets.tsv (unique sample counts per indexed dataset)
- _count_microbe.tsv and other _count_domain.tsv files (per-domain match counts)
- NCBI lineage files from the 'lineages' folder for taxonomy ID resolution
- Searched compound/spectrum list (GNPS library entries as .mgf or USIs in .csv/.tsv)

## Outputs

- Structured TSV with one row per compound listing matched taxa and unique sample counts
- Per-compound taxonomic distribution summarizing which genera/species/strains match each spectrum

## How to apply

After running jobs.py repeatedly with skip_existing=True until no new output is generated, parse the generated _matches.tsv (all scans matching the searched spectrum), _datasets.tsv (number of unique samples per indexed dataset), and _count_microbe.tsv (per-domain match counts) files. For each of the searched spectra, extract the taxonomic lineage of matching datasets from the microbeMASST tree output (_microbe.html/_microbe.json backed by NCBI taxonomy IDs in the lineages folder, covering 8 kingdoms down to 542 strains). Aggregate hits into a single TSV with one row per compound listing taxa and unique sample counts, preserving domain-specific counts (e.g., microbe vs. plant/tissue/food MASST). Validate by checking the table exists, has the expected number of compound rows, and reproduces reported distributions such as lovastatin matching only Aspergillus and no matches in human cell lines.

## Related tools

- **microbeMASST** (Source of the taxonomically informed search hits (_matches.tsv, _datasets.tsv, _count_microbe.tsv) that the aggregation step consolidates) — https://github.com/robinschmid/microbe_masst
- **Fast Search API** (Batch search backend whose per-entry failures the repeated jobs.py runs with skip_existing=True must compensate for before aggregation) — https://github.com/robinschmid/microbe_masst
- **Python 3.10** (Runtime for running jobs.py and parsing the TSV outputs into the aggregated table) — https://github.com/robinschmid/microbe_masst

## Examples

```
python code/jobs.py  # then parse _matches.tsv, _datasets.tsv and _count_microbe.tsv into a per-compound taxa x unique-sample-count TSV
```

## Evaluation signals

- Aggregated output table exists as TSV and contains exactly the expected number of compound rows (e.g., nine for the microbeMASST validation set)
- Lovastatin shows Aspergillus as its exclusive taxon in the aggregated table
- Salinosporamide A maps only to Salinispora tropica strains
- No human cell line hits appear for the searched microbial molecules
- Aggregation was performed only after repeated jobs.py runs produced no new output with skip_existing=True (all API entries retried)

## Limitations

- Some Fast Search API entries fail on a given run, so a single jobs.py execution misses matches; only repeated re-runs with skip_existing=True catch all possible matches
- _matches.tsv includes samples that are not part of the curated domain-specific MASSTs, so counts must be filtered per domain before taxonomic interpretation
- No changelog found for the repository, so output schema/behavior may drift over time

## Evidence

- [readme] A _matches.tsv file will be generated. This contains all the scans found to match your searched spectrum of interest: "A _matches.tsv file will be generated. This contains all the scans found to match your searched spectrum of interest"
- [readme] datasets tsv description: "A _datasets.tsv file will be generated. This contains the number of unique samples found to be matching your searched spectrum"
- [readme] count domain tsv description: "A series of _count_domain.tsv files will be generated, containing information on matches found for each specific domain MASST."
- [methods] rerun requirement: "Make sure to run [jobs.py] **_a couple of times_**, until no new output is generated by having the option: `skip_existing=True`"
- [readme] unindexed sample caveat: "This includes also samples that are not part of the curated domain-specific MASSTs."
