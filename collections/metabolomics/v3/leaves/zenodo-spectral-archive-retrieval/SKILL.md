---
name: zenodo-spectral-archive-retrieval
description: Use when a downstream task needs the curated Spectraverse MS/MS library
  itself, or must independently verify the reported library statistics (488,630 spectra,
  44,237 unique compounds) from the deposited archive.
when_to_use_negative:
- The task concerns MassSpecGym, NPLIB1, or another spectral library rather than the
  Spectraverse Zenodo release v1.0.1.
- The spectra to be counted are raw instrument files (mzML/mzXML) rather than the
  curated, deposited MGF library files.
- The goal is to re-run the curation pipeline itself (e.g., library_cleaning_1.yaml
  / library_cleaning_2.yaml filters) rather than audit the final deposited statistics.
edam_operation: http://edamontology.org/operation_3283
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Parses MGF-format MS/MS library records into spectrum objects so counts and
    structure annotations (InChIKey/SMILES) can be extracted
  repo: https://github.com/matchms/matchms
- name: Python
  role: Runtime environment (3.10 or higher) in which the archive is parsed and counts
    computed
- name: conda
  role: Environment manager used to create the spectraverse environment from environment.yml
    before parsing
- name: Spectraverse-analysis
  role: Reference preprocessing repository whose environment setup and MGF data-directory
    conventions this retrieval/verification procedure follows
  repo: https://github.com/skinniderlab/spectraverse-analysis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:86a294ca6ad66a65e5acf48e8d99e8d5fcb59c48fe9a38175cca7393b7e7452f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - Spectraverse-analysis
  - conda
  - matchms
  edam_operation: http://edamontology.org/operation_3283
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# zenodo-spectral-archive-retrieval

## Summary

Retrieve the Spectraverse v1.0.1 MS/MS reference library (488,630 spectra over 44,237 unique small molecules) from its Zenodo DOI and verify counts by parsing the MGF-format library files with matchms.

## When to use

Use when a downstream task needs the curated Spectraverse MS/MS library itself, or must independently verify the reported library statistics (488,630 spectra, 44,237 unique compounds) from the deposited archive.

## When NOT to use

- The task concerns MassSpecGym, NPLIB1, or another spectral library rather than the Spectraverse Zenodo release v1.0.1.
- The spectra to be counted are raw instrument files (mzML/mzXML) rather than the curated, deposited MGF library files.
- The goal is to re-run the curation pipeline itself (e.g., library_cleaning_1.yaml / library_cleaning_2.yaml filters) rather than audit the final deposited statistics.

## Inputs

- Spectraverse v1.0.1 archive downloaded from Zenodo DOI 10.5281/zenodo.17870921
- MGF-format MS/MS library files extracted from the archive

## Outputs

- summary_statistics.csv containing the verified total spectrum count and unique-compound count
- Parsed matchms spectrum objects for downstream filtering or similarity workflows

## How to apply

Download the Spectraverse v1.0.1 archive from Zenodo (https://doi.org/10.5281/zenodo.17870921) and extract the library files. Parse all MS/MS records from the MGF-format library files into matchms spectrum objects under Python 3.10+ (set up via conda env create -f environment.yml; conda activate spectraverse). Count total parsed spectra records, then extract the standardized structure annotations (InChIKey/SMILES) from each record and de-duplicate on the structure identifier to count unique small molecules. Compare both computed counts against the reported totals of 488,630 spectra and 44,237 unique small molecules; any discrepancy flags a parsing or counting error rather than a genuine mismatch with the paper. Write the verified counts to summary_statistics.csv so the audit is reproducible.

## Related tools

- **matchms** (Parses MGF-format MS/MS library records into spectrum objects so counts and structure annotations (InChIKey/SMILES) can be extracted) — https://github.com/matchms/matchms
- **Python** (Runtime environment (3.10 or higher) in which the archive is parsed and counts computed)
- **conda** (Environment manager used to create the spectraverse environment from environment.yml before parsing)
- **Spectraverse-analysis** (Reference preprocessing repository whose environment setup and MGF data-directory conventions this retrieval/verification procedure follows) — https://github.com/skinniderlab/spectraverse-analysis

## Examples

```
conda env create -f environment.yml; conda activate spectraverse; python -c "from matchms.importing import load_from_mgf; spectra = list(load_from_mgf('data/spectraverse.mgf')); print(len(spectra))"
```

## Evaluation signals

- Total parsed spectrum count equals 488,630, matching the value reported in the paper.
- Unique-compound count, de-duplicated on the standardized structure identifier (InChIKey/SMILES), equals 44,237.
- Every parsed record carries a structure annotation; records with missing InChIKey/SMILES are investigated before deduplication.
- Downloaded archive resolves at DOI 10.5281/zenodo.17870921 and reports version v1.0.1.
- Counts are recorded in summary_statistics.csv and a discrepancy triggers re-parsing rather than silent acceptance.

## Limitations

- The reported counts apply to Spectraverse v1.0.1; future releases are planned but per-release DOIs with semantic versioning are not yet available.
- Counts are the final post-curation totals; spectra removed by the pipeline's cleaning filters (low-resolution, zero/uniform intensity fragments, unwanted adducts, near-duplicates) are not present in the archive and cannot be tallied from it.
- A count mismatch indicates a parsing or counting error on the auditor's side, not necessarily an error in the deposited library — parsing must be rechecked before drawing conclusions.

## Evidence

- [intro] Spectraverse contains 488,630 MS/MS spectra from 44,237 unique small molecules and is freely available from Zenodo at https://doi.org/10.5281/zenodo.17870921.: "Spectraverse contains 488,630 MS/MS spectra from 44,237 unique small
molecules and is freely available from Zenodo at https://doi.org/10.5281/zenodo.17870921."
- [discussion] The current release of Spectraverse (v1.0.1) is archived on Zenodo (https://doi.org/10.5281/zenodo.17870921): "The current release of Spectraverse (v1.0.1) is archived on Zenodo (https://doi.org/10.5281/zenodo.17870921)"
- [methods] Place raw data files (MGF format) in the `data/` directory.: "Prepare your data: Place raw data files (MGF format) in the `data/` directory."
- [readme] Python 3.10 or higher: "### Prerequisites

- Python 3.10 or higher
- pip or conda package manager"
- [readme] conda env create -f environment.yml; conda activate spectraverse: "conda env create -f environment.yml
   conda activate spectraverse"
