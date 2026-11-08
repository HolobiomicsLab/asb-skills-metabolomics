---
name: mass-spectral-analog-search
description: Use when you have MS/MS reference spectra (MGF from MZmine, or a USI
  list in CSV/TSV) and want to find matching or analog spectra in thousands of indexed
  public datasets (GNPS/MassIVE, Metabolomics Workbench, MetaboLights, NORMAN).
when_to_use_negative:
- When only a single spectrum needs searching and an interactive web app (e.g., microbeMASST
  or plantMASST) suffices instead of a batch job
- When the input is already a feature table or quantified peak matrix rather than
  raw MS/MS spectra or USI lists — this skill searches spectra, not feature intensities
- When the goal is targeted quantification of known metabolites rather than analog
  discovery across public datasets
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Fast Search API (fasst)
  role: Backend service that executes the batch spectral search against all indexed
    datasets
  repo: https://fasst.gnps2.org/fastsearch/
- name: microbe_masst (jobs.py)
  role: Python driver script that submits spectra to the Fast Search API and assembles
    domainMASST outputs
  repo: https://github.com/robinschmid/microbe_masst
- name: metadataMASST
  role: Web app used to aggregate and visualize outputs from multiple domainMASST
    searches
  repo: https://masst.gnps2.org/metadatamasst/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:49af06e46253bab7fd54e82a1d80267076c1538cb5db20b3782ec629b5b7426f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fast Search API (fasst)
  - metadataMASST
  - microbe_masst (jobs.py)
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# mass-spectral-analog-search

## Summary

Searches a reference MS/MS spectrum (or an MGF/USI list) against all publicly indexed untargeted metabolomics datasets via the GNPS Fast Search API to retrieve analog spectra passing a modified-cosine threshold, enabling domain-specific and drug-analog propagation across public repositories.

## When to use

Use when you have MS/MS reference spectra (MGF from MZmine, or a USI list in CSV/TSV) and want to find matching or analog spectra in thousands of indexed public datasets (GNPS/MassIVE, Metabolomics Workbench, MetaboLights, NORMAN).

## When NOT to use

- When only a single spectrum needs searching and an interactive web app (e.g., microbeMASST or plantMASST) suffices instead of a batch job
- When the input is already a feature table or quantified peak matrix rather than raw MS/MS spectra or USI lists — this skill searches spectra, not feature intensities
- When the goal is targeted quantification of known metabolites rather than analog discovery across public datasets

## Inputs

- MGF file of MS/MS spectra generated via MZmine from the GNPS molecular networking workflow
- CSV or TSV file of USIs (Universal Spectrum Identifiers), one per spectrum
- Domain-specific search parameters: minimum cosine score, m/z tolerance, minimum number of matching peaks

## Outputs

- _matches.tsv: all scans matching the queried spectrum in indexed data, including samples outside the curated domain set
- _library.tsv: GNPS library spectra matching the query, enabling Level 2 Metabolomics Standards Initiative annotation
- _datasets.tsv: number of unique matching samples per indexed dataset
- _count_<domain>.tsv: match counts for each domain-specific MASST (microbe, plant, tissue, microbiome, food)
- _<domain>.html: interactive HTML tree files per domainMASST
- _<domain>.json: JSON tree files per domainMASST
- Aggregated drug-analog spectral pair table (e.g., drug_analog_spectral_pairs.tsv) when running across the full drug reference library

## How to apply

