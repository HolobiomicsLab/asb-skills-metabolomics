---
name: msms-library-count-verification
description: Use when a deposited MS/MS reference library (MGF or matchms-readable
  format) carries reported totals of spectra and unique compounds that must be independently
  verified against the Zenodo archive before downstream use or benchmarking.
when_to_use_negative:
- Input spectra are raw instrument files (mzML/mzXML) rather than the already-curated
  library release; the pipeline's curation filters, not this counting verification,
  apply there
- The goal is quality filtering or de-duplication of the library (library_cleaning_1.yaml
  / library_cleaning_2.yaml matchms filters), not reproducing the final curated counts
- The library under test has no published totals to compare against, so there is no
  validation criterion for the count comparison step
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Parses the MGF-format Spectraverse library files into spectrum objects whose
    metadata (InChIKey/SMILES) and peak records are counted
  repo: https://github.com/matchms/matchms
- name: spectraverse-analysis
  role: Provides the preprocessing environment (environment.yml, conda env 'spectraverse')
    and code in which the counting/verification runs
  repo: https://github.com/skinniderlab/spectraverse-analysis
- name: Python
  role: Runtime (Python 3.10 or higher) for executing the parsing and counting steps
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:bef7410ed707a0ccb4e32217c1bc974565e67e10b11ce8a09470f0018bed179a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - matchms
  - spectraverse-analysis
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# msms-library-count-verification

## Summary

Reproduces the reported headline statistics of a curated MS/MS spectral library (here, Spectraverse's 488,630 spectra over 44,237 unique small molecules) by parsing the deposited MGF-format library files with matchms, counting total spectra records and de-duplicating structure annotations, then comparing computed counts to published totals.

## When to use

Use when a deposited MS/MS reference library (MGF or matchms-readable format) carries reported totals of spectra and unique compounds that must be independently verified against the Zenodo archive before downstream use or benchmarking.

## When NOT to use

- Input spectra are raw instrument files (mzML/mzXML) rather than the already-curated library release; the pipeline's curation filters, not this counting verification, apply there
- The goal is quality filtering or de-duplication of the library (library_cleaning_1.yaml / library_cleaning_2.yaml matchms filters), not reproducing the final curated counts
- The library under test has no published totals to compare against, so there is no validation criterion for the count comparison step

## Inputs

- Zenodo release archive of the Spectraverse library (v1.0.1, doi 10.5281/zenodo.17870921)
- MGF-format MS/MS library files extracted from the archive
- Per-record structure annotations (InChIKey/SMILES) in spectrum metadata

## Outputs

- Total count of parsed MS/MS spectra records
- Count of unique small molecules de-duplicated on standardized structure identifiers
- summary_statistics.csv with the verified counts

## How to apply

Download the library release archive from its stated DOI (Spectraverse v1.0.1, Zenodo doi 10.5281/zenodo.17870921) and extract the MGF-format library files. Under Python 3.10+, set up the environment per the spectraverse-analysis repository (conda env create -f environment.yml; conda activate spectraverse) and parse every MS/MS record with matchms. Count the total number of parsed spectrum objects, then extract the standardized structure annotations (InChIKey/SMILES) from each record's metadata and count unique compounds by de-duplicating on the structure identifier. Compare both computed values against the published totals of 488,630 spectra and 44,237 unique small molecules; any mismatch signals a parsing, filtering, or counting error rather than a true discrepancy in the resource.

## Related tools

- **matchms** (Parses the MGF-format Spectraverse library files into spectrum objects whose metadata (InChIKey/SMILES) and peak records are counted) — https://github.com/matchms/matchms
- **spectraverse-analysis** (Provides the preprocessing environment (environment.yml, conda env 'spectraverse') and code in which the counting/verification runs) — https://github.com/skinniderlab/spectraverse-analysis
- **Python** (Runtime (Python 3.10 or higher) for executing the parsing and counting steps)

## Examples

```
conda env create -f environment.yml; conda activate spectraverse; python -c "from matchms.importing import load_from_mgf; specs=list(load_from_mgf('data/spectraverse.mgf')); print(len(specs), len({s.get('inchikey') for s in specs}))"
```

## Evaluation signals

- Tabulated total spectrum count equals 488,630 exactly
- Unique-compound count (de-duplicated on InChIKey/SMILES) equals 44,237 exactly
- Number of parsed matchms spectrum objects matches the number of MGF entries before any filtering is applied
- summary_statistics.csv is written and contains both verified counts
- Discrepancies in either count are treated as flags of a parsing or counting error, not as resource-level differences

## Limitations

- Counts only verify the final curated release; completeness of near-duplicate spectra removal for Spectraverse itself is not explicitly stated in the paper
- Results depend on matchms parsing behavior and the modified matchms libraries shipped with the repository (matchms/matchms.zip replaces the original matchms libraries), so stock matchms versions may parse metadata differently
- Requires the exact Zenodo release (v1.0.1); future re-releases could change totals without a changelog

## Evidence

- [intro] Reported library totals and DOI: "Spectraverse contains 488,630 MS/MS spectra from 44,237 unique small
molecules and is freely available from Zenodo at https://doi.org/10.5281/zenodo.17870921."
- [discussion] Archive on Zenodo: "The current release of Spectraverse (v1.0.1) is archived on Zenodo (https://doi.org/10.5281/zenodo.17870921)"
- [methods] Python version requirement: "Python 3.10 or higher"
- [methods] MGF input data placement: "Place raw data files (MGF format) in the `data/` directory."
- [methods] Modified matchms libraries: "use the zip file at matchms/matchms.zip and replace it original matchms libraries"
- [readme] Repository purpose: "This repository contains code used to preprocess and harmonize MS/MS spectra and their associated metadata in the compilation of Spectraverse."
