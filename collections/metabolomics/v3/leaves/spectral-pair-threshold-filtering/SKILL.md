---
name: spectral-pair-threshold-filtering
description: Use when batch-searching reference MS/MS spectra against repository-scale
  untargeted metabolomics datasets and need to discard weak analog matches before
  building a drug-analog spectral pair table.
when_to_use_negative:
- When only parent-drug identification (not analog/metabolite propagation) is needed
  and no analog spectral search is required
- When the input spectra have already been matched and threshold-filtered (e.g., a
  pre-existing drug-analog pair table is available)
- When using the standalone single-spectrum MASST web apps rather than batch searching
  multiple spectra across repositories
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: microbeMASST (domainMASST batch search)
  role: Batch execution of FastMASST-style analog searches via the Fast Search API
    against multiple public metabolomics repositories with configurable cosine/mz/ion
    thresholds
  repo: https://github.com/robinschmid/microbe_masst
- name: GNPS Drug Library
  role: Source of 103,209 drug and drug-metabolite reference MS/MS spectra used as
    queries in the threshold-filtered analog search
  repo: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
- name: Fast Search API (fasst.gnps2.org)
  role: Backend service executing the modified cosine scoring against indexed GNPS/MassIVE,
    Metabolomics Workbench, MetaboLights, and NORMAN data
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:2ba3cbf91e748ad579e788f84dc717c2b4a68bbe94d3084c37f72d77098b1755
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/robinschmid/microbe_masst
  tools:
  - Fast Search API (fasst.gnps2.org)
  - GNPS Drug Library
  - microbeMASST (domainMASST batch search)
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# spectral-pair-threshold-filtering

## Summary

Filters drug-analog spectral matches by requiring a minimum modified cosine score of 0.8 and at least 6 matched ions, ensuring only high-confidence reference-spectrum-to-dataset-peak pairs are retained for downstream drug exposure analysis.

## When to use

Use when batch-searching reference MS/MS spectra against repository-scale untargeted metabolomics datasets and need to discard weak analog matches before building a drug-analog spectral pair table.

## When NOT to use

- When only parent-drug identification (not analog/metabolite propagation) is needed and no analog spectral search is required
- When the input spectra have already been matched and threshold-filtered (e.g., a pre-existing drug-analog pair table is available)
- When using the standalone single-spectrum MASST web apps rather than batch searching multiple spectra across repositories

## Inputs

- Reference MS/MS spectra from the GNPS Drug Library (drug and drug-metabolite reference spectra)
- Batch of .mgf files generated via MZmine from GNPS molecular networking workflows, or .csv/.tsv files containing USIs
- Fast Search API parameters: minimum cosine score, m/z tolerance, and minimum number of matching peaks

## Outputs

- drug_analog_spectral_pairs.tsv — a structured table of reference-spectrum-to-dataset-peak pairs passing the modified cosine >= 0.8 and >= 6 matched-ions thresholds
- A _matches.tsv file listing all scans found to match the searched spectrum in the indexed data
- A _library.tsv file listing GNPS library spectra matching the searched spectrum (Level 2 Metabolomics Standards Initiative annotation)
- A _datasets.tsv file with counts of unique samples matching the searched spectrum in each indexed dataset
- Interactive HTML trees (e.g., _domain.html) and JSON tree files (e.g., _microbe.json) for domain-specific MASST outputs
- Series of _count_domain.tsv files containing match counts for each specific domain MASST

## How to apply

Run a FastMASST-style analog search using the Fast Search API against indexed data from GNPS/MassIVE, Metabolomics Workbench, MetaboLights, and NORMAN, scoring each reference spectrum against dataset peaks with modified cosine. Apply the threshold filter: retain only pairs with modified cosine score >= 0.8 and at least 6 matched ions. Configure the minimum cosine score, m/z tolerance, and minimum matching peaks parameters in the jobs.py batch script according to the drug analog search needs. Because the Fast Search API can intermittently fail on some entries, re-run the batch with skip_existing=True a couple of times until no new matches are emitted, ensuring threshold filtering is applied to the complete result set. Output pairs passing the filter into a structured drug-analog spectral pair table keyed by the metabolomics USI and dataset of origin.

## Related tools

- **microbeMASST (domainMASST batch search)** (Batch execution of FastMASST-style analog searches via the Fast Search API against multiple public metabolomics repositories with configurable cosine/mz/ion thresholds) — https://github.com/robinschmid/microbe_masst
- **GNPS Drug Library** (Source of 103,209 drug and drug-metabolite reference MS/MS spectra used as queries in the threshold-filtered analog search) — https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
- **Fast Search API (fasst.gnps2.org)** (Backend service executing the modified cosine scoring against indexed GNPS/MassIVE, Metabolomics Workbench, MetaboLights, and NORMAN data)

## Examples

```
python code/jobs.py --input_directory input_mgf/ --output_directory out/  # configure min cosine=0.8, min matched peaks=6, mz tolerance in jobs.py
```

## Evaluation signals

- Verify every emitted spectral pair satisfies modified cosine >= 0.8 AND >= 6 matched ions; any pair below either threshold indicates filter misconfiguration
- Confirm the drug_analog_spectral_pairs.tsv contains mappings across the searched datasets (~3500 LC-MS/MS datasets) and that each pair references a valid USI and dataset identifier
- Re-run jobs.py with skip_existing=True until no new output is generated; residual new matches after a rerun indicate the API did not complete and threshold filtering is not yet exhaustive
- Check the _library.tsv output to confirm matched spectra correspond to GNPS library entries, supporting Level 2 Metabolomics Standards Initiative annotation consistency

## Limitations

- The Fast Search API can fail on some batch entries, requiring multiple re-runs with skip_existing=True before threshold filtering captures all possible matches
- Requires Python 3.10; using a different Python version may break the jobs.py batch pipeline
- Threshold values (cosine >= 0.8, 6 matched ions) are configurable and must be adjusted to the research question; defaults may not be optimal for all analog-propagation use cases
- The _matches.tsv output includes samples not part of the curated domain-specific MASSTs, so additional filtering may be needed downstream

## Evidence

- [methods] For each reference spectrum, run a batch FastMASST-style analog search against every dataset using modified cosine scoring with a minimum score of 0.8 and at least 6 matched ions.: [evidence span withheld — anchor status: not_found]
- [methods] Collect all reference-spectrum-to-dataset-peak pairs passing the threshold into a single drug-analog spectral pair table.: [evidence span withheld — anchor status: not_found]
- [readme] Check and adjust the different parameters for the search, such as minimum cosine score, mz tolerance, and number of minimum matching peaks based on your research question.: [evidence span withheld — anchor status: external_doc]
- [readme] Make sure to run [jobs.py] a couple of times, until no new output is generated by having the option: skip_existing=True.: [evidence span withheld — anchor status: not_found]
- [methods] Validation: confirm that the output table contains the expected drug-analog pair mappings across the searched datasets, with each pair satisfying the modified cosine >= 0.8 and >= 6 matched-ions thresholds.: [evidence span withheld — anchor status: not_found]
