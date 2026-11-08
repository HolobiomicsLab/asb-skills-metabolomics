---
name: mzml-spectral-data-parsing
description: Use when raw profile LC-DTIM-MS data in mzML format must be parsed and
  searched for predicted primary-ion m/z species from molecular formulae prior to
  RT-dimension peak picking.
when_to_use_negative:
- Input is already a peak table or feature list rather than raw profile mzML data
  from LC-DTIM-MS
- Data were acquired on non-drift-tube instruments (CCSfind targets drift tube IM-MS
  data with single-field CCS calibration coefficients Tfix and Beta and a defined
  drift gas)
- Only one-off interactive exploration is needed and the compulsory inputs (molecular
  formulae list, CCS calibration coefficients, drift gas identity, primary ion species)
  are unavailable
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CCSfind
  role: Main tool that parses mzML files and searches monoisotopic primary-ion m/z
    traces for targeted molecular formulae
  repo: https://github.com/sangeeta97/ccs_find.git
- name: polars
  role: Holds the parsed mzML data in lazy frames for memory-efficient m/z trace extraction
- name: tidyms
  role: Provides the CentWave implementation used for peak picking of extracted ion
    traces in the retention-time dimension
- name: conda
  role: Creates the required Python 3.9 environment for running the processor
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/jms.5040
    title: 'CCSfind: A tool for chemically informed LC‐IM‐MS database building'
  source_content_hash: sha256:0ed606e89f5a1c6289660bcffed7daa1773c0863cd520e6e460e0f08d4b4886c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sangeeta97/ccs_find
  repo_url: https://github.com/sangeeta97/ccs_find.git
  tools:
  - CCSfind
  - Python
  - conda
  - numpy
  - pandas
  - pip
  - polars
  - tidyms
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# mzml-spectral-data-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parse open-source mzML profile data from LC-DTIM-MS experiments into memory-efficient polars lazy frames so targeted primary-ion m/z traces can be extracted and peak-picked in the RT dimension with tidyms.

## When to use

Use when raw profile LC-DTIM-MS data in mzML format must be parsed and searched for predicted primary-ion m/z species from molecular formulae prior to RT-dimension peak picking.

## When NOT to use

- Input is already a peak table or feature list rather than raw profile mzML data from LC-DTIM-MS
- Data were acquired on non-drift-tube instruments (CCSfind targets drift tube IM-MS data with single-field CCS calibration coefficients Tfix and Beta and a defined drift gas)
- Only one-off interactive exploration is needed and the compulsory inputs (molecular formulae list, CCS calibration coefficients, drift gas identity, primary ion species) are unavailable

## Inputs

- Profile data mzML input file from LC-DTIM-MS (up to 1 GB)
- List of molecular formulae to search in the mzML file
- config.ini with analysis parameters (mass extraction window, expected LC peak width, peak abundance settings)

## Outputs

- peak_table.csv with confirmed m/z traces and their RT values
- In-memory parsed mzML representation (polars lazy frames) supporting faster re-runs

## How to apply

Set up a Python 3.9 environment (via conda or pip with requirements.txt), since CCSfind requires this exact version. Parse the profile mzML input file into polars lazy frames for memory-efficient processing, then search each monoisotopic primary-ion m/z derived from the input molecular formulae against the parsed spectra, optionally enabling the search for multiply charged ions. Extract each primary-ion m/z trace over retention time and arrival time with intensities, and peak-pick the traces in the RT dimension using the tidyms CentWave implementation. Assemble confirmed m/z traces with RT values into peak_table.csv. The design allows re-running the analysis faster on an already parsed mzML file with different settings or additional molecular formulae.

## Related tools

- **CCSfind** (Main tool that parses mzML files and searches monoisotopic primary-ion m/z traces for targeted molecular formulae) — https://github.com/sangeeta97/ccs_find.git
- **polars** (Holds the parsed mzML data in lazy frames for memory-efficient m/z trace extraction)
- **tidyms** (Provides the CentWave implementation used for peak picking of extracted ion traces in the retention-time dimension)
- **conda** (Creates the required Python 3.9 environment for running the processor)

## Examples

```
conda create --name ccs_find python=3.9; pip install -r requirements.txt; python processor.py -g config.ini
```

## Evaluation signals

- The run completes via processor.py with the -g config.ini flag and a readable peak_table.csv is produced
- mzML files of up to 1 GB are processed in less than 2 min on a desktop computer with 32 GB RAM
- Computational time scales linearly with input mzML file size or number of input molecular formulae
- Extracted traces correspond to the monoisotopic primary-ion m/z values derived from the input molecular formulae, including any multiply charged ions if enabled
- Peak picking in the RT dimension succeeds on extracted ion traces (tidyms CentWave) before downstream isotopologue matching confirmation

## Limitations

- Python 3.9 is a must; other versions are not supported
- Requires profile data mzML from LC-DTIM-MS plus compulsory inputs: molecular formulae list, single-field CCS calibration coefficients (Tfix and Beta), drift gas identity, and primary ion species
- Peak picking must be followed by filtering using isotopologue matching confirmation before results are trusted for database entries

## Evidence
