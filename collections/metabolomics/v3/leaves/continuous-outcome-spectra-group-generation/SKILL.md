---
name: continuous-outcome-spectra-group-generation
description: Use when developing or benchmarking a metabolomics algorithm (e.g. PCA,
  regression) that requires many simulated spectra with a continuous covariate, and
  no real spectra with known ground-truth concentrations are available.
when_to_use_negative:
- The analysis target is real patient/sample spectra with measured concentrations
  rather than simulated data with known ground truth
- The study design is case-control (discrete outcome, e.g. heart transplant versus
  healthy) rather than a continuous covariate — use the discrete-outcome grouped mode
  instead
- Only a single spectrum is needed (e.g. to inspect one metabolite's peak pattern)
  — the single-spectrum mode is simpler than setting up a 100-replicate grouped simulation
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_3520
tools:
- name: MetAssimulo 2.0
  role: Web application that performs the grouped continuous-outcome spectra simulation,
    including blood protein background and inter-metabolite correlations
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: Python
  role: Implementation language of the MetAssimulo 2.0 web app; used to run apps/index.py
    and downstream PCA on the exported spectra
- name: conda
  role: Creates and activates the py37_metassimulo_2_env environment that supplies
    the project dependencies
- name: gdown
  role: Downloads the Input.zip bundle containing the parameters.txt and reference
    metabolite spectra
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf045
    title: 'MetAssimulo 2.0: a web app for simulating realistic 1D and 2D metabolomic
      1H NMR spectra'
  source_content_hash: sha256:8356cdafeef1d4e5381a337dc0b17f5bc8297320ef5d00c5b55f6b072843d006
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yanyan5420/MetAssimulo_2.git
  tools:
  - Anaconda
  - Git
  - MetAssimulo 2.0
  - Python
  - conda
  - gdown
  - pip
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# Continuous-outcome spectra group generation

## Summary

Generate groups of simulated 1D 1H NMR spectra whose metabolite concentrations follow a linear regression model against a continuous outcome (e.g. BMI), using MetAssimulo 2.0's grouped-spectra mode with inter-metabolite correlations enabled.

## When to use

Use when developing or benchmarking a metabolomics algorithm (e.g. PCA, regression) that requires many simulated spectra with a continuous covariate, and no real spectra with known ground-truth concentrations are available.

## When NOT to use

- The analysis target is real patient/sample spectra with measured concentrations rather than simulated data with known ground truth
- The study design is case-control (discrete outcome, e.g. heart transplant versus healthy) rather than a continuous covariate — use the discrete-outcome grouped mode instead
- Only a single spectrum is needed (e.g. to inspect one metabolite's peak pattern) — the single-spectrum mode is simpler than setting up a 100-replicate grouped simulation

## Inputs

- MetAssimulo 2.0 Input/parameters.txt configuration file (downloaded Input.zip)
- Metabolite selection (e.g. the 10 metabolites correlated with BMI) and their reference NMR spectra (MetaboLights MTBLS12081)
- Choice of biofluid: human urine, blood, or cerebral spinal fluid
- Continuous outcome specification (e.g. body mass index)

## Outputs

- Matrix of 100 simulated 1D 1H NMR spectra (samples x spectral buckets/ppm variables)
- Associated continuous outcome values (e.g. BMI) for each simulated replicate
- Optionally, grouped 2D J-Res and COSY spectra for the same replicates

## How to apply

Launch MetAssimulo 2.0 via `python3 apps/index.py -p Input/parameters.txt` inside the py37_metassimulo_2_env conda environment, then open http://127.0.0.1:8060/. In the web app, choose the grouped-spectra simulation for the target biofluid (e.g. human blood), select the metabolites of interest (e.g. the 10 BMI-correlated metabolites), and set the continuous outcome (e.g. body mass index) so that concentrations are drawn from the section 2.2.2 linear regression model. Enable inter-metabolite correlations so realistic co-variation is reproduced, and request 100 replicates. Export the simulated spectra matrix together with the outcome values; for blood, the built-in broad protein background makes spectra correlate with real blood spectra at a Pearson correlation of approximately 0.82.

## Related tools

- **MetAssimulo 2.0** (Web application that performs the grouped continuous-outcome spectra simulation, including blood protein background and inter-metabolite correlations) — https://github.com/yanyan5420/MetAssimulo_2.git
- **Python** (Implementation language of the MetAssimulo 2.0 web app; used to run apps/index.py and downstream PCA on the exported spectra)
- **conda** (Creates and activates the py37_metassimulo_2_env environment that supplies the project dependencies)
- **gdown** (Downloads the Input.zip bundle containing the parameters.txt and reference metabolite spectra)

## Examples

```
conda activate py37_metassimulo_2_env; export PYTHONPATH=$HOME/MetAssimulo_2/:$PYTHONPATH; python3 apps/index.py -p Input/parameters.txt
```

## Evaluation signals

- The Dash server reports 'Dash is running on http://127.0.0.1:8060/' after launch, confirming the app is reachable before configuring the simulation
- The exported spectra matrix has the requested number of replicates (e.g. 100 samples) and the outcome values align row-wise with the spectra
- For blood simulations, Pearson correlation between simulated and real blood spectra should be approximately 0.82
- Downstream PCA on the exported spectra reproduces the expected finding, e.g. PC2 capturing most BMI-related variance with lactate having the highest PC2 loading
- The parameters.txt file and Input/ directory are present in the project root, confirming the Input.zip download and unzip succeeded

## Limitations

- Requires the project dependencies to be installed via environment.yml into a Python 3.7 conda environment (py37_metassimulo_2_env); no changelog is provided for version drift
- The reference metabolite NMR spectra must be downloaded from the provided Google Drive link via gdown; unavailable network access to that link blocks setup
- Blood-spectrum realism depends on the built-in broad protein background and reaches only ~0.82 Pearson correlation with real blood spectra, so simulated spectra are an approximation

## Evidence

- [abstract] Grouped continuous-outcome mode is a documented MetAssimulo 2.0 capability: "MetAssimulo 2.0 simulates both single, and groups, of spectra with both discrete (case-control, e.g. heart transplant versus healthy) and continuous (e.g. body mass index) outcomes"
- [abstract] Blood realism metric grounded in the abstract: "This enhancement enables a closer approximation to real blood spectra, achieving a Pearson correlation of approximately 0.82."
- [readme] Launch command and config file path from the README: "export PYTHONPATH=$HOME/MetAssimulo_2/:$PYTHONPATH
python3 apps/index.py -p Input/parameters.txt"
- [readme] Web app URL confirming successful launch: "`Dash is running on http://127.0.0.1:8060/`"
- [abstract] Reference spectra deposition used as simulation input: "The relevant NMR spectra for metabolites are deposited in MetaboLights with accession number MTBLS12081."
- [readme] Input data download step required before running: "pip install gdown
gdown https://drive.google.com/uc?id=12k-nbIcoME5GSkD8u1iYUcNzg6l-7msW
unzip Input.zip"
- [readme] Conda environment setup required to run the app: "conda env create -f environment.yml
conda activate py37_metassimulo_2_env"
