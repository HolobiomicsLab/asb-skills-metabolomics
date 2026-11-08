---
name: chromatographic-trace-extraction
description: Use when you have a profile-mode mzML file from LC-DTIM-MS and a list
  of molecular formulae (or predicted ion m/z values) and need to locate each monoisotopic
  primary-ion m/z trace over RT and arrival time, then peak-pick in the RT dimension.
when_to_use_negative:
- Input is already a peak/feature table rather than raw profile-mode mzML — trace
  extraction requires raw spectra
- Data are not LC-IM-MS measurements (no retention-time or ion-mobility dimension)
  — the RT-dimension CentWave picking and arrival-time traces would not apply
- Goal is untargeted feature detection across all ions rather than targeted extraction
  of formulae-derived primary-ion traces
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CCSfind
  role: Tool whose processor.py orchestrates targeted m/z trace search, extraction,
    and RT peak picking from parsed mzML files
  repo: https://github.com/sangeeta97/ccs_find.git
- name: tidyms
  role: Provides the CentWave peak-picking implementation applied in the retention-time
    dimension to each extracted ion trace
- name: polars
  role: Loads the parsed mzML file into lazy frames for memory-efficient trace extraction
- name: numpy
  role: Dependency supporting numerical processing of extracted traces
- name: pandas
  role: Dependency supporting assembly and writing of the peak table
- name: conda
  role: Creates the Python 3.9 environment in which the extraction pipeline runs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/jms.5040
    title: 'CCSfind: A tool for chemically informed LC‐IM‐MS database building'
  source_content_hash: sha256:f1eec6244adde5827051ebebe6f674165b9f28476688994a31021f5281000a54
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
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# chromatographic-trace-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Targeted extraction of primary-ion m/z traces from a parsed mzML file followed by retention-time CentWave peak picking, as implemented in CCSfind for LC-IM-MS data. It converts a list of predicted ion m/z values into a confirmed peak table with RT and intensity information.

## When to use

Use when you have a profile-mode mzML file from LC-DTIM-MS and a list of molecular formulae (or predicted ion m/z values) and need to locate each monoisotopic primary-ion m/z trace over RT and arrival time, then peak-pick in the RT dimension.

## When NOT to use

- Input is already a peak/feature table rather than raw profile-mode mzML — trace extraction requires raw spectra
- Data are not LC-IM-MS measurements (no retention-time or ion-mobility dimension) — the RT-dimension CentWave picking and arrival-time traces would not apply
- Goal is untargeted feature detection across all ions rather than targeted extraction of formulae-derived primary-ion traces

## Inputs

- Profile-mode mzML file from LC-DTIM-MS (parsed into polars lazy frames; up to ~1 GB)
- List of input molecular formulae (from which monoisotopic primary-ion m/z values are derived)
- config.ini file with analysis parameters (extraction window, LC peak width, peak abundance settings)

## Outputs

- Extracted primary-ion m/z traces over RT and arrival time with intensities
- peak_table.csv: peak table of confirmed m/z traces with RT values

## How to apply

Load the parsed mzML file into polars lazy frames for memory-efficient processing in a Python 3.9 environment installed via conda and pip with requirements.txt. For each monoisotopic primary-ion m/z derived from the input molecular formulae (optionally including multiply charged ions and additional adducts in positive or negative mode), search the parsed spectra and extract the m/z trace over retention time and arrival time with intensities. Peak-pick each extracted ion trace in the RT dimension using the tidyms CentWave implementation, with user-adjustable options for expected LC peak width and expected peak abundance (noisy vs. well-defined peaks). Assemble the confirmed m/z traces with their RT values into peak_table.csv; isotopologue matching confirmation (A, A+1, A+2) can be applied downstream as a filter after peak picking. Processing is expected to handle mzML files up to 1 GB in under 2 minutes on a 32 GB RAM desktop, with runtime scaling linearly with file size or number of formulae.

## Related tools

- **CCSfind** (Tool whose processor.py orchestrates targeted m/z trace search, extraction, and RT peak picking from parsed mzML files) — https://github.com/sangeeta97/ccs_find.git
- **tidyms** (Provides the CentWave peak-picking implementation applied in the retention-time dimension to each extracted ion trace)
- **polars** (Loads the parsed mzML file into lazy frames for memory-efficient trace extraction)
- **numpy** (Dependency supporting numerical processing of extracted traces)
- **pandas** (Dependency supporting assembly and writing of the peak table)
- **conda** (Creates the Python 3.9 environment in which the extraction pipeline runs)

## Examples

```
python processor.py -g config.ini
```

## Evaluation signals

- Running `python processor.py -g config.ini` completes without error
- peak_table.csv exists after the run and is readable as a table
- Each row in peak_table.csv corresponds to a confirmed m/z trace with an associated RT value
- Processing time is consistent with mzML files up to 1 GB handled in under 2 minutes on a 32 GB RAM desktop, scaling linearly with file size or number of formulae
- Re-running on an already parsed mzML file with different settings or additional formulae completes faster than the initial parse

## Limitations

- Requires a profile-data mzML input file from LC-DTIM-MS; it is a targeted, formula-driven search, not an untargeted peak detection
- Extraction and picking quality depends on user-tunable settings: mass extraction window for isotopologue confirmation (A, A+1, A+2), expected LC peak width, and expected peak abundance (noisy vs. well-defined peaks)
- Computation scales linearly with the size of the input mzML file or the number of input molecular formulae, so very large file/formula combinations increase runtime

## Evidence

- [results] CCSfind searches each monoisotopic primary-ion mass, extracts its m/z trace over RT and arrival time with intensities: [evidence span withheld — anchor status: not_found]
- [results] each primary-ion m/z trace is then peak-picked in the RT dimension using the tidyms package: [evidence span withheld — anchor status: not_found]
- [readme] The tool uses a mzML input files to provide a unified set of results within a single data processing step: "The tool uses a mzML input files to provide a unified set of results within a single data processing step which includes filtering using isotopologue matching confirmation after peak picking."
- [readme] Option to adjust the expected LC peak width.: "Option to adjust the expected LC peak width.
* Option to adjust the expected abundance of the peak (noisy or well-defined peak)."
- [abstract] Files of up to 1 GB can be processed in less than 2 min on a desktop computer with 32 GB RAM: "Files of up to 1 GB can be processed in less than 2 min on a desktop computer with 32 GB RAM"
