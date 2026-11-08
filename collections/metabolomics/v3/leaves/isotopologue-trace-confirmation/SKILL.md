---
name: isotopologue-trace-confirmation
description: Use when building LC-IM-MS (drift tube IM-MS) databases from profile
  mzML files for a list of molecular formulae, and candidate primary-ion m/z traces
  must be confirmed by isotopologue matching after RT peak picking.
when_to_use_negative:
- Input is already a peak/feature table rather than profile LC-DTIM-MS mzML data —
  no trace extraction or CentWave peak picking is needed.
- The goal is untargeted feature detection without a priori molecular formulae; this
  skill is chemically informed and requires input molecular formulae to derive target
  primary-ion m/z.
- Data are not drift tube IM-MS (LC-IM-MS) measurements, since CCSfind's workflow
  (arrival-time traces, CCS calibration, drift gas identity) is specific to DTIM-MS.
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CCSfind
  role: 'Host tool: searches monoisotopic primary-ion m/z, extracts ion traces, and
    applies isotopologue matching confirmation after peak picking in a single processing
    step'
  repo: https://github.com/sangeeta97/ccs_find.git
- name: tidyms
  role: Provides the CentWave peak-picking implementation used on each extracted primary-ion
    m/z trace in the RT dimension
- name: polars
  role: Holds the parsed mzML spectra as lazy frames for memory-efficient trace extraction
- name: pandas
  role: Dependency used alongside numpy for assembling and writing the peak table
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/jms.5040
    title: 'CCSfind: A tool for chemically informed LC‐IM‐MS database building'
  source_content_hash: sha256:d48cebce570d0a65356ab91cbb405329b83feebd870f930d1f8688ae25036819
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
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# isotopologue-trace-confirmation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Chemically informed targeted peak picking in CCSfind: monoisotopic primary-ion m/z traces from a parsed mzML file are extracted over retention time, peak-picked in the RT dimension with the tidyms CentWave implementation, and filtered by isotopologue matching confirmation (A, A+1, A+2) before being written to a peak table.

## When to use

Use when building LC-IM-MS (drift tube IM-MS) databases from profile mzML files for a list of molecular formulae, and candidate primary-ion m/z traces must be confirmed by isotopologue matching after RT peak picking.

## When NOT to use

- Input is already a peak/feature table rather than profile LC-DTIM-MS mzML data — no trace extraction or CentWave peak picking is needed.
- The goal is untargeted feature detection without a priori molecular formulae; this skill is chemically informed and requires input molecular formulae to derive target primary-ion m/z.
- Data are not drift tube IM-MS (LC-IM-MS) measurements, since CCSfind's workflow (arrival-time traces, CCS calibration, drift gas identity) is specific to DTIM-MS.

## Inputs

- Profile data mzML input file from LC-DTIM-MS (up to ~1 GB)
- List of molecular formulae to be searched for in the mzML file
- config.ini with analysis parameters (extraction window, LC peak width, peak abundance, charged-ion search option)

## Outputs

- peak_table.csv — peak table of confirmed primary-ion m/z traces with their RT values
- Confirmed ion traces (A, A+1, A+2 isotopologues) feeding CCSfind's downstream CCS/database steps

## How to apply

Load the parsed mzML file into polars lazy frames for memory-efficient processing, then search each monoisotopic primary-ion m/z derived from the input molecular formulae against the parsed spectra (with an option to search for multiply charged ions). Extract each primary-ion m/z trace over RT and arrival time with intensities, and peak-pick each trace in the retention-time dimension using the tidyms CentWave implementation. Adjust the mass extraction window for isotopologue ion confirmation of the A, A+1, and A+2 ions, and optionally tune the expected LC peak width and expected peak abundance (noisy vs well-defined). Assemble the peak table of confirmed m/z traces with their RT values and write it to peak_table.csv; isotopologue matching confirmation after peak picking acts as the filtering step within CCSfind's single unified data processing workflow.

## Related tools

- **CCSfind** (Host tool: searches monoisotopic primary-ion m/z, extracts ion traces, and applies isotopologue matching confirmation after peak picking in a single processing step) — https://github.com/sangeeta97/ccs_find.git
- **tidyms** (Provides the CentWave peak-picking implementation used on each extracted primary-ion m/z trace in the RT dimension)
- **polars** (Holds the parsed mzML spectra as lazy frames for memory-efficient trace extraction)
- **pandas** (Dependency used alongside numpy for assembling and writing the peak table)

## Examples

```
python processor.py -g config.ini
```

## Evaluation signals

- Running `python processor.py -g config.ini` completes without error and produces a peak_table.csv that exists and is a readable table.
- Each retained m/z trace has confirmed A, A+1, and A+2 isotopologue ions within the configured mass extraction window.
- RT peak positions in peak_table.csv correspond to chromatographic peaks found by the tidyms CentWave implementation on the extracted ion traces.
- Processing of a ≤1 GB mzML file finishes in under 2 min on a 32 GB RAM desktop, consistent with the reported linear scaling with mzML file size or number of input formulae.
- Confirmed traces can be re-derived faster when re-running the already-parsed mzML file with different settings or additional formulae.

## Limitations

- Requires profile (not pre-centroided/feature-table) mzML input from LC-DTIM-MS, plus mandatory inputs: molecular formulae list, single-field CCS calibration coefficients (Tfix and Beta), drift gas identity, and primary ion species.
- Confirmation depends on user-adjustable parameters (mass extraction window, expected LC peak width, expected peak abundance for noisy vs well-defined peaks), so misconfiguration can reject genuine traces or pass noise.
- Performance envelope of <2 min for ≤1 GB mzML assumes a desktop computer with 32 GB RAM; larger files or very large formula lists may exceed this.

## Evidence
