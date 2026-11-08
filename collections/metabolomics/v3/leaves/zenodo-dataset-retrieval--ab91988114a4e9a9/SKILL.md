---
name: zenodo-dataset-retrieval
description: Use when a reproduction or audit task requires retrieving the deposited
  MSnLib spectral files from Zenodo record 11163380 to count MS2 spectra, merge/deduplicate
  MSn trees, and compute compound coverage against Extended Data Tables 2 and 3.
when_to_use_negative:
- When the goal is to acquire new raw MSn spectra rather than reprocess the already
  deposited and deduplicated MSnLib library files
- When the task requires converting Thermo RAW instrument files to mzML/MGF — that
  conversion is handled by ThermoRawFileParser, not by fetching the finished Zenodo
  deposit
- When only metadata cleanup or plate-map preparation is needed rather than library-level
  spectrum counting
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: pip
  role: Installs the Python dependencies required to run the reproduction workflow
- name: Prefect
  role: Orchestrates the metadata cleanup and library-building flows whose outputs
    form the Zenodo deposit
  repo: https://github.com/corinnabrungs/msn_tree_library
- name: mzmine
  role: Open-source pipeline used to acquire and process the high-throughput data
    underlying the MSnLib library
  repo: https://github.com/mzmine/mzmine
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:1c849150cc544b16c98c17b6e91114a0e1b598b7163b304ed4be8d3123b37d4c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/corinnabrungs/msn_tree_library
  tools:
  - Prefect
  - mzmine
  - pip
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# zenodo-dataset-retrieval

## Summary

Locate and programmatically download the MSnLib spectral library deposit from Zenodo (record 11163380) so the reported totals of 357,065 MS2 spectra, >2.3 million merged MSn spectra, and 87% compound coverage over 30,008 unique small molecules can be independently reproduced.

## When to use

Use when a reproduction or audit task requires retrieving the deposited MSnLib spectral files from Zenodo record 11163380 to count MS2 spectra, merge/deduplicate MSn trees, and compute compound coverage against Extended Data Tables 2 and 3.

## When NOT to use

- When the goal is to acquire new raw MSn spectra rather than reprocess the already deposited and deduplicated MSnLib library files
- When the task requires converting Thermo RAW instrument files to mzML/MGF — that conversion is handled by ThermoRawFileParser, not by fetching the finished Zenodo deposit
- When only metadata cleanup or plate-map preparation is needed rather than library-level spectrum counting

## Inputs

- Zenodo deposit files for record 11163380 (MSnLib merged and deduplicated MS2 and MSn spectral library)
- Compound metadata table linking spectra to the 30,008 unique small molecules (PubChem/LOTUS/DrugBank/DrugCentral identifiers from the metadata template)
- Extended Data Tables 2 and 3 of the MSnLib paper as the reference for expected totals

## Outputs

- library_totals.csv containing the reproduced MS2 spectrum count, merged MSn spectrum count, and compound coverage percentage
- Reproduced per-compound coverage tally over 30,008 unique small molecules

## How to apply

Fetch the deposit files for Zenodo record 11163380 (the MSnLib library) and load the spectral records with Python (pandas/pyteomics or equivalent). Parse each record and count all MS2-level spectra, then merge duplicate spectra across files and deduplicate to aggregate the multi-stage (MSn) tree records. Join the resulting spectral counts against the compound metadata (30,008 unique small molecules drawn from sources such as LOTUS, DrugBank, DrugCentral, and the Broad Drug Repurposing Hub listed in the msn_tree_library README) to compute the fraction of compounds with at least one spectrum. Reproduced totals should match the published figures: 357,065 MS2 spectra, >2.3 million merged MSn spectra, and 87% compound coverage, mirroring Extended Data Tables 2 and 3; write the summary as library_totals.csv.

## Related tools

- **pip** (Installs the Python dependencies required to run the reproduction workflow)
- **Prefect** (Orchestrates the metadata cleanup and library-building flows whose outputs form the Zenodo deposit) — https://github.com/corinnabrungs/msn_tree_library
- **mzmine** (Open-source pipeline used to acquire and process the high-throughput data underlying the MSnLib library) — https://github.com/mzmine/mzmine

## Examples

```
prefect deploy metadata_cleanup_prefect:cleanup_file --name local-deploy --pool local-work
```

## Evaluation signals

- Reproduced MS2 spectrum count equals exactly 357,065
- Merged and deduplicated MSn spectrum count exceeds 2.3 million
- Computed compound coverage equals 87% of the 30,008 unique small molecules
- library_totals.csv is produced with the MS2 count, merged MSn count, and coverage percentage
- Counts are reproducible after re-running the merge and deduplication step on the same Zenodo files

## Limitations

- External databases used for compound metadata (Dictionary of Natural Products, DrugBank) need a local file and/or special access; otherwise set them to False in jobs.py, which can affect the coverage denominator
- If no structure information is provided in the metadata, it is queried from PubChem by Name search, so ambiguous compound names can change which spectra map to which compound
- The validation targets (357,065 MS2 spectra, >2.3M MSn, 87%) reflect the deposited merged library; reprocessing raw acquisition output rather than the deposit will not match these totals

## Evidence

- [other] MS2 spectrum total 357,065: [evidence span withheld — anchor status: not_found]
- [abstract] Merged MSn spectra >2.3 million: "a machine learning-ready open resource of >2 million spectra in MS n trees of 30,008 unique small molecules"
- [readme] Compound metadata source per README: "If no structure information is provided, it is queried from PubChem by Name search."
- [readme] Database access caveat: "some need a local file and/or special access otherwise set it to False in the jobs.py"
- [abstract] Pipeline software underlying the library: "built with a high-throughput data acquisition and processing pipeline in the open-source software mzmine"
