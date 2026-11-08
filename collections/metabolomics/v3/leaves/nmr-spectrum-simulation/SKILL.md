---
name: nmr-spectrum-simulation
description: Use when you need synthetic metabolomic 1H NMR spectra (1D, 2D J-Res,
  or 2D COSY) of urine, blood, or CSF mixtures — e.g. for benchmarking spectral processing
  algorithms or reproducing paper comparisons against real spectra.
when_to_use_negative:
- You need simulation of NMR modalities outside 1D 1H, 2D J-Res, and 2D COSY (e.g.
  HSQC, NOESY) — MetAssimulo 2.0's scope is 1D and 2D J-Res/COSY metabolomic 1H spectra
- You need spectra of biofluids other than urine, blood, or cerebral spinal fluid
- You already have real experimental NMR spectra and only need processing/statistical
  analysis rather than synthetic data generation
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3292
tools:
- name: MetAssimulo 2.0
  role: 'Core simulation engine: generates 1D and 2D (J-Res, COSY) metabolomic 1H
    NMR spectra of urine, blood, and CSF with discrete/continuous outcomes, protein
    background, and inter-metabolite correlations'
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: conda
  role: Creates and activates the pinned py37_metassimulo_2_env environment from environment.yml
- name: gdown
  role: Downloads the Input.zip metabolite reference spectra data from Google Drive
    before simulation
- name: Python
  role: Runs the Dash web app (apps/index.py) and post-simulation comparison/analysis
    scripts (e.g. skyline projection, Pearson correlation overlays)
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf045
    title: 'MetAssimulo 2.0: a web app for simulating realistic 1D and 2D metabolomic
      1H NMR spectra'
  source_content_hash: sha256:358301c0ec73c48049c6b9bc0f0a81791b7eef353c83d54b812593357f902b9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yanyan5420/MetAssimulo_2.git
  tools:
  - MetAssimulo 2.0
  - Python
  - conda
  - gdown
  - pip
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# nmr-spectrum-simulation

## Summary

Simulates realistic 1D and 2D metabolomic 1H NMR spectra (J-Res and COSY) of urine, blood, and CSF using MetAssimulo 2.0, a Python-based Dash web app, to generate synthetic spectral data for algorithm development and method validation.

## When to use

Use when you need synthetic metabolomic 1H NMR spectra (1D, 2D J-Res, or 2D COSY) of urine, blood, or CSF mixtures — e.g. for benchmarking spectral processing algorithms or reproducing paper comparisons against real spectra.

## When NOT to use

- You need simulation of NMR modalities outside 1D 1H, 2D J-Res, and 2D COSY (e.g. HSQC, NOESY) — MetAssimulo 2.0's scope is 1D and 2D J-Res/COSY metabolomic 1H spectra
- You need spectra of biofluids other than urine, blood, or cerebral spinal fluid
- You already have real experimental NMR spectra and only need processing/statistical analysis rather than synthetic data generation

## Inputs

- MetAssimulo 2.0 parameters file (Input/parameters.txt) defining mixture composition and spectral parameters
- Metabolite reference spectra from the Input.zip download / MetaboLights MTBLS12081 deposit
- Optional real reference NMR spectra (e.g. real urine J-Res) for overlay comparison

## Outputs

- Simulated 1D 1H NMR spectrum of the selected biofluid
- Simulated 2D J-Res spectrum and/or 2D COSY spectrum (with or without peak shifts)
- Overlaid simulated-vs-real comparison figures and similarity metrics (e.g. Pearson correlation of aligned projections)

## How to apply

Clone the MetAssimulo_2 repository and create the pinned Python 3.7 environment with `conda env create -f environment.yml; conda activate py37_metassimulo_2_env`. Download the metabolite spectral reference data with gdown (Input.zip, tied to the MetaboLights MTBLS12081 deposit) and unzip it into the project root. Launch the Dash web app via `python3 apps/index.py -p Input/parameters.txt` after exporting PYTHONPATH, then configure the simulation in the browser: choose the biofluid matrix, the number of metabolites (e.g. 52 for urine), matrix type (discrete case-control or continuous outcomes like BMI), and whether to enable pH-based chemical-shift adjustment, protein background (for blood), inter-metabolite correlations, or 2D peak shifts (COSY can be simulated without peak shifts per Supplementary Figs S2–S3). Run the simulation and export the simulated spectrum, then validate by comparing against real spectra — e.g. skyline-project the simulated 2D J-Res along the J-dimension, restrict to the 0.8–4.5 ppm window, overlay on the real urine spectrum, and quantify agreement (the paper reports Pearson correlation ≈ 0.82 for simulated vs real blood spectra with protein background).

