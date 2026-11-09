---
name: concentration-range-sigma-estimation
description: Use when generating simulated metabolite concentrations from HMDB 5.0
  concentration ranges (mean and SD) and a discrete outcome such as case-control status,
  where each metabolite's range must be converted into a standard deviation before
  sampling.
when_to_use_negative:
- The analysis goal is a continuous outcome (e.g. body mass index or age) — use the
  continuous module's equation (2) regression with user-supplied means, SDs, replicate
  number and error SD instead of the 3-sigma range conversion.
- Only simulated spectra are needed without realistic concentration-level variation
  — MetAssimulo 2.0's spectral simulation can be run without the concentration-sampling
  step.
- Input concentration data are already per-sample measurements with their own empirical
  SDs, rather than HMDB 5.0 range entries that must be converted.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetAssimulo 2.0
  role: Hosts the discrete-outcome concentration-sampling module that consumes the
    sigma-estimated means and sigmas and writes concentration_results.csv
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: Python
  role: Implementation language of MetAssimulo 2.0's concentration-sampling and truncated-normal
    sampling code
- name: MetAssimulo 2
  role: Command-line entry point (apps/index.py) run with the parameter file to execute
    the concentration simulation
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf045
    title: 'MetAssimulo 2.0: a web app for simulating realistic 1D and 2D metabolomic
      1H NMR spectra'
  source_content_hash: sha256:3ff6dcc9186d185437fe649a815d9ec12f40c0276b2beb0357551b2799e32d66
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yanyan5420/MetAssimulo_2.git
  tools:
  - MetAssimulo 2
  - MetAssimulo 2.0
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# concentration-range-sigma-estimation

## Summary

Estimate the per-metabolite standard deviation (sigma) needed for simulating metabolite concentrations in MetAssimulo 2.0 by converting HMDB 5.0-derived concentration ranges into sigma via the 3-sigma rule, then sampling means and sigmas by truncated-normal draws around the HMDB values.

## When to use

Use when generating simulated metabolite concentrations from HMDB 5.0 concentration ranges (mean and SD) and a discrete outcome such as case-control status, where each metabolite's range must be converted into a standard deviation before sampling.

## When NOT to use

- The analysis goal is a continuous outcome (e.g. body mass index or age) — use the continuous module's equation (2) regression with user-supplied means, SDs, replicate number and error SD instead of the 3-sigma range conversion.
- Only simulated spectra are needed without realistic concentration-level variation — MetAssimulo 2.0's spectral simulation can be run without the concentration-sampling step.
- Input concentration data are already per-sample measurements with their own empirical SDs, rather than HMDB 5.0 range entries that must be converted.

## Inputs

- Metabolite concentration ranges (mean and SD) from HMDB 5.0 entries supplied in the MetAssimulo 2.0 Input data package
- Parameter file (Input/parameters.txt) specifying the discrete-outcome simulation settings
- Discrete outcome definition (e.g. case-control, heart transplant versus healthy)

## Outputs

- Per-metabolite sampled mean concentration and sigma concentration values used by the discrete-outcome module
- Simulated metabolite concentrations written to concentration_results.csv (discrete and continuous results combined)

## How to apply

Load the metabolite concentration ranges (mean and SD) from the HMDB 5.0 entries bundled in MetAssimulo 2.0's Input data package. For each metabolite, convert its concentration range to a standard deviation using the 3-sigma rule, treating the range as ±3 sigma so that simulated concentrations stay within the reported range. Then draw a mean concentration and a sigma concentration for each metabolite by truncated-normal sampling around the HMDB-supplied values, which prevents implausible negatives and keeps means within the expected bounds. These per-metabolite mean and sigma pairs feed directly into MetAssimulo 2.0's discrete-outcome concentration-sampling module, whose output is written together with the continuous-module results to concentration_results.csv. Correctness is judged by confirming the script runs and that the output file contains concentration rows for the requested metabolites matching the parameter file's discrete settings.

## Related tools

- **MetAssimulo 2.0** (Hosts the discrete-outcome concentration-sampling module that consumes the sigma-estimated means and sigmas and writes concentration_results.csv) — https://github.com/yanyan5420/MetAssimulo_2.git
- **Python** (Implementation language of MetAssimulo 2.0's concentration-sampling and truncated-normal sampling code)
- **MetAssimulo 2** (Command-line entry point (apps/index.py) run with the parameter file to execute the concentration simulation) — https://github.com/yanyan5420/MetAssimulo_2.git

## Examples

```
export PYTHONPATH=$HOME/MetAssimulo_2/:$PYTHONPATH; python3 apps/index.py -p Input/parameters.txt
```

## Evaluation signals

- The MetAssimulo 2.0 script runs without error after the environment is created and activated.
- concentration_results.csv is produced and contains concentration rows for the metabolites requested in the parameter file.
- The discrete rows in concentration_results.csv reflect the discrete settings given in Input/parameters.txt.
- Per-metabolite sampled sigmas are consistent with the 3-sigma interpretation of the HMDB 5.0 concentration ranges (range ≈ ±3 sigma).

## Limitations

- The 3-sigma rule is an assumption that the reported HMDB concentration range corresponds to ±3 sigma; metabolites with unusually wide or narrow ranges will have correspondingly rough sigma estimates.
- The discrete module relies on HMDB 5.0-derived ranges; concentration values outside the supplied Input data are not covered.
- Results for the discrete and continuous modules are written to a single file (concentration_results.csv), so separating outcome types requires parsing the combined output.

## Evidence

- [other] For the discrete module, convert each metabolite's concentration range to an SD via the 3-sigma rule, then draw mean and SD concentrations by truncated-normal sampling around the HMDB values.: [evidence span withheld — anchor status: not_found]
- [other] Write the discrete and continuous simulated concentrations to a single results file (concentration_results.csv).: [evidence span withheld — anchor status: not_found]
- [abstract] MetAssimulo 2.0 simulates both single, and groups, of spectra with both discrete (case-control, e.g. heart transplant versus healthy) and continuous (e.g. body mass index) outcomes and includes: "MetAssimulo 2.0 simulates both single, and groups, of spectra with both discrete (case-control, e.g. heart transplant versus healthy) and continuous (e.g. body mass index) outcomes and includes"
- [other] Load the metabolite concentration ranges (mean and SD) from the HMDB 5.0 entries in the package Input data.: [evidence span withheld — anchor status: not_found]
- [other] Validation: confirm the script runs and the output file contains concentration rows for the metabolites requested, matching the parameter file's discrete and continuous settings.: [evidence span withheld — anchor status: not_found]
- [readme] python3 apps/index.py -p Input/parameters.txt: "python3 apps/index.py -p Input/parameters.txt"
