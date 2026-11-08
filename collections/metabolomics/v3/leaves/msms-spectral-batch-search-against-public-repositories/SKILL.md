---
name: msms-spectral-batch-search-against-public-repositories
description: Use when you have multiple MS/MS spectra (e.g. a .mgf from MZmine feature-based
  networking, or a USI .csv/.tsv list) and want to find which public datasets and
  taxonomic lineages they match.
when_to_use_negative:
- The input is a single spectrum for a one-off lookup — use the standalone web apps
  (e.g. microbeMASST at masst.gnps2.org/microbemasst) instead of the batch jobs.py
  pipeline.
- The input is raw instrument data not yet processed — spectra must first be deconvoluted
  into a .mgf via MZmine or supplied as USIs.
- The analysis requires only sample-level feature statistics (peak areas, blank subtraction)
  rather than repository-wide spectrum-to-dataset matching.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: jobs.py (Fast Search API)
  role: 'Drives the batch search: submits the .mgf or USI list via the Fast Search
    API (fasst.gnps2.org/fastsearch) and orchestrates all output generation'
  repo: https://github.com/robinschmid/microbe_masst
- name: MZmine
  role: Generates the single .mgf input file from raw LC-MS/MS data via the GNPS molecular
    networking workflow
  repo: https://github.com/mzmine/mzmine
- name: microbeMASST / plantMASST / tissueMASST / microbiomeMASST / foodMASST / metadataMASST
  role: Domain-specific MASSTs whose per-domain outputs (_count_domain.tsv, _domain.html
    trees) and aggregated visualization are generated from the batch search results
  repo: https://masst.gnps2.org/metadatamasst/
- name: GNPS_MASST
  role: Standalone web applications for searching one spectrum at a time (alternative
    to batch jobs.py)
  repo: https://github.com/mwang87/GNPS_MASST
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:815f10076e581046fa6f31986bb10f31961d348e51195a2db84ecc6144670228
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
  - jobs.py (Fast Search API)
  - metadataMASST
  - microbeMASST / plantMASST / tissueMASST / microbiomeMASST / foodMASST / metadataMASST
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# MS/MS spectral batch search against public repositories

## Summary

Executes a batch spectral search of many MS/MS spectra (.mgf or USI list) via the Fast Search API (fasst.gnps2.org/fastsearch) against currently indexed data in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN, generating per-spectrum match tables plus interactive domain trees (microbe/plant/tissue/microbiome/food).

## When to use

Use when you have multiple MS/MS spectra (e.g. a .mgf from MZmine feature-based networking, or a USI .csv/.tsv list) and want to find which public datasets and taxonomic lineages they match.

## When NOT to use

- The input is a single spectrum for a one-off lookup — use the standalone web apps (e.g. microbeMASST at masst.gnps2.org/microbemasst) instead of the batch jobs.py pipeline.
- The input is raw instrument data not yet processed — spectra must first be deconvoluted into a .mgf via MZmine or supplied as USIs.
- The analysis requires only sample-level feature statistics (peak areas, blank subtraction) rather than repository-wide spectrum-to-dataset matching.

## Inputs

- Single .mgf file generated via MZmine from the GNPS molecular networking workflow
- List of Universal Spectrum Identifiers (USIs) as a .csv or .tsv file
- Search parameters: minimum cosine score, mz tolerance, number of minimum matching peaks
- Python 3.10 environment

## Outputs

- Interactive HTML trees per domain-specific MASST (_domain.html, e.g. _microbe.html)
- JSON tree files per domain (_microbe.json etc.)
- _matches.tsv: all scans matching each searched spectrum in currently indexed data
- _library.tsv: GNPS library spectra matching the spectrum of interest (MSI Level 2 annotation)
- _datasets.tsv: number of unique matching samples per currently indexed dataset
- _count_domain.tsv series: matches per specific domain MASST

## How to apply

