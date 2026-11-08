---
name: nmr-reference-spectrum-retrieval
description: Use when you need reference-like 1D 1H NMR spectra for blood/urine/CSF
  and no suitable experimental spectra are available, or when benchmarking spectral
  algorithms against simulated-vs-real comparisons.
when_to_use_negative:
- You need experimentally measured NMR spectra for biomarker discovery rather than
  simulated reference spectra for algorithm development
- The biofluid of interest is outside MetAssimulo 2.0's scope (it simulates urine,
  blood, and cerebral spinal fluid only)
- You need raw NMR data processing rather than spectral simulation — this skill generates
  synthetic spectra, it does not process FIDs
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: MetAssimulo 2.0
  role: Web application that simulates the 1D 1H NMR blood spectrum with the broad
    protein background used as the reference-like spectrum
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: conda
  role: Creates and activates the pinned Python environment (py37_metassimulo_2_env)
    required to run the simulator
- name: gdown
  role: Downloads the Input.zip simulation parameters data from Google Drive before
    running the simulation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf045
    title: 'MetAssimulo 2.0: a web app for simulating realistic 1D and 2D metabolomic
      1H NMR spectra'
  source_content_hash: sha256:9e18439358b1fde2e3db2cfd798c62f2a6b58e80d1567c6f2d396c2fdd5a2d60
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yanyan5420/MetAssimulo_2.git
  tools:
  - Anaconda
  - Git
  - MetAssimulo 2.0
  - Miniconda
  - conda
  - gdown
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# nmr-reference-spectrum-retrieval

## Summary

Generate realistic simulated 1D 1H NMR spectra of biofluids (urine, blood, CSF) with MetAssimulo 2.0, using the broad protein (albumin) background for blood to approximate real blood spectra, and validate the approximation by Pearson correlation (~0.82).

## When to use

Use when you need reference-like 1D 1H NMR spectra for blood/urine/CSF and no suitable experimental spectra are available, or when benchmarking spectral algorithms against simulated-vs-real comparisons.

## When NOT to use

- You need experimentally measured NMR spectra for biomarker discovery rather than simulated reference spectra for algorithm development
- The biofluid of interest is outside MetAssimulo 2.0's scope (it simulates urine, blood, and cerebral spinal fluid only)
- You need raw NMR data processing rather than spectral simulation — this skill generates synthetic spectra, it does not process FIDs

## Inputs

- Input/parameters.txt simulation configuration (blood plasma 1D 1H NMR parameters) from the MetAssimulo_2 Input data download
- Real blood 1H NMR spectrum (spectral intensity vector) for the comparison
- GitHub repository yanyan5420/MetAssimulo_2 with environment.yml

## Outputs

- Simulated 1D blood plasma 1H NMR spectrum with broad protein (albumin) background
- Pearson correlation coefficient between simulated and real spectra, written to blood_spectrum_correlation.csv

## How to apply

Clone https://github.com/yanyan5420/MetAssimulo_2.git and download the simulation input data (Input.zip via gdown from the Google Drive link in the README), which provides the Input/parameters.txt used to configure the simulation. Create the conda environment with `conda env create -f environment.yml` and activate py37_metassimulo_2_env. Launch the MetAssimulo 2.0 Dash web app with `python3 apps/index.py -p Input/parameters.txt` (after exporting PYTHONPATH) and open http://127.0.0.1:8060/. Run the 1D blood plasma 1H NMR simulation with the broad protein (albumin) background enabled — this enhancement is what makes blood spectra realistic — then compute the Pearson correlation between the simulated and real blood spectral intensity vectors. A Pearson correlation of approximately 0.82 against a real blood spectrum indicates the simulation was configured correctly; write the value to blood_spectrum_correlation.csv.

## Related tools

- **MetAssimulo 2.0** (Web application that simulates the 1D 1H NMR blood spectrum with the broad protein background used as the reference-like spectrum) — https://github.com/yanyan5420/MetAssimulo_2.git
- **conda** (Creates and activates the pinned Python environment (py37_metassimulo_2_env) required to run the simulator)
- **gdown** (Downloads the Input.zip simulation parameters data from Google Drive before running the simulation)

## Examples

```
conda activate py37_metassimulo_2_env; export PYTHONPATH=$HOME/MetAssimulo_2/:$PYTHONPATH; python3 apps/index.py -p Input/parameters.txt
```

## Evaluation signals

- Pearson correlation between simulated and real blood spectral intensity vectors is approximately 0.82 when the protein background is enabled
- Dash app confirms it is running: 'Dash is running on http://127.0.0.1:8060/' and the URL is reachable in a browser
- conda environment py37_metassimulo_2_env activates without errors after `conda env create -f environment.yml`
- Input/parameters.txt exists after unzipping Input.zip obtained via the gdown link
- blood_spectrum_correlation.csv is produced containing the computed Pearson correlation value

## Limitations

- Realism of blood spectra depends on enabling the broad protein background; without it, the approximation to real blood spectra is weaker
- Simulation requires an external data download from Google Drive (gdown), so the pipeline fails without network access to that link
- The pinned environment is Python 3.7 (py37_metassimulo_2_env), so newer Python environments may not reproduce the setup as documented

## Evidence

- [abstract] MetAssimulo 2.0 expands functionality to urine, blood, and cerebral spinal fluid, enhancing the realism of blood spectra by incorporating a broad protein background.: "MetAssimulo 2.0 expands functionality to urine, blood, and cerebral spinal fluid, enhancing the realism of blood spectra by incorporating a broad protein background."
- [abstract] This enhancement enables a closer approximation to real blood spectra, achieving a Pearson correlation of approximately 0.82.: "This enhancement enables a closer approximation to real blood spectra, achieving a Pearson correlation of approximately 0.82."
- [methods] git clone https://github.com/yanyan5420/MetAssimulo_2.git: "git clone https://github.com/yanyan5420/MetAssimulo_2.git"
- [readme] After successfully running the project using the above provided commands, you should see a message indicating: Dash is running on http://127.0.0.1:8060/: "After successfully running the project using the above provided commands, you should see a message indicating: 

`Dash is running on http://127.0.0.1:8060/`"
- [abstract] In this context, simulation of spectral data becomes a tractable solution for algorithm development.: "In this context, simulation of spectral data becomes a tractable solution for algorithm development."
