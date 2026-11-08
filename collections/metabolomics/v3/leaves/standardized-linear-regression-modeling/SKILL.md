---
name: standardized-linear-regression-modeling
description: Use when simulating metabolomic spectra whose concentrations must correlate
  with a continuous biological variable (e.g. BMI, age) rather than a discrete case-control
  label.
when_to_use_negative:
- The outcome of interest is discrete/case-control (e.g. heart transplant versus healthy)
  — use the discrete module that converts concentration ranges to SDs via the 3-sigma
  rule and truncated-normal sampling instead.
- The target is a single uncorrelated concentration draw rather than a response-linked
  set — the single-spectrum mode of MetAssimulo 2.0 is the correct choice.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: MetAssimulo 2.0
  role: Implements the standardized linear regression (equation 2) to sample metabolite
    concentrations correlated with the continuous outcome before spectral simulation
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: Python
  role: Language in which MetAssimulo 2.0 is reimplemented and in which PCA validation
    of the simulated spectra is performed
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: conda
  role: Environment manager used to create and activate py37_metassimulo_2_env that
    runs the MetAssimulo 2.0 web app
- name: scikit-learn
  role: Used to apply PCA on the 100-spectra matrix to confirm PC2 captures BMI-related
    variance
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf045
    title: 'MetAssimulo 2.0: a web app for simulating realistic 1D and 2D metabolomic
      1H NMR spectra'
  source_content_hash: sha256:6cc08ba5d4d3a7a249a3e14b5472a96020a3f15d5129e23c3330d2a7466f0879
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yanyan5420/MetAssimulo_2.git
  tools:
  - Anaconda
  - Git
  - MetAssimulo 2
  - MetAssimulo 2.0
  - Python
  - conda
  - gdown
  - pip
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# Standardized linear regression modeling

## Summary

Generates correlated simulated metabolite concentrations linked to a continuous outcome (e.g. BMI) by standardizing user-supplied mean/SD/replicate/error parameters and applying MetAssimulo 2.0's equation (2) linear regression, so downstream spectral simulation carries realistic inter-metabolite and metabolite-outcome structure.

## When to use

Use when simulating metabolomic spectra whose concentrations must correlate with a continuous biological variable (e.g. BMI, age) rather than a discrete case-control label.

## When NOT to use

- The outcome of interest is discrete/case-control (e.g. heart transplant versus healthy) — use the discrete module that converts concentration ranges to SDs via the 3-sigma rule and truncated-normal sampling instead.
- The target is a single uncorrelated concentration draw rather than a response-linked set — the single-spectrum mode of MetAssimulo 2.0 is the correct choice.

## Inputs

- Per-metabolite mean concentration and standard deviation (e.g. from HMDB 5.0 entries in MetAssimulo 2.0's Input data)
- Number of replicate samples n
- Measurement-error standard deviation
- Correlation coefficient a linking each metabolite to the continuous outcome
- Assumed distribution of the continuous outcome (e.g. BMI)

## Outputs

- Simulated metabolite concentration matrix (n samples x metabolites) correlated with the continuous outcome via equation (2)
- Concentration vector with enabled inter-metabolite correlations
- concentration_results.csv with discrete and continuous simulated concentrations

## How to apply

For each metabolite, supply a mean, standard deviation, the number of replicate samples n, a measurement-error SD, and a correlation coefficient a that links the metabolite to the continuous outcome, plus the assumed distribution of the outcome. Standardize these inputs, then for each metabolite draw n concentration values from which the sample mean and sample SD of both the metabolite and the response are computed; plug these standardized quantities into MetAssimulo 2.0's equation (2), a linear regression relating standardized metabolite concentration to the standardized outcome, to obtain the simulated concentrations. Enable inter-metabolite correlations so the resulting concentration vector exhibits the same covariance structure observed in real data. The simulated concentrations are then fed to the spectral simulation module (for blood, urine, or CSF 1H NMR) with the broad protein background enabled to mimic real blood spectra. Validate by checking that the regression reproduces the specified correlation a and that a PCA on the resulting spectra (e.g. 100 BMI-linked blood replicates) places the BMI signal on the second principal component with lactate showing the highest PC2 loading.

## Related tools

- **MetAssimulo 2.0** (Implements the standardized linear regression (equation 2) to sample metabolite concentrations correlated with the continuous outcome before spectral simulation) — https://github.com/yanyan5420/MetAssimulo_2.git
- **Python** (Language in which MetAssimulo 2.0 is reimplemented and in which PCA validation of the simulated spectra is performed) — https://github.com/yanyan5420/MetAssimulo_2.git
- **conda** (Environment manager used to create and activate py37_metassimulo_2_env that runs the MetAssimulo 2.0 web app)
- **scikit-learn** (Used to apply PCA on the 100-spectra matrix to confirm PC2 captures BMI-related variance)

## Examples

```
python3 apps/index.py -p Input/parameters.txt
```

## Evaluation signals

- The simulated concentration matrix reproduces the user-specified correlation coefficient a between each metabolite and the continuous outcome.
- PCA on ~100 simulated blood spectra places the BMI-related variance on PC2, not PC1, matching the paper's section 2.2.2 result.
- Lactate exhibits the largest absolute PC2 loading in the BMI simulation, consistent with the reported finding.
- concentration_results.csv contains concentration rows for the requested metabolites matching the parameters.txt continuous settings.
- Simulated blood spectra (with protein background) approximate real blood spectra at a Pearson correlation of approximately 0.82.

## Limitations

- Requires the user to pre-supply per-metabolite means, SDs, replicate count n, error SD, correlation coefficient a, and the outcome distribution — the model does not estimate these from data.
- Validation of the correlation structure is indirect: the paper demonstrates it through downstream PCA of the simulated spectra rather than direct statistical tests on the concentration matrix.
- The continuous-outcome regression (equation 2) is specific to MetAssimulo 2.0's parameterization and is not portable to other spectral simulation frameworks without re-derivation.

## Evidence

- [other] the user supplies per-metabolite mean and SD, correlation coefficient a, error SD, replicate number n, and the response distribution: [evidence span withheld — anchor status: not_found]
- [other] MetAssimulo 2.0 then samples n values to calculate the sample mean and sample SD for each metabolite and the response, from which simulated concentrations are computed based on equation (2): [evidence span withheld — anchor status: not_found]
- [abstract] simulates both single, and groups, of spectra with both discrete (case-control, e.g. heart transplant versus healthy) and continuous (e.g. body mass index) outcomes and includes: "simulates both single, and groups, of spectra with both discrete (case-control, e.g. heart transplant versus healthy) and continuous (e.g. body mass index) outcomes and includes"
- [other] PCA on the 100 simulated blood spectra shows that the second principal component (PC2) captures most of the BMI-related variance: [evidence span withheld — anchor status: not_found]
- [readme] conda env create -f environment.yml: "conda env create -f environment.yml
conda activate py37_metassimulo_2_env"
