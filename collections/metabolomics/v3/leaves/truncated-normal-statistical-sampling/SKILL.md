---
name: truncated-normal-statistical-sampling
description: Use when you need to generate realistic metabolite concentration vectors
  that follow a truncated-normal distribution around known per-metabolite means and
  SDs, and whose variation correlates with a continuous biological variable such as
  age or BMI.
when_to_use_negative:
- When you already have real, measured metabolite concentration data and only need
  to feed it directly into the spectral simulator rather than generate synthetic concentrations
- When the biological outcome is discrete (e.g. case-control such as heart transplant
  versus healthy) rather than a continuous variable like age or BMI — the discrete
  module uses a different (non-regression) concentration-sampling approach
- When you need simulated NMR spectra themselves rather than the underlying concentration
  vectors that drive them
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0204
tools:
- name: MetAssimulo 2.0
  role: Python web application that performs the truncated-normal sampling and equation
    (2) linear regression to simulate concentrations correlated with continuous outcomes
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: Python
  role: Implementation language of the MetAssimulo 2.0 web app that carries out the
    concentration-sampling modules
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf045
    title: 'MetAssimulo 2.0: a web app for simulating realistic 1D and 2D metabolomic
      1H NMR spectra'
  source_content_hash: sha256:50267a0422cc76fe55786ff8be5061b83b648553d399c7b4648c66cad3658644
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yanyan5420/MetAssimulo_2.git
  tools:
  - MetAssimulo 2
  - MetAssimulo 2.0
  - Python
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# truncated-normal-statistical-sampling

## Summary

This skill simulates metabolite concentrations by truncated-normal sampling around a supplied mean and SD (or HMDB-derived ranges converted via the 3-sigma rule), then correlates the sampled concentrations with a continuous outcome using MetAssimulo 2.0's linear regression equation (2). It produces per-metabolite concentration vectors used to drive simulated 1D/2D 1H NMR spectra.

## When to use

Use when you need to generate realistic metabolite concentration vectors that follow a truncated-normal distribution around known per-metabolite means and SDs, and whose variation correlates with a continuous biological variable such as age or BMI.

## When NOT to use

- When you already have real, measured metabolite concentration data and only need to feed it directly into the spectral simulator rather than generate synthetic concentrations
- When the biological outcome is discrete (e.g. case-control such as heart transplant versus healthy) rather than a continuous variable like age or BMI — the discrete module uses a different (non-regression) concentration-sampling approach
- When you need simulated NMR spectra themselves rather than the underlying concentration vectors that drive them

## Inputs

- Per-metabolite concentration ranges (mean and SD) from HMDB 5.0 entries in the MetAssimulo_2 'Input' data folder
- User-supplied continuous-outcome parameters: per-metabolite mean, SD, replicate number n, error SD, correlation coefficient a, and the response distribution
- Parameters file (Input/parameters.txt) specifying discrete and continuous simulation settings

## Outputs

- Simulated metabolite concentration vectors correlated with a continuous outcome such as age or BMI
- concentration_results.csv containing concentration rows for the requested metabolites under both discrete and continuous settings

## How to apply

First, load per-metabolite concentration ranges (mean and SD) from the HMDB 5.0 entries included in the MetAssimulo_2 repository's Input data folder (downloaded via gdown). Convert any concentration range to an SD using the 3-sigma rule, then draw each metabolite's mean and SD concentrations by truncated-normal sampling around the HMDB-derived values so the simulated variation is physiologically bounded. For the continuous-outcome module, supply the standardized per-metabolite means, SDs, replicate number n, error SD, correlation coefficient a, and the response distribution; MetAssimulo 2.0 then samples n values to obtain the sample mean and sample SD for each metabolite and the outcome, and applies the linear regression of equation (2) to produce simulated concentrations that correlate with the target variable. Rationale: truncated-normal sampling keeps concentrations within the observed physiological range while respecting inter-metabolite correlations, yielding realistic input concentrations for downstream 1H NMR spectral simulation.

## Related tools

- **MetAssimulo 2.0** (Python web application that performs the truncated-normal sampling and equation (2) linear regression to simulate concentrations correlated with continuous outcomes) — https://github.com/yanyan5420/MetAssimulo_2.git
- **Python** (Implementation language of the MetAssimulo 2.0 web app that carries out the concentration-sampling modules) — https://github.com/yanyan5420/MetAssimulo_2.git

## Examples

```
conda activate py37_metassimulo_2_env; export PYTHONPATH=$HOME/MetAssimulo_2/:$PYTHONPATH; python3 apps/index.py -p Input/parameters.txt
```

## Evaluation signals

- The script runs end-to-end without errors after activating py37_metassimulo_2_env and exporting PYTHONPATH to the MetAssimulo_2 root
- concentration_results.csv contains concentration rows for every metabolite requested in Input/parameters.txt
- Simulated concentrations remain within the physiological bounds implied by the 3-sigma-converted HMDB ranges (no negative or out-of-range concentrations)
- Simulated concentrations show a linear relationship with the continuous outcome consistent with the user-supplied correlation coefficient a (Pearson correlation should approximate a for sufficiently large n)
- Both discrete and continuous simulation modes appear in the output file, matching the parameters file's discrete and continuous settings

## Limitations

- HMDB 5.0 concentration ranges must be converted to SDs via the 3-sigma rule, so SD estimates are approximate for metabolites whose ranges are not truly 3-sigma wide
- The continuous-outcome simulation assumes a linear relationship between concentrations and the outcome (equation 2); non-linear outcome dependencies are not modelled
- Only the 1H NMR simulation scope (urine, blood, and CSF) is supported; concentration sampling for other biofluids is out of scope

## Evidence

- [other] equation (2) linear regression: [evidence span withheld — anchor status: not_found]
- [other] 3-sigma rule conversion: [evidence span withheld — anchor status: not_found]
- [abstract] MetAssimulo 2.0 scope: "MetAssimulo 2.0 simulates both single, and groups, of spectra with both discrete (case-control, e.g. heart transplant versus healthy) and continuous (e.g. body mass index) outcomes and includes"
- [abstract] MetAssimulo 2.0 repo availability: "The code and the detailed instruction/tutorial for MetAssimulo 2.0 is available at https://github.com/yanyan5420/MetAssimulo_2.git"
- [readme] README invocation: "python3 apps/index.py -p Input/parameters.txt"
