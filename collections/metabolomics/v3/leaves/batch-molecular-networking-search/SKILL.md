---
name: batch-molecular-networking-search
description: Use when you need to find where reference MS/MS spectra (drug, metabolite,
  or compound library entries) appear as analogs across thousands of indexed LC-MS/MS
  datasets in public repositories such as GNPS/MassIVE, MetaboLights, and Metabolomics
  Workbench.
when_to_use_negative:
- When you already have a curated, dataset-level drug exposure table and only need
  pharmacologic interpretation (use the GNPS Drug Library metadata directly instead
  of re-searching)
- When the input is a single, already-annotated feature table where compound identities
  are known and no repository-scale analog propagation is needed
- When the goal is immediate clinical decision-making from untargeted metabolomics
  data — the resource is explicitly not intended for that purpose
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GNPS Fast Search API
  role: Backend API used to execute batch analog searches of reference MS/MS spectra
    against all indexed untargeted metabolomics datasets simultaneously
  repo: https://github.com/mwang87/GNPS_MASST
- name: microbe_masst (domainMASSTs)
  role: Repository containing jobs.py and domain-specific MASST tools (microbeMASST,
    plantMASST, tissueMASST, microbiomeMASST, foodMASST) for running batch searches
    and generating aggregated outputs
  repo: https://github.com/robinschmid/microbe_masst
- name: GNPS Drug Library
  role: The reference MS/MS spectral library of drugs and their metabolites/analogs
    whose spectra are propagated across public repositories via the batch search
  repo: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:e440316fe82d0be0d933791fb1099f0879ad9ebc3ead76cab6603f30b30f8657
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mwang87/GNPS_MASST
  tools:
  - GNPS (Global Natural Product Social Molecular Networking)
  - GNPS Drug Library
  - GNPS Fast Search API
  - microbe_masst (domainMASSTs)
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# batch-molecular-networking-search

## Summary

A batch spectral search workflow that propagates reference MS/MS spectra (e.g. the ~103,000 drug and drug-metabolite spectra in the GNPS Drug Library) across thousands of indexed untargeted metabolomics datasets using the Fast Search API, producing analog spectral pair tables that link reference compounds back to their occurrences in public repositories.

## When to use

Use when you need to find where reference MS/MS spectra (drug, metabolite, or compound library entries) appear as analogs across thousands of indexed LC-MS/MS datasets in public repositories such as GNPS/MassIVE, MetaboLights, and Metabolomics Workbench.

## When NOT to use

- When you already have a curated, dataset-level drug exposure table and only need pharmacologic interpretation (use the GNPS Drug Library metadata directly instead of re-searching)
- When the input is a single, already-annotated feature table where compound identities are known and no repository-scale analog propagation is needed
- When the goal is immediate clinical decision-making from untargeted metabolomics data — the resource is explicitly not intended for that purpose

## Inputs

- Reference MS/MS spectra (e.g. 103,209 drug and drug-metabolite spectra from the GNPS Drug Library)
- Single .mgf file of peak spectra generated via MZmine molecular networking
- List of USIs (Universal Spectrum Identifiers) in .csv or .tsv format
- Indexed repository datasets (~3500 public LC-MS/MS untargeted metabolomics datasets from GNPS/MassIVE, MetaboLights, Metabolomics Workbench, and NORMAN, accessed via the Fast Search API)

## Outputs

- Drug-analog spectral pair table (drug_analog_spectral_pairs.tsv) containing all reference-spectrum-to-dataset-peak pairs passing the modified cosine >= 0.8 and >= 6 matched-ion thresholds (>19 million pairs for the drug library)
- A _matches.tsv file with all scans matching the searched spectrum in indexed data
- A _library.tsv file with GNPS library spectra matching the searched spectrum (enables Level 2 Metabolomics Standards Initiative annotation)
- A _datasets.tsv file with the number of unique samples matching each searched spectrum per indexed dataset
- Interactive HTML tree files (e.g. _domain.html) and corresponding JSON files for domain-specific MASST visualizations
- Per-domain count files (_count_domain.tsv) summarizing matches for each domain MASST

