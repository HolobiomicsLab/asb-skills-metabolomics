---
name: retention-time-peak-picking
description: Use when a targeted, formula-driven workflow needs to extract monoisotopic
  primary-ion m/z traces (optionally multiply charged) from LC-IM-MS mzML data and
  pick chromatographic peaks in the RT dimension, rather than performing untargeted
  feature detection.
when_to_use_negative:
- Input is already a processed feature table or peak list rather than profile-data
  mzML from LC-DTIM-MS
- The analysis is untargeted/global feature detection rather than targeted searching
  of known molecular formulae and their predicted ion species
- Data are not drift tube IM-MS (LC-DTIM-MS) measurements, since CCSfind targets drift
  tube IM-MS data with CCS calibration inputs
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CCSfind
  role: Tool that searches monoisotopic primary-ion m/z traces from input molecular
    formulae in the parsed mzML and orchestrates RT-dimension CentWave peak picking
  repo: https://github.com/sangeeta97/ccs_find.git
- name: tidyms
  role: Provides the CentWave peak-picking implementation applied to each extracted
    primary-ion m/z trace in the RT dimension
- name: polars
  role: Loads the parsed mzML file into lazy frames for memory-efficient trace extraction
- name: Python 3.9
  role: Runtime environment (created via conda) in which processor.py executes the
    peak-picking workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/jms.5040
    title: 'CCSfind: A tool for chemically informed LC‐IM‐MS database building'
  source_content_hash: sha256:7cfd15f546227dc8b43b4ae516a70122a155058f66f6c8d25bb73c3938533888
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
  - Python 3.9
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
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# retention-time-peak-picking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Targeted LC peak picking in the retention-time dimension for molecular-formula-derived primary-ion m/z traces extracted from parsed mzML files, as implemented in CCSfind using the tidyms CentWave implementation, producing a peak table of confirmed traces with RT values.

## When to use

Use when a targeted, formula-driven workflow needs to extract monoisotopic primary-ion m/z traces (optionally multiply charged) from LC-IM-MS mzML data and pick chromatographic peaks in the RT dimension, rather than performing untargeted feature detection.

## When NOT to use

- Input is already a processed feature table or peak list rather than profile-data mzML from LC-DTIM-MS
- The analysis is untargeted/global feature detection rather than targeted searching of known molecular formulae and their predicted ion species
- Data are not drift tube IM-MS (LC-DTIM-MS) measurements, since CCSfind targets drift tube IM-MS data with CCS calibration inputs

## Inputs

- Profile-data mzML input file from LC-DTIM-MS (drift tube IM-MS)
- List of molecular formulae to search in the mzML file
- config.ini parameter file (mass extraction window, expected LC peak width, expected peak abundance)

## Outputs

- peak_table.csv containing confirmed m/z traces with their retention-time values
- Extracted ion traces over RT and arrival time with intensities (intermediate)

## How to apply

Load the parsed mzML file into polars lazy frames so spectra can be searched memory-efficiently; mzML files up to 1 GB are processed in under 2 min on a 32 GB RAM desktop. For each input molecular formula, derive the monoisotopic primary-ion m/z (enabling the option to search for multiply charged ions) and extract the m/z trace over retention time and arrival time with intensities. Peak-pick each ion trace in the RT dimension with the tidyms CentWave implementation, adjusting the expected LC peak width and expected peak abundance (noisy vs. well-defined peak) in config.ini as decision parameters. Optionally adjust the mass extraction window for isotopologue confirmation (A, A+1, A+2) so that filtering using isotopologue matching confirmation follows peak picking within the same processing step. Assemble confirmed m/z traces with their RT values into peak_table.csv.

## Related tools

- **CCSfind** (Tool that searches monoisotopic primary-ion m/z traces from input molecular formulae in the parsed mzML and orchestrates RT-dimension CentWave peak picking) — https://github.com/sangeeta97/ccs_find.git
- **tidyms** (Provides the CentWave peak-picking implementation applied to each extracted primary-ion m/z trace in the RT dimension)
- **polars** (Loads the parsed mzML file into lazy frames for memory-efficient trace extraction)
- **Python 3.9** (Runtime environment (created via conda) in which processor.py executes the peak-picking workflow)

## Examples

```
python processor.py -g config.ini
```

## Evaluation signals

- Run completes via processor.py with the -g config.ini flag without errors
- peak_table.csv exists in the output directory and is readable as a table
- Peak table rows correspond to confirmed m/z traces with populated RT values for the searched formulae
- Performance consistent with the reported envelope: mzML files up to 1 GB processed in under 2 min on a 32 GB RAM desktop
- Isotopologue confirmation (A, A+1, A+2) filtering applied after peak picking yields a consistent set of confirmed traces when re-run on an already-parsed mzML with different settings

## Limitations

- Requires profile data mzML input from LC-DTIM-MS; five compulsory inputs are needed (mzML, formula list, CCS calibration coefficients Tfix and Beta, drift gas identity, primary ion species)
- Python 3.9 is a must for the conda/pip environment
- Expected LC peak width and peak abundance (noisy vs. well-defined) must be set appropriately; misconfiguration degrades peak picking

## Evidence

- [other] CCSfind searches each monoisotopic primary-ion mass, extracts its m/z trace over RT and arrival time with intensities, and each primary-ion m/z trace is then peak-picked in the RT dimension using the tidyms package.: [evidence span withheld — anchor status: not_found]