## Related tools

- **MetAssimulo 2.0** (Core simulation engine: generates 1D and 2D (J-Res, COSY) metabolomic 1H NMR spectra of urine, blood, and CSF with discrete/continuous outcomes, protein background, and inter-metabolite correlations) — https://github.com/yanyan5420/MetAssimulo_2.git
- **conda** (Creates and activates the pinned py37_metassimulo_2_env environment from environment.yml)
- **gdown** (Downloads the Input.zip metabolite reference spectra data from Google Drive before simulation)
- **Python** (Runs the Dash web app (apps/index.py) and post-simulation comparison/analysis scripts (e.g. skyline projection, Pearson correlation overlays)) — https://github.com/yanyan5420/MetAssimulo_2.git

## Examples

```
git clone https://github.com/yanyan5420/MetAssimulo_2.git; cd MetAssimulo_2; conda activate py37_metassimulo_2_env; export PYTHONPATH=$HOME/MetAssimulo_2/:$PYTHONPATH; python3 apps/index.py -p Input/parameters.txt
```

## Evaluation signals

- The Dash app reports 'Dash is running on http://127.0.0.1:8060/' after launch, confirming correct environment and PYTHONPATH setup
- For J-Res validation: the simulated urine spectrum (52 metabolites) should show reduced spectral density vs the real spectrum but preserve high/medium-abundance peaks such as creatinine and citric acid in the 0.8–4.5 ppm window
- For blood spectra: simulated spectra with protein background should achieve a Pearson correlation of approximately 0.82 against real blood spectra
- For COSY: the exported output file exists and is a valid spectral format consistent with a 2D COSY simulation of the specified mixture (diagonal-projection peak detection runs without error)
- The conda environment py37_metassimulo_2_env resolves without dependency conflicts, and apps/index.py runs with Input/parameters.txt without import errors

## Limitations

- Simulated urine J-Res spectra are less densely populated than real spectra (only 52 metabolites simulated), so low-abundance metabolites present in real spectra may be missing
- No changelog is available for the repository, so version-to-version behavior changes cannot be tracked from documentation
- Requires a Python 3.7 (py37_metassimulo_2_env) environment and local Dash server; the web app runs on http://127.0.0.1:8060/ rather than being a hosted service

## Evidence

- [abstract] MetAssimulo 2.0 expands functionality to urine, blood, and cerebral spinal fluid, enhancing the realism of blood spectra by incorporating a broad protein background.: "MetAssimulo 2.0 expands functionality to urine, blood, and cerebral spinal fluid, enhancing the realism of blood spectra by incorporating a broad protein background."
- [abstract] Here, we introduce MetAssimulo 2.0, a comprehensive upgrade of the MetAssimulo 1.b metabolomic 1H NMR simulation tool, reimplemented as a Python-based web application.: "Here, we introduce MetAssimulo 2.0, a comprehensive upgrade of the MetAssimulo 1.b metabolomic 1H NMR simulation tool, reimplemented as a Python-based web application."
- [abstract] This enhancement enables a closer approximation to real blood spectra, achieving a Pearson correlation of approximately 0.82.: "This enhancement enables a closer approximation to real blood spectra, achieving a Pearson correlation of approximately 0.82."
- [readme] conda env create -f environment.yml
conda activate py37_metassimulo_2_env: "conda env create -f environment.yml
conda activate py37_metassimulo_2_env"
- [methods] export PYTHONPATH=$HOME/MetAssimulo_2/:$PYTHONPATH
python3 apps/index.py -p Input/parameters.txt: "export PYTHONPATH=$HOME/MetAssimulo_2/:$PYTHONPATH
python3 apps/index.py -p Input/parameters.txt"
- [readme] Dash is running on http://127.0.0.1:8060/: "`Dash is running on http://127.0.0.1:8060/`"
