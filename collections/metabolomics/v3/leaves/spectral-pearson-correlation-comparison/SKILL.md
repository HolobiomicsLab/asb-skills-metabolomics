---
name: spectral-pearson-correlation-comparison
description: Use when you have a simulated 1D 1H NMR spectrum (e.g. MetAssimulo 2.0
  blood output) and a corresponding real spectrum, and you need a single numeric similarity
  score to validate simulation realism.
when_to_use_negative:
- The task is simulating urine or CSF spectra — MetAssimulo's realism enhancement
  via protein background applies to blood spectra specifically.
- You need a simulation rather than a similarity assessment — this skill only scores
  agreement between an existing simulated and real spectrum.
- The spectra are on incomparable intensity scales or aligned to different chemical-shift
  axes without prior alignment/preprocessing.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3292
tools:
- name: MetAssimulo 2.0
  role: Generates the simulated 1D blood plasma 1H NMR spectrum with the broad protein
    background that is correlated against the real spectrum
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: Python
  role: Implementation language of the MetAssimulo 2.0 web app; used to load the two
    spectral intensity vectors and compute the Pearson correlation
- name: conda
  role: Creates and activates the py37_metassimulo_2_env environment needed to run
    the simulation before correlation
- name: gdown
  role: Downloads the Input.zip simulation input data (including parameters.txt) from
    Google Drive before running the app
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf045
    title: 'MetAssimulo 2.0: a web app for simulating realistic 1D and 2D metabolomic
      1H NMR spectra'
  source_content_hash: sha256:82933d71285ed8b97d5146ca86fe65bd789108cb085c7cd9eb39a92c219a3a2a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yanyan5420/MetAssimulo_2.git
  tools:
  - Anaconda
  - Git
  - MetAssimulo 2.0
  - Miniconda
  - Python
  - conda
  - gdown
  - pip
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# spectral-pearson-correlation-comparison

## Summary

Quantifies how closely a simulated 1D 1H NMR spectrum (e.g. a MetAssimulo 2.0 blood plasma simulation with broad protein background) matches a real spectrum by computing the Pearson correlation coefficient between their spectral intensity vectors.

## When to use

Use when you have a simulated 1D 1H NMR spectrum (e.g. MetAssimulo 2.0 blood output) and a corresponding real spectrum, and you need a single numeric similarity score to validate simulation realism.

## When NOT to use

- The task is simulating urine or CSF spectra — MetAssimulo's realism enhancement via protein background applies to blood spectra specifically.
- You need a simulation rather than a similarity assessment — this skill only scores agreement between an existing simulated and real spectrum.
- The spectra are on incomparable intensity scales or aligned to different chemical-shift axes without prior alignment/preprocessing.

## Inputs

- Simulated 1D blood plasma 1H NMR spectrum from MetAssimulo 2.0 (protein/albumin background enabled, parameters from Input/parameters.txt)
- Real (experimental) blood 1D 1H NMR spectrum as a spectral intensity vector

## Outputs

- Pearson correlation coefficient between simulated and real spectral intensity vectors (~0.82 expected)
- blood_spectrum_correlation.csv recording the computed correlation value

## How to apply

Run the MetAssimulo 2.0 web app (Dash on http://127.0.0.1:8060/) with Input/parameters.txt, and simulate the 1D blood plasma 1H NMR spectrum with the broad protein (albumin) background enabled, since without it blood simulations are less realistic. Load both the simulated spectrum and the real blood spectrum as spectral intensity vectors on a common scale, then compute the Pearson correlation coefficient between the paired intensity vectors. Accept the simulation as realistic when the correlation reaches approximately 0.82, as reported in the article; write the computed value to blood_spectrum_correlation.csv for traceability. The rationale is that the broad protein background dominates much of the real blood spectral envelope, so its inclusion is what drives the correlation up from a metabolites-only simulation.

## Related tools

- **MetAssimulo 2.0** (Generates the simulated 1D blood plasma 1H NMR spectrum with the broad protein background that is correlated against the real spectrum) — https://github.com/yanyan5420/MetAssimulo_2.git
- **Python** (Implementation language of the MetAssimulo 2.0 web app; used to load the two spectral intensity vectors and compute the Pearson correlation)
- **conda** (Creates and activates the py37_metassimulo_2_env environment needed to run the simulation before correlation)
- **gdown** (Downloads the Input.zip simulation input data (including parameters.txt) from Google Drive before running the app)

## Examples

```
git clone https://github.com/yanyan5420/MetAssimulo_2.git; conda activate py37_metassimulo_2_env; export PYTHONPATH=$HOME/MetAssimulo_2/:$PYTHONPATH; python3 apps/index.py -p Input/parameters.txt
```

## Evaluation signals

- Computed Pearson correlation against the real blood spectrum should be approximately 0.82 when the protein background is enabled
- The value written to blood_spectrum_correlation.csv matches the correlation recomputed from the spectral intensity vectors
- The Dash app reports 'Dash is running on http://127.0.0.1:8060/' before simulation, confirming correct setup
- Both spectra are represented as intensity vectors over the same spectral axis before correlation

## Limitations

- The ~0.82 correlation applies to blood spectra with the broad protein background enabled; metabolites-only simulations will approximate real blood spectra less closely.
- MetAssimulo 2.0 is a simulation tool — the correlation validates realism but does not quantify biological validity of individual metabolite concentrations.
- No changelog was found for the repository (per the article's missing-signals note), so version drift in outputs is possible.

## Evidence

- [abstract] This enhancement enables a closer approximation to real blood spectra, achieving a Pearson correlation of approximately 0.82.: "This enhancement enables a closer approximation to real blood spectra, achieving a Pearson correlation of approximately 0.82."
- [abstract] enhancing the realism of blood spectra by incorporating a broad protein background: "MetAssimulo 2.0 expands functionality to urine, blood, and cerebral spinal fluid, enhancing the realism of blood spectra by incorporating a broad protein background."
- [methods] conda env create -f environment.yml
conda activate py37_metassimulo_2_env: "conda env create -f environment.yml
conda activate py37_metassimulo_2_env"
- [readme] Dash is running on http://127.0.0.1:8060/: "`Dash is running on http://127.0.0.1:8060/`"
