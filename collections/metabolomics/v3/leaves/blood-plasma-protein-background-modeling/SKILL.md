---
name: blood-plasma-protein-background-modeling
description: Use when simulating blood plasma/serum 1H NMR spectra where metabolite-only
  peaks look unrealistically clean and a realistic broad macromolecular background
  is needed to approximate real blood spectra.
when_to_use_negative:
- The target biofluid is urine or cerebral spinal fluid — the broad protein background
  enhancement is specific to blood spectrum realism.
- You need a 2D simulation (J-Res or COSY); the protein-background blood realism result
  reported here applies to the 1D 1H blood plasma spectrum.
- You only need metabolite peak lists or concentrations rather than a simulated full
  spectrum with realistic background.
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: MetAssimulo 2.0
  role: Python-based web application that simulates the 1D blood plasma 1H NMR spectrum
    with the broad protein (albumin) background enabled
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: conda
  role: Creates and activates the py37_metassimulo_2_env environment from environment.yml
    to run MetAssimulo 2.0
- name: gdown
  role: Downloads the Input.zip simulation data (parameters.txt) from Google Drive
    before running the blood simulation
- name: Git
  role: Clones the yanyan5420/MetAssimulo_2 repository
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf045
    title: 'MetAssimulo 2.0: a web app for simulating realistic 1D and 2D metabolomic
      1H NMR spectra'
  source_content_hash: sha256:d57c1ecf2f7cd4192ba66a51d58ab5d68a4fca2a1815f307be9ac2257e024c83
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
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# blood-plasma-protein-background-modeling

## Summary

Enhances simulated blood 1H 1H NMR spectra by adding a broad protein (albumin) background, which brings the simulated spectrum close to a real blood spectrum (Pearson correlation ~0.82), as implemented in MetAssimulo 2.0.

## When to use

Use when simulating blood plasma/serum 1H NMR spectra where metabolite-only peaks look unrealistically clean and a realistic broad macromolecular background is needed to approximate real blood spectra.

## When NOT to use

- The target biofluid is urine or cerebral spinal fluid — the broad protein background enhancement is specific to blood spectrum realism.
- You need a 2D simulation (J-Res or COSY); the protein-background blood realism result reported here applies to the 1D 1H blood plasma spectrum.
- You only need metabolite peak lists or concentrations rather than a simulated full spectrum with realistic background.

## Inputs

- Input/parameters.txt blood simulation configuration from the MetAssimulo_2 repository Input folder (downloaded via gdown)
- Real blood 1H 1H NMR spectrum for comparison
- MetAssimulo 2.0 metabolite spectral data (MetaboLights accession MTBLS12081)

## Outputs

- Simulated 1D blood plasma 1H NMR spectrum including the broad protein background
- Pearson correlation coefficient (~0.82 expected) between simulated and real blood spectra, e.g. written to blood_spectrum_correlation.csv

## How to apply

Run MetAssimulo 2.0 locally: clone https://github.com/yanyan5420/MetAssimulo_2.git, download the Input data with gdown (Google Drive id 12k-nbIcoME5GSkD8u1iYUcNzg6l-7msW, unzip Input.zip to get Input/parameters.txt), and create the conda environment py37_metassimulo_2_env from environment.yml. Launch the web app with python3 apps/index.py -p Input/parameters.txt (after exporting PYTHONPATH) and open http://127.0.0.1:8060/. Configure a 1D blood plasma 1H NMR simulation via Input/parameters.txt with the broad protein (albumin) background enabled; this protein background is the key feature that differentiates realistic blood simulations from metabolite-only simulations. Validate by computing the Pearson correlation between the simulated spectral intensity vector and a real blood spectrum, expecting approximately 0.82, and record the result (e.g. blood_spectrum_correlation.csv).

## Related tools

- **MetAssimulo 2.0** (Python-based web application that simulates the 1D blood plasma 1H NMR spectrum with the broad protein (albumin) background enabled) — https://github.com/yanyan5420/MetAssimulo_2.git
- **conda** (Creates and activates the py37_metassimulo_2_env environment from environment.yml to run MetAssimulo 2.0)
- **gdown** (Downloads the Input.zip simulation data (parameters.txt) from Google Drive before running the blood simulation)
- **Git** (Clones the yanyan5420/MetAssimulo_2 repository)

## Examples

```
export PYTHONPATH=$HOME/MetAssimulo_2/:$PYTHONPATH; python3 apps/index.py -p Input/parameters.txt
```

## Evaluation signals

- Pearson correlation between the protein-background simulated blood spectrum and a real blood spectrum is approximately 0.82.
- Dash app reports 'Dash is running on http://127.0.0.1:8060/' and the URL loads in a browser.
- Input/parameters.txt exists after unzipping Input.zip obtained via gdown (id 12k-nbIcoME5GSkD8u1iYUcNzg6l-7msW).
- Conda environment py37_metassimulo_2_env activates without errors and apps/index.py runs with -p Input/parameters.txt.
- Correlation value is written to blood_spectrum_correlation.csv as the validation record.

## Limitations

- The ~0.82 Pearson correlation is an approximation, not an exact match — the protein background improves but does not perfectly reproduce real blood spectra.
- Requires Python 3.7 (py37_metassimulo_2_env) and a running local web app; the input data must be fetched from Google Drive, which may fail if the link or network is unavailable.
- Reported specifically for blood spectra; urine and CSF simulations are handled differently (no broad protein background mentioned for them).

## Evidence

- [abstract] enhancing the realism of blood spectra by incorporating a broad protein background: "MetAssimulo 2.0 expands functionality to urine, blood, and cerebral spinal fluid, enhancing the realism of blood spectra by incorporating a broad protein background."
- [abstract] Pearson correlation of approximately 0.82: "This enhancement enables a closer approximation to real blood spectra, achieving a Pearson correlation of approximately 0.82."
- [readme] run the project with parameters.txt: "python3 apps/index.py -p Input/parameters.txt"
- [readme] Dash web app URL: "Dash is running on http://127.0.0.1:8060/"
