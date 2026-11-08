---
name: metabolite-concentration-sampling
description: Use when simulating metabolomic 1H NMR spectra for algorithm development
  or method validation, where the simulated concentrations must reflect a real biological
  variable (e.g. case-control status, age, BMI) and inter-metabolite correlations.
when_to_use_negative:
- When the input is already a real metabolomic dataset and no synthetic spectra are
  needed for algorithm development.
- When the goal is to simulate spectra for metabolites not covered by the HMDB 5.0
  entries bundled with MetAssimulo 2.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetAssimulo 2.0
  role: Web app that performs the metabolite concentration sampling and subsequent
    1D/2D 1H NMR spectral simulation
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: Python
  role: Implementation language of the MetAssimulo 2.0 web application used to run
    the concentration sampling
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: HMDB 5.0
  role: Source of metabolite concentration ranges (mean and SD) used as sampling inputs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf045
    title: 'MetAssimulo 2.0: a web app for simulating realistic 1D and 2D metabolomic
      1H NMR spectra'
  source_content_hash: sha256:73eac020c175e6af0f36c69a6adf22ba0c4f4df0a646b359ac3fe5e1a75bbdcf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yanyan5420/MetAssimulo_2.git
  tools:
  - HMDB 5.0
  - MetAssimulo 2
  - MetAssimulo 2.0
  - Python
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# metabolite-concentration-sampling

## Summary

Simulates realistic metabolite concentrations correlated with a biological variable (discrete or continuous) using MetAssimulo 2.0, drawing per-metabolite mean and SD values from HMDB 5.0 entries via truncated-normal sampling and linear regression.

## When to use

Use when simulating metabolomic 1H NMR spectra for algorithm development or method validation, where the simulated concentrations must reflect a real biological variable (e.g. case-control status, age, BMI) and inter-metabolite correlations.

## When NOT to use

- When the input is already a real metabolomic dataset and no synthetic spectra are needed for algorithm development.
- When the goal is to simulate spectra for metabolites not covered by the HMDB 5.0 entries bundled with MetAssimulo 2.

## Inputs

- Metabolite concentration ranges (mean and SD) from HMDB 5.0 entries in the MetAssimulo 2 package Input data
- User-supplied per-metabolite mean, SD, error SD, replicate number n, correlation coefficient a, and response distribution (for the continuous module)
- Case-control group labels or a continuous outcome variable (e.g. age, body mass index)

## Outputs

- Simulated metabolite concentration values for discrete and continuous outcome modules (concentration_results.csv)
- Concentration profiles that feed into downstream 1D/2D 1H NMR spectral simulation in MetAssimulo 2.0

## How to apply

Load the metabolite concentration ranges (mean and SD) from HMDB 5.0 entries in the package Input data. For discrete outcomes (e.g. heart transplant versus healthy), convert each metabolite's concentration range to an SD via the 3-sigma rule and draw mean and SD concentrations by truncated-normal sampling around the HMDB values. For continuous outcomes (e.g. body mass index), standardize user-supplied per-metabolite mean, SD, error SD, replicate count n, and the correlation coefficient a, then sample n values to compute the sample mean and sample SD used in the linear regression of equation (2). Write both discrete and continuous simulated concentrations to concentration_results.csv, then feed them into the spectral simulation stage of MetAssimulo 2.0.

## Related tools

- **MetAssimulo 2.0** (Web app that performs the metabolite concentration sampling and subsequent 1D/2D 1H NMR spectral simulation) — https://github.com/yanyan5420/MetAssimulo_2.git
- **Python** (Implementation language of the MetAssimulo 2.0 web application used to run the concentration sampling) — https://github.com/yanyan5420/MetAssimulo_2.git
- **HMDB 5.0** (Source of metabolite concentration ranges (mean and SD) used as sampling inputs)

## Examples

```
python3 apps/index.py -p Input/parameters.txt
```

## Evaluation signals

- The script runs and the output concentration_results.csv contains concentration rows for the metabolites requested, matching the parameter file's discrete and continuous settings.
- For the continuous module, verify that the sampled n values yield the intended linear regression of equation (2) with the user-supplied correlation coefficient a.
- For the discrete module, confirm the truncated-normal draw stays within the 3-sigma-derived SD bounds from the HMDB concentration range.
- Simulated concentrations correlate with the intended outcome (case-control or continuous) at the level specified by the correlation coefficient a.

## Limitations

- The concentration ranges are bounded by the HMDB 5.0 entries bundled with MetAssimulo 2; metabolites outside this set cannot be sampled.
- For the continuous module, the correlation structure is limited to the linear regression in equation (2) driven by the user-supplied correlation coefficient a.

## Evidence

- [methods] In the continuous-outcome module, the user supplies per-metabolite mean and SD, correlation coefficient a, error SD, replicate number n, and the response distribution: [evidence span withheld — anchor status: not_found]
- [methods] simulated concentrations are computed based on equation (2): [evidence span withheld — anchor status: not_found]
- [methods] convert each metabolite's concentration range to an SD via the 3-sigma rule: [evidence span withheld — anchor status: not_found]
- [abstract] simulates both single, and groups, of spectra with both discrete (case-control, e.g. heart transplant versus healthy) and continuous (e.g. body mass index) outcomes: "MetAssimulo 2.0 simulates both single, and groups, of spectra with both discrete (case-control, e.g. heart transplant versus healthy) and continuous (e.g. body mass index) outcomes and includes"
- [abstract] The relevant NMR spectra for metabolites are deposited in MetaboLights with accession number MTBLS12081: "The relevant NMR spectra for metabolites are deposited in MetaboLights with accession number MTBLS12081."
- [readme] python3 apps/index.py -p Input/parameters.txt: "export PYTHONPATH=$HOME/MetAssimulo_2/:$PYTHONPATH
python3 apps/index.py -p Input/parameters.txt"
