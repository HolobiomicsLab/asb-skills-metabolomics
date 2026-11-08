---
name: microbe-taxonomy-dataset-filtering
description: Use when microbeMASST (jobs.py / Fast Search API) returns _matches.tsv
  hits that include samples not part of the curated microbeMASST domain and you must
  restrict hits to microbial monoculture datasets.
when_to_use_negative:
- The goal is to keep all public matches regardless of source organism — _matches.tsv
  is intentionally unfiltered and should be used as-is.
- The sample of interest lies outside the microbial monoculture space (e.g. plant,
  tissue, or food contexts); use plantMASST, tissueMASST, or foodMASST instead of
  restricting to microbeMASST.
- Input data are not single MS/MS spectra (e.g. already aggregated feature tables
  without spectra).
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: microbeMASST
  role: Standalone web app and batch search domain whose curated microbe spectrum
    set defines the filtering target
  repo: https://github.com/robinschmid/microbe_masst
- name: Fast Search API
  role: Performs the actual spectrum-vs-index matching whose results are then restricted
    to the microbe domain
- name: GNPS_MASST
  role: Standalone web application code base for one-spectrum-at-a-time domain MASST
    searches
  repo: https://github.com/mwang87/GNPS_MASST
- name: MZmine
  role: Generates the .mgf file whose spectra feed the microbeMASST search that is
    subsequently filtered
  repo: https://github.com/mzmine/mzmine
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:3e60d5566b28c9675e0dfd0fd1aed64e76e113008d82bb881bd2438b6ebf8897
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/robinschmid/microbe_masst
  tools:
  - Fast Search API
  - GNPS_MASST
  - MZmine
  - microbeMASST
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# microbe-taxonomy-dataset-filtering

## Summary

Restricts microbeMASST Fast Search matches to the curated bacterial, fungal, and archaeal monoculture reference space by combining Fast Search raw output with the microbeMASST taxonomic lineage counts, excluding samples outside the microbe domain and reporting per-rank coverage.

## When to use

Use when microbeMASST (jobs.py / Fast Search API) returns _matches.tsv hits that include samples not part of the curated microbeMASST domain and you must restrict hits to microbial monoculture datasets.

## When NOT to use

- The goal is to keep all public matches regardless of source organism — _matches.tsv is intentionally unfiltered and should be used as-is.
- The sample of interest lies outside the microbial monoculture space (e.g. plant, tissue, or food contexts); use plantMASST, tissueMASST, or foodMASST instead of restricting to microbeMASST.
- Input data are not single MS/MS spectra (e.g. already aggregated feature tables without spectra).

## Inputs

- Single MS/MS spectrum or spectrum list as an .mgf file generated via MZmine, or a .csv/.tsv file of USIs
- Fast Search API result files: _matches.tsv, _datasets.tsv, _library.tsv
- microbeMASST lineage/taxonomy coverage (kingdom–strain ranks, NCBI taxonomy IDs)

## Outputs

- _count_domain.tsv series of per-domain match counts
- _microbe.json and _microbe.html interactive taxonomic tree
- Filtered per-dataset/sample match counts restricted to the microbeMASST domain
- _library.tsv GNPS library hits enabling MSI Level 2 annotation

## How to apply

Run the microbeMASST batch search via jobs.py through the Fast Search API (fasst.gnps2.org/fastsearch/), which queries each input MS/MS spectrum — a single .mgf file from MZmine or a .csv/.tsv list of USIs — against currently indexed GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN data. Because the Fast Search indexes public datasets beyond microbial monocultures, the raw _matches.tsv 'contains also samples that are not part of the curated domain-specific MASSTs'; filtering therefore means retaining only scans whose parent dataset/sample falls within microbeMASST coverage (8 kingdoms, 20 phyla, 48 classes, 124 orders, 278 families, 561 genera, 1379 species, 542 strains), which is emitted as the _count_domain.tsv / _microbe.json / _microbe.html outputs. Match quality is controlled at the search-parameter level by adjusting the minimum cosine score, m/z tolerance, and number of minimum matching peaks before the domain restriction is applied. Because some Fast Search entries silently fail, re-run jobs.py with skip_existing=True until no new output is generated so that filtering does not discard spectra that simply have not been searched yet. Python 3.10 is required. Rationale: taxonomic restriction converts a generic MASST similarity match into a microbe-specific result, since microbeMASST's value comes from referencing spectra extracted only from microbial monoculture extracts.

## Related tools

- **microbeMASST** (Standalone web app and batch search domain whose curated microbe spectrum set defines the filtering target) — https://github.com/robinschmid/microbe_masst
- **Fast Search API** (Performs the actual spectrum-vs-index matching whose results are then restricted to the microbe domain)
- **GNPS_MASST** (Standalone web application code base for one-spectrum-at-a-time domain MASST searches) — https://github.com/mwang87/GNPS_MASST
- **MZmine** (Generates the .mgf file whose spectra feed the microbeMASST search that is subsequently filtered) — https://github.com/mzmine/mzmine

## Examples

```
python code/jobs.py  # with entries ('input_directory/input_file', 'output_directory/output_prefix'), run repeatedly until no new output with skip_existing=True
```

## Evaluation signals

- _count_domain.tsv per-domain counts sum to a value consistent with the filtered _matches.tsv entries
- Every retained match maps to a sample/dataset covered by microbeMASST's 542-strain/1379-species lineage space
- Re-running jobs.py with skip_existing=True produces no new output, confirming all spectra were actually searched before filtering
- _microbe.html tree renders non-empty; a fully empty _microbe.json indicates the filter removed everything or the search silently failed
- All reported _library.tsv GNPS library hits persist as Level 2 MSI annotations in the final filtered output

## Limitations

- Fast Search API entries can silently fail; repeated re-runs of jobs.py (skip_existing=True) are needed until no new output is produced.
- Raw Fast Search results include non-microbe samples, so filtering is mandatory if only microbial monoculture data are wanted.
- Input must be one spectrum per query — standalone web apps only handle a single spectrum at a time; batch handling requires jobs.py.

## Evidence

- [readme] Fast Search raw output includes non-domain samples: "This contains all the scans found to match your searched spectrum of interest in the data that have been currently indexed. This includes also samples that are not part of the curated domain-specific"
- [readme] Batch search indexes the public repositories that need domain filtering: "execute a batch search of multiple MS/MS spectra against the current indexed data in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN and generate multiple outputs for all listed"
- [readme] Search parameters control match quality before filtering: "Check and adjust the different parameters for the search, such as minimum cosine score, mz tolerance, and number of minimum matching peaks based on your research question."
- [readme] Repeated re-runs required to guarantee all spectra searched: "Make sure to run [jobs.py] **_a couple of times_**, until no new output is generated by having the option: `skip_existing=True`"
- [methods] microbeMASST taxonomic coverage that defines the filter boundary: "microbeMASST | 8 | 20 | 48 | 124 | 278 | 561 | 1379 | 542"
- [readme] Per-domain count outputs are the filtering result: "A series of _count_domain.tsv files will be generated, containing information on matches found for each specific domain MASST."
