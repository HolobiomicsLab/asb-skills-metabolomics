---
name: gnps-molecular-networking-output-parsing
description: Use when preparing individual or lists of MS/MS spectra as USI-formatted
  .csv/.tsv entries or MZmine .mgf output to run against microbeMASST/domainMASSTs
  via the Fast Search API.
when_to_use_negative:
- Input is already a processed feature table or abundance matrix rather than raw MS/MS
  spectra needing USI/.mgf representation
- Spectra are already submitted and matched outputs (_matches.tsv) exist for all entries
  — skip parsing and re-run only with skip_existing=True
- Working outside the Python 3.10 environment that jobs.py requires
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: microbeMASST
  role: Downstream consumer of the parsed USI/.mgf spectrum entries, which are batch-searched
    against microbial monoculture reference data
  repo: https://masst.gnps2.org/microbemasst/
- name: Fast Search API
  role: Executes the batch search of the parsed MS/MS spectra against indexed GNPS/MassIVE,
    Metabolomics Workbench, Metabolights, and NORMAN datasets
- name: MZmine
  role: Generates the .mgf input file containing the spectra to be parsed and submitted
    for MASST search
  repo: https://github.com/mzmine/mzmine
- name: jobs.py
  role: Batch runner that reads the parsed input files from its files list and invokes
    the Fast Search API per spectrum
  repo: https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:32c4fa8c793dbf58bacabc31564163e1af77f5d4d76a651886dd537da733a2e0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fast Search API
  - MZmine
  - jobs.py
  - microbeMASST
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# GNPS molecular networking output parsing

## Summary

Parses MS/MS spectrum inputs for GNPS-based MASST batch searches by extracting USIs from .csv/.tsv files or parsing MZmine-generated .mgf files, providing the per-spectrum entries that jobs.py submits to the Fast Search API.

## When to use

Use when preparing individual or lists of MS/MS spectra as USI-formatted .csv/.tsv entries or MZmine .mgf output to run against microbeMASST/domainMASSTs via the Fast Search API.

## When NOT to use

- Input is already a processed feature table or abundance matrix rather than raw MS/MS spectra needing USI/.mgf representation
- Spectra are already submitted and matched outputs (_matches.tsv) exist for all entries — skip parsing and re-run only with skip_existing=True
- Working outside the Python 3.10 environment that jobs.py requires

## Inputs

- USI-formatted spectrum list in .csv or .tsv format
- Single .mgf file generated via MZmine (molecular networking in GNPS workflow)
- jobs.py files list entries as ('input_directory/input_file', 'output_directory/output_prefix')

## Outputs

- Per-spectrum Fast Search API query entries consumed by microbeMASST/domainMASST batch search
- Parsed spectrum records (precursor m/z, fragment peaks) for scoring against indexed GNPS/MassIVE spectra

## How to apply

Represent each MS/MS spectrum of interest either as a single-entry list in USI format saved as a .csv or .tsv file, or collect all spectra from an LC-MS/MS run into a single .mgf file generated via MZmine from the molecular networking in GNPS workflow. Verify each entry carries the USI or the MZmine-derived scan attributes (precursor m/z, peaks, charge) needed by the Fast Search Tool for modified cosine similarity scoring. Add file entries to the files list in jobs.py as ('input_directory/input_file', 'output_directory/output_prefix'), optionally setting skip_existing=True so that re-runs only process spectra whose Fast Search API calls previously failed. The parsed spectra are then batch-searched against currently indexed GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN data, and results are written per-spectrum to _matches.tsv, _library.tsv, and _datasets.tsv outputs.

## Related tools

- **microbeMASST** (Downstream consumer of the parsed USI/.mgf spectrum entries, which are batch-searched against microbial monoculture reference data) — https://masst.gnps2.org/microbemasst/
- **Fast Search API** (Executes the batch search of the parsed MS/MS spectra against indexed GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN datasets)
- **MZmine** (Generates the .mgf input file containing the spectra to be parsed and submitted for MASST search) — https://github.com/mzmine/mzmine
- **jobs.py** (Batch runner that reads the parsed input files from its files list and invokes the Fast Search API per spectrum) — https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py

## Examples

```
python jobs.py  # files list entries like ('input_directory/input_file', 'output_directory/output_prefix') with skip_existing=True
```

## Evaluation signals

- Every spectrum in the input .csv/.tsv or .mgf file produces a corresponding entry in the _matches.tsv output — no empty result sets due to silent Fast Search API failures
- Re-running jobs.py with skip_existing=True generates no new output files after consecutive runs, confirming all parsed spectra were searched
- The _library.tsv file returns GNPS library matches enabling Level 2 annotation for the searched spectra, indicating spectra were parsed with sufficient peak/m/z information
- All output prefixes (_matches.tsv, _library.tsv, _datasets.tsv, _microbe.html, _microbe.json) are present for each entry in the jobs.py files list

## Limitations

- The Fast Search API can silently fail for some entries, so jobs.py must be run a couple of times with skip_existing=True until no new output is generated
- Only single .mgf files from MZmine or USI lists in .csv/.tsv are supported as inputs
- Python 3.10 is required to run the jobs.py batch runner

## Evidence

- [readme] You can run either a single .mgf file generated via [MZmine](https://github.com/mzmine/mzmine), from the molecular networking in GNPS workflow, or a list of [USIs](https://www.nature.com/articles/s41592-021-01184-6) provided either via a .csv or .tsv file.: "You can run either a single .mgf file generated via [MZmine](https://github.com/mzmine/mzmine), from the molecular networking in GNPS workflow, or a list of"
- [readme] Make sure to run [jobs.py] **_a couple of times_**, until no new output is generated by having the option: `skip_existing=True`: "Make sure to run [jobs.py] **_a couple of times_**, until no new output is generated by having the option: `skip_existing=True`. Due to the Fast Search API some of the entries will fail."
- [readme] A _matches.tsv file will be generated. This contains all the scans found to match your searched spectrum of interest in the data that have been currently indexed.: "A _matches.tsv file will be generated. This contains all the scans found to match your searched spectrum of interest in the data that have been currently indexed."
- [readme] This contains a list of spectra from the [GNPS libraries](https://library.gnps2.org/) found to match your spectrum of interest. This enables a Level 2 annotation according the Metabolomics Standards Initiative.: "This contains a list of spectra from the [GNPS libraries](https://library.gnps2.org/) found to match your spectrum of interest. This enables a Level 2 annotation according the Metabolomics Standards"
