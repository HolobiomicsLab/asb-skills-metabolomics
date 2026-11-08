---
name: targeted-ion-mz-search
description: Use when you have profile-mode LC-DTIM-MS mzML data and a list of molecular
  formulae, and need targeted extraction of primary-ion m/z traces with RT-dimension
  peak picking rather than untargeted feature detection.
when_to_use_negative:
- Input is already a feature table or peak list — this skill performs targeted extraction
  from raw profile-mode mzML, not re-processing of picked features.
- You need untargeted/de novo feature detection across the whole mzML file — CCSfind
  only targets primary-ion m/z values derived from supplied molecular formulae.
- Data is not LC-IM-MS (drift tube IM-MS) profile mzML — the trace extraction spans
  both RT and arrival time dimensions and assumes drift tube IM-MS measurements.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CCSfind
  role: Tool implementing the targeted primary-ion m/z search, trace extraction, and
    RT-dimension CentWave peak picking in a single data processing step
  repo: https://github.com/sangeeta97/ccs_find.git
- name: polars
  role: Holds the parsed mzML spectra as lazy frames for memory-efficient targeted
    m/z search and trace extraction
- name: tidyms
  role: Provides the CentWave implementation used to peak-pick each extracted primary-ion
    m/z trace in the retention-time dimension
- name: Python 3.9
  role: Runtime environment (conda/pip with requirements.txt) in which the processor.py
    analysis runs
  repo: https://github.com/sangeeta97/ccs_find.git
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/jms.5040
    title: 'CCSfind: A tool for chemically informed LC‐IM‐MS database building'
  source_content_hash: sha256:d55e7db3cf44b69ecfe52ea947f268052a24604cfdb43b410070261aa14b6650
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
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# targeted-ion-mz-search

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A chemically informed, targeted extraction step in CCSfind that searches each monoisotopic primary-ion m/z (derived from input molecular formulae) against a parsed LC-IM-MS mzML file, extracts its m/z trace over RT and arrival time, and peak-picks the trace in the retention-time dimension.

## When to use

Use when you have profile-mode LC-DTIM-MS mzML data and a list of molecular formulae, and need targeted extraction of primary-ion m/z traces with RT-dimension peak picking rather than untargeted feature detection.

## When NOT to use

- Input is already a feature table or peak list — this skill performs targeted extraction from raw profile-mode mzML, not re-processing of picked features.
- You need untargeted/de novo feature detection across the whole mzML file — CCSfind only targets primary-ion m/z values derived from supplied molecular formulae.
- Data is not LC-IM-MS (drift tube IM-MS) profile mzML — the trace extraction spans both RT and arrival time dimensions and assumes drift tube IM-MS measurements.

## Inputs

- Profile data mzML input file from LC-DTIM-MS
- List of molecular formulae to be searched for in the mzML file
- Primary ion species to be searched for (with option for multiply charged ions and additional adducts in positive and negative modes)
- config.ini file with extraction and peak-picking parameters (mass extraction window, expected LC peak width, peak abundance)

## Outputs

- Extracted primary-ion m/z traces over RT and arrival time with intensities
- peak_table.csv — peak table of confirmed m/z traces with their RT values

## How to apply

Load the parsed mzML file into polars lazy frames for memory-efficient processing, then search each monoisotopic primary-ion m/z derived from the input molecular formulae against the parsed spectra, optionally enabling the search for multiply charged ions. For each hit, extract the m/z trace over retention time and arrival time with intensities, keeping an adjustable mass extraction window that also supports isotopologue ion confirmation (A, A+1, A+2). Peak-pick each primary-ion m/z trace in the RT dimension using the tidyms CentWave implementation, tuning expected LC peak width and expected peak abundance (noisy vs. well-defined) as parameters. Assemble confirmed m/z traces with their RT values into peak_table.csv; isotopologue matching confirmation is applied downstream as a filter after peak picking.

## Related tools

- **CCSfind** (Tool implementing the targeted primary-ion m/z search, trace extraction, and RT-dimension CentWave peak picking in a single data processing step) — https://github.com/sangeeta97/ccs_find.git
- **polars** (Holds the parsed mzML spectra as lazy frames for memory-efficient targeted m/z search and trace extraction)
- **tidyms** (Provides the CentWave implementation used to peak-pick each extracted primary-ion m/z trace in the retention-time dimension)
- **Python 3.9** (Runtime environment (conda/pip with requirements.txt) in which the processor.py analysis runs) — https://github.com/sangeeta97/ccs_find.git

## Examples

```
python processor.py -g config.ini
```

## Evaluation signals

- processor.py run with the -g config.ini flag completes without error and produces peak_table.csv in the working directory
- peak_table.csv exists and is readable as a table (e.g., loadable via pandas) with confirmed m/z traces and their RT values
- Peak count scales with the number of input molecular formulae — computational time scales linearly with mzML file size or number of input formulae
- Performance envelope respected: mzML files up to 1 GB processed in under 2 min on a desktop computer with 32 GB RAM
- Isotopologue ions (A, A+1, A+2) are confirmable within the configured mass extraction window for picked primary-ion peaks

## Limitations

- Requires a profile data mzML input file from LC-DTIM-MS (drift tube IM-MS); it is not designed for non-IM or centroided-only data
- Is targeted, not untargeted: only ions matching formulae-derived primary-ion m/z values (plus optional adducts/multiply charged ions) are found
- Single-field CCS calibration coefficients (Tfix and Beta) and drift gas identity must be supplied separately for downstream CCS steps
- Python 3.9 is required ('python 3.9 is a must'); other versions are not supported

## Evidence

- [other] Search each monoisotopic primary-ion m/z derived from the input molecular formulae against the parsed spectra, enabling the option to search for multiply charged ions.: [evidence span withheld — anchor status: not_found]
