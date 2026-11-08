---
name: hmdb-concentration-data-parsing
description: Use when preparing MetAssimulo 2.0 simulations and you need human metabolite
  concentration ranges (mean and SD) drawn from HMDB 5.0 to populate the Input/parameters.txt
  file.
when_to_use_negative:
- You already have user-supplied per-metabolite mean and SD values and do not need
  HMDB-derived ranges.
- Your target biological matrix is not covered by the HMDB human metabolite entries
  bundled with MetAssimulo 2.0 (urine, blood, CSF).
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0218
tools:
- name: MetAssimulo 2.0
  role: Consumes the HMDB-derived concentration parameters to run the discrete and
    continuous outcome sampling modules
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: Python
  role: Hosts the MetAssimulo 2.0 web application that executes the concentration
    sampling after parameters are parsed
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: gdown
  role: Downloads the Input.zip archive containing the HMDB 5.0 metabolite concentration
    data
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf045
    title: 'MetAssimulo 2.0: a web app for simulating realistic 1D and 2D metabolomic
      1H NMR spectra'
  source_content_hash: sha256:9fbcb7a68b71f4898f47e17307b2c4da1cbd614a1c7061e44f82445cd02a0b31
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yanyan5420/MetAssimulo_2.git
  tools:
  - MetAssimulo 2
  - MetAssimulo 2.0
  - Python
  - gdown
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# hmdb-concentration-data-parsing

## Summary

Extracts per-metabolite mean and SD concentration values from HMDB 5.0 entries bundled in the MetAssimulo 2.0 Input data, converting them into the parameter file consumed by the discrete and continuous concentration-sampling modules.

## When to use

Use when preparing MetAssimulo 2.0 simulations and you need human metabolite concentration ranges (mean and SD) drawn from HMDB 5.0 to populate the Input/parameters.txt file.

## When NOT to use

- You already have user-supplied per-metabolite mean and SD values and do not need HMDB-derived ranges.
- Your target biological matrix is not covered by the HMDB human metabolite entries bundled with MetAssimulo 2.0 (urine, blood, CSF).

## Inputs

- HMDB 5.0 metabolite entries included in the MetAssimulo 2.0 Input data package (Input.zip)
- MetAssimulo 2.0 parameter file (Input/parameters.txt)

## Outputs

- Per-metabolite mean and SD concentration values formatted for Input/parameters.txt
- Simulated metabolite concentrations written to concentration_results.csv

## How to apply

Locate the HMDB 5.0 metabolite entries inside the MetAssimulo 2.0 package Input data (downloaded via gdown and unzipped as Input.zip). For each metabolite of interest, read its mean and SD concentration values from the HMDB record and write them into the parameter file at Input/parameters.txt. For the discrete-outcome module, convert each metabolite's concentration range to an SD using the 3-sigma rule so that truncated-normal sampling around the HMDB mean and SD can produce simulated concentrations. For the continuous-outcome module, supply the per-metabolite mean and SD alongside the correlation coefficient a, error SD, replicate number n, and response distribution so that equation (2) can be applied. The parsing step must produce values consistent with the module's sampling logic, which then writes simulated concentrations to concentration_results.csv.

## Related tools

- **MetAssimulo 2.0** (Consumes the HMDB-derived concentration parameters to run the discrete and continuous outcome sampling modules) — https://github.com/yanyan5420/MetAssimulo_2.git
- **Python** (Hosts the MetAssimulo 2.0 web application that executes the concentration sampling after parameters are parsed) — https://github.com/yanyan5420/MetAssimulo_2.git
- **gdown** (Downloads the Input.zip archive containing the HMDB 5.0 metabolite concentration data)

## Examples

```
python3 apps/index.py -p Input/parameters.txt
```

## Evaluation signals

- The Input/parameters.txt file contains a mean and SD value for each requested metabolite sourced from its HMDB 5.0 entry.
- Discrete-module parameters include an SD derived from the 3-sigma rule applied to the HMDB concentration range.
- Running the discrete and continuous modules produces a concentration_results.csv file containing rows for the metabolites specified in the parameter file.
- Simulated concentrations for a given metabolite fall within a plausible range around the HMDB mean and SD values used as input.

## Limitations

- HMDB values are human metabolite reference ranges; matrices not represented in the Input data (e.g. non-human samples) are not supported by this parsing route.
- The discrete module requires the 3-sigma rule conversion, which assumes the HMDB concentration range can be approximated as ±3 SD around the mean.
- The MetAssimulo 2.0 environment must be activated (py37_metassimulo_2_env) and PYTHONPATH set before the parameter file can be consumed.

## Evidence

- [full_text] Discrete module HMDB parameter source: [evidence span withheld — anchor status: not_found]
- [full_text] 3-sigma rule for SD conversion: [evidence span withheld — anchor status: not_found]
- [full_text] Continuous module parameters: [evidence span withheld — anchor status: not_found]
- [full_text] Output file name: [evidence span withheld — anchor status: not_found]
- [readme] Repository run command: "python3 apps/index.py -p Input/parameters.txt"