## How to apply

Start from a set of reference MS/MS spectra — for the drug-exposure use case, the 103,209 drug and drug-metabolite spectra in the GNPS Drug Library. Using the Fast Search API (fasst.gnps2.org/fastsearch), run a batch analog search against the indexed data from ~3500 public LC-MS/MS untargeted metabolomics datasets. Use modified cosine scoring with a minimum score of 0.8 and at least 6 matched ions as the analog-match threshold; spectra can be submitted either as a single .mgf file generated via MZmine or as a list of USIs in a .csv or .tsv file. Because some searches fail through the API, re-run the batch job (with skip_existing=True) a couple of times until no new output is produced. The workflow yields >19 million drug-analog spectral pairs, each mapped to its parent metabolomics USI and dataset of origin, suitable for downstream propagation or exposure-readout analyses.

## Related tools

- **GNPS Fast Search API** (Backend API used to execute batch analog searches of reference MS/MS spectra against all indexed untargeted metabolomics datasets simultaneously) — https://github.com/mwang87/GNPS_MASST
- **microbe_masst (domainMASSTs)** (Repository containing jobs.py and domain-specific MASST tools (microbeMASST, plantMASST, tissueMASST, microbiomeMASST, foodMASST) for running batch searches and generating aggregated outputs) — https://github.com/robinschmid/microbe_masst
- **GNPS Drug Library** (The reference MS/MS spectral library of drugs and their metabolites/analogs whose spectra are propagated across public repositories via the batch search) — https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library

## Examples

```
python code/jobs.py  # edit jobs.py to add ("input.mgf", "output/drug_analog") entries and set modified cosine >= 0.8, >= 6 matched ions; run repeatedly with skip_existing=True
```

## Evaluation signals

- Verify that the output analog pair table satisfies the acceptance thresholds: every pair has modified cosine score >= 0.8 and at least 6 matched ions
- Confirm that the total number of drug-analog spectral pairs is on the order of >19 million when searching the full 103,209-spectrum GNPS Drug Library
- Check that each matched pair in the output is mapped to a valid parent metabolomics USI and its dataset of origin
- Re-run jobs.py with skip_existing=True until no new output is generated, confirming API-failure retries have completed
- Validate that the _library.tsv output enables Level 2 Metabolomics Standards Initiative annotation by checking that returned matches are GNPS library spectra

## Limitations

- Some entries fail on the first pass through the Fast Search API; the workflow must be re-run a couple of times (with skip_existing=True) to catch all possible matches
- The workflow requires Python 3.10 specifically
- The analog search only captures spectra that pass the cosine-score/ion-match thresholds; very divergent analogs or metabolites with insufficient spectral similarity will be missed
- The indexed data include samples that are not part of the curated domain-specific MASSTs, so matches may include out-of-scope datasets that require downstream filtering
- The GNPS Drug Library resource is not intended for immediate clinical decision-making

## Evidence

- [other] fastMASST with analog search across MetaboLights, Metabolomics Workbench, and GNPS/MassIVE (~3500 LC-MS/MS datasets): [evidence span withheld — anchor status: not_found]
- [other] modified cosine scoring with a minimum score of 0.8 and at least 6 matched ions: [evidence span withheld — anchor status: not_found]
- [readme] Fast Search API batch search description from README: [evidence span withheld — anchor status: not_found]
- [readme] USI input format and mgf option: [evidence span withheld — anchor status: not_found]
- [readme] Retry requirement due to API failures: [evidence span withheld — anchor status: not_found]
- [abstract] Resource not intended for clinical decision-making: "While not intended for immediate clinical decision-making, it supports data-driven exploration of drug exposures where traditional records are limited or unreliable."