Add input entries to jobs.py as ('input_directory/input_file','output_directory/output_prefix'), accepting either a single .mgf file generated via MZmine from a GNPS molecular networking workflow or a list of USIs provided as .csv/.tsv. Check and adjust search parameters — minimum cosine score, mz tolerance, and number of minimum matching peaks — before running, since these directly control match stringency and should be tuned to the research question. Run jobs.py through Python 3.10 against the Fast Search API (fasst.gnps2.org/fastsearch); because some Fast Search API entries fail intermittently, re-run jobs.py a couple of times with skip_existing=True until no new output is generated so all possible matches are captured. Collect the six output families: interactive _domain.html trees and _domain.json per domain-specific MASST, _matches.tsv (all scans matching each searched spectrum, including samples outside curated domain MASSTs), _library.tsv (GNPS library spectra matches enabling MSI Level 2 annotation), _datasets.tsv (unique matching samples per indexed dataset), and _count_domain.tsv (per-domain match counts). Optionally visualize aggregated outputs via metadataMASST.

## Related tools

- **jobs.py (Fast Search API)** (Drives the batch search: submits the .mgf or USI list via the Fast Search API (fasst.gnps2.org/fastsearch) and orchestrates all output generation) — https://github.com/robinschmid/microbe_masst
- **MZmine** (Generates the single .mgf input file from raw LC-MS/MS data via the GNPS molecular networking workflow) — https://github.com/mzmine/mzmine
- **microbeMASST / plantMASST / tissueMASST / microbiomeMASST / foodMASST / metadataMASST** (Domain-specific MASSTs whose per-domain outputs (_count_domain.tsv, _domain.html trees) and aggregated visualization are generated from the batch search results) — https://masst.gnps2.org/metadatamasst/
- **GNPS_MASST** (Standalone web applications for searching one spectrum at a time (alternative to batch jobs.py)) — https://github.com/mwang87/GNPS_MASST

## Examples

```
python code/jobs.py  # entries as ("input/input.mgf", "output/prefix"); re-run with skip_existing=True until no new outputs
```

## Evaluation signals

- All _matches.tsv, _library.tsv, and _datasets.tsv files exist for every input entry listed in jobs.py.
- Re-running jobs.py with skip_existing=True produces no new outputs, indicating the batch search completed across all entries.
- _library.tsv contains GNPS library matches consistent with MSI Level 2 annotation for spectra of interest.
- _datasets.tsv counts report plausible unique-sample totals per indexed dataset (GNPS/MassIVE, Metabolomics Workbench, Metabolights, NORMAN).
- _count_domain.tsv totals per domain are consistent with the interactive _domain.html trees.

## Limitations

- Some entries fail on a single run due to the Fast Search API; sequential re-runs with skip_existing=True are required to catch all possible matches.
- Search outputs include samples that are not part of the curated domain-specific MASSTs, so _matches.tsv requires additional filtering for domain-restricted conclusions.
- Python 3.10 must be used; other versions are unsupported by jobs.py.
- Tuning of minimum cosine score, mz tolerance, and minimum matching peaks is left to the user and materially affects match recall/precision.

## Evidence

- [methods] batch search spans methods+readme: [evidence span withheld — anchor status: not_found]
- [readme] README execute batch run: "Check and adjust the different parameters for the search, such as minimum cosine score, mz tolerance, and number of minimum matching peaks"
- [readme] README re-run instruction: "Make sure to run [jobs.py] **_a couple of times_**, until no new output is generated by having the option: `skip_existing=True`. Due to the Fast Search API some of the entries will fail"
- [readme] README outputs list: "A _matches.tsv file will be generated. This contains all the scans found to match your searched spectrum of interest in the data that have been currently indexed"
- [readme] README library annotation: [evidence span withheld — anchor status: not_found]
- [readme] README input formats: [evidence span withheld — anchor status: not_found]
- [readme] README domain outputs: "A series of interactive HTML trees files will be generated for each domain-specific MASST ending with _domain.html"