Prepare the input as either an MGF file generated from MZmine molecular networking or a CSV/TSV of USIs, then edit jobs.py in robinschmid/microbe_masst to add entries as (input_path, output_prefix) tuples and set search parameters (minimum cosine score, m/z tolerance, minimum matching peaks). Run jobs.py via the Fast Search API at https://fasst.gnps2.org/fastsearch/ so each spectrum is searched against all currently indexed GNPS/MassIVE, Metabolomics Workbench, MetaboLights, and NORMAN data; re-run the job with skip_existing=True a couple of times until no new output is generated, because some Fast Search entries fail intermittently. For the drug-analog use case, the article reports analog propagation was run with a modified cosine score cutoff of ≥ 0.8 and at least 6 matched ions across ~3500 LC-MS/MS datasets, retrieving analogous spectra for 24.6% of the 103,209 drug reference spectra (19 million drug-analog pairs). The job outputs include _matches.tsv (all matched scans with sample metadata), _library.tsv (GNPS library spectra hits for Level 2 MSI annotation), _datasets.tsv (per-dataset unique sample match counts), interactive HTML trees ending in _domain.html (e.g., _microbe.html, _tissue.html), and JSON tree files per domainMASST; metadataMASST can be used to aggregate and visualize cross-domain search outputs.

## Related tools

- **Fast Search API (fasst)** (Backend service that executes the batch spectral search against all indexed datasets) — https://fasst.gnps2.org/fastsearch/
- **microbe_masst (jobs.py)** (Python driver script that submits spectra to the Fast Search API and assembles domainMASST outputs) — https://github.com/robinschmid/microbe_masst
- **metadataMASST** (Web app used to aggregate and visualize outputs from multiple domainMASST searches) — https://masst.gnps2.org/metadatamasst/

## Examples

```
python jobs.py --input spectra.mgf --output_prefix drug_analog --min_cosine 0.8 --min_matches 6 --skip_existing True
```

## Evaluation signals

- Verify that all required output files (_matches.tsv, _library.tsv, _datasets.tsv, _count_<domain>.tsv, _<domain>.html, _<domain>.json) are generated for every input spectrum
- Check that each reported match in _matches.tsv satisfies the configured minimum cosine score and minimum matching-peak thresholds
- Confirm the drug-analog pair table contains pairs meeting modified cosine ≥ 0.8 and ≥ 6 matched ions, consistent with the article's thresholds
- Re-run jobs.py with skip_existing=True until no new outputs appear to confirm no spectra were dropped due to Fast Search API failures
- Verify Python 3.10 was used, as required by the microbe_masst README

## Limitations

- Fast Search API entries can fail intermittently, requiring multiple re-runs with skip_existing=True to capture all matches
- Only Python 3.10 is supported
- Search is limited to datasets currently indexed in GNPS/MassIVE, Metabolomics Workbench, MetaboLights, and NORMAN — coverage depends on the index at run time
- Analog search requires setting a cosine/matching-peak threshold appropriate to the research question; overly permissive thresholds inflate false analog pairs
- The article notes that existing MS/MS reference libraries capture only the parent drug, so analog propagation is essential but sensitive to the completeness of the drug reference library

## Evidence

- [readme] Running jobs.py allows users to leverage the Fast Search API and execute a batch search of multiple MS/MS spectra against the current indexed data in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN and generate multiple outputs for all listed domainMASSTs simultaneously.: [evidence span withheld — anchor status: external_doc]
- [results] Using MS/MS spectral alignment via repository-scale molecular networking and fastMASST with analog search across MetaboLights, Metabolomics Workbench, and GNPS/MassIVE (~3500 LC-MS/MS datasets), the authors retrieved analogous MS/MS spectra for 24.6% of the 103,209 drug/drug-metabolite reference spectra, yielding over 19 million drug-analog spectral pairs.: [evidence span withheld — anchor status: not_found]
- [readme] A _library.tsv file will be generated. This contains a list of spectra from the GNPS libraries found to match your spectrum of interest. This enables a Level 2 annotation according the Metabolomics Standards Initiative.: [evidence span withheld — anchor status: external_doc]
- [readme] Make sure to run jobs.py a couple of times, until no new output is generated by having the option: skip_existing=True. Due to the Fast Search API some of the entries will fail.: [evidence span withheld — anchor status: external_doc]
- [readme] Check and adjust the different parameters for the search, such as minimum cosine score, mz tolerance, and number of minimum matching peaks based on your research question.: [evidence span withheld — anchor status: external_doc]
