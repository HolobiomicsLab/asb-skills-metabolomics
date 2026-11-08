---
name: jres-skyline-projection-generation
description: Use when you need realistic simulated 1H NMR spectra (1D or 2D J-Res/COSY)
  of urine, blood, or CSF for algorithm development or benchmarking, or when comparing
  a simulated spectrum against a real spectrum (e.g. urine 0.8–4.5 ppm J-Res).
when_to_use_negative:
- When the input is already a processed/real experimental spectrum and the goal is
  pure peak annotation rather than simulation — MetAssimulo generates simulated spectra,
  not annotations.
- When the target spectrum type or metabolite set is outside the tool's scope (e.g.
  matrices or spectral dimensions not covered by the tool's urine/blood/CSF 1H 1D
  and 2D J-Res/COSY simulation).
- When a fully dense spectrum identical to the real sample is required — the article
  notes the simulated urine J-Res spectrum is less densely populated (52 metabolites)
  than the real one.
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: MetAssimulo 2.0
  role: Web application (Dash) that performs the metabolomic 1H NMR spectrum simulation
    (1D and 2D J-Res/COSY) and generates the spectra used for projection and comparison.
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: gdown
  role: Downloads the Input.zip data bundle containing simulation parameters and metabolite
    spectra.
- name: Anaconda / Miniconda
  role: Environment manager that installs the project dependencies via environment.yml
    (py37_metassimulo_2_env).
- name: Git
  role: Clones the MetAssimulo_2 repository.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf045
    title: 'MetAssimulo 2.0: a web app for simulating realistic 1D and 2D metabolomic
      1H NMR spectra'
  source_content_hash: sha256:d39709c314520f53027a5d2529449fbeabc08b59003c722b8bbee4c08bd604db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yanyan5420/MetAssimulo_2.git
  tools:
  - Anaconda / Miniconda
  - Git
  - MetAssimulo 2.0
  - conda
  - gdown
  - pip
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# jres-skyline-projection-generation

## Summary

Generates simulated 2D J-resolved (J-Res) 1H NMR spectra of metabolomic samples using MetAssimulo 2.0 and projects them via a skyline (vertical) projection for direct visual and quantitative comparison with real spectra, e.g. human urine J-Res data from MetaboLights MTBLS12081.

## When to use

Use when you need realistic simulated 1H NMR spectra (1D or 2D J-Res/COSY) of urine, blood, or CSF for algorithm development or benchmarking, or when comparing a simulated spectrum against a real spectrum (e.g. urine 0.8–4.5 ppm J-Res).

## When NOT to use

- When the input is already a processed/real experimental spectrum and the goal is pure peak annotation rather than simulation — MetAssimulo generates simulated spectra, not annotations.
- When the target spectrum type or metabolite set is outside the tool's scope (e.g. matrices or spectral dimensions not covered by the tool's urine/blood/CSF 1H 1D and 2D J-Res/COSY simulation).
- When a fully dense spectrum identical to the real sample is required — the article notes the simulated urine J-Res spectrum is less densely populated (52 metabolites) than the real one.

## Inputs

- Input.zip bundle with Input/parameters.txt (simulation configuration) and metabolite NMR reference spectra
- Real 1H NMR J-Res spectra for comparison (e.g. human urine spectra from MetaboLights accession MTBLS12081)
- Parameters file specifying matrix (urine/blood/CSF), metabolite list, spectral type, and outcome mode

## Outputs

- Simulated 1D or 2D J-Res 1H NMR spectrum
- Skyline projection of the simulated J-Res spectrum for 1D-like comparison
- Overlaid simulated-vs-real comparison figure and similarity metrics (e.g. Pearson correlation)

## How to apply

Install MetAssimulo 2.0 from the GitHub repo (clone yanyan5420/MetAssimulo_2, create the py37_metassimulo_2_env conda environment from environment.yml), download the Input.zip data bundle with gdown, and launch the Dash web app with `python3 apps/index.py -p Input/parameters.txt`. Configure the simulation for the target matrix (urine, blood, or CSF), selecting the desired metabolite list (e.g. 52 urine metabolites), spectral type (1D or 2D J-Res), and outcome mode (discrete case-control or continuous such as body mass index), with pH-based chemical-shift adjustment of metabolite resonance positions. For blood spectra, enable the broad protein background to improve realism. For J-Res comparisons, restrict both simulated and real spectra to a common window (e.g. 0.8–4.5 ppm), normalise/rescale intensities, compute the skyline projection along the J dimension, and quantify agreement (e.g. Pearson correlation of aligned projections, noting reduced spectral density for simulated spectra).

## Related tools

- **MetAssimulo 2.0** (Web application (Dash) that performs the metabolomic 1H NMR spectrum simulation (1D and 2D J-Res/COSY) and generates the spectra used for projection and comparison.) — https://github.com/yanyan5420/MetAssimulo_2.git
- **gdown** (Downloads the Input.zip data bundle containing simulation parameters and metabolite spectra.)
- **Anaconda / Miniconda** (Environment manager that installs the project dependencies via environment.yml (py37_metassimulo_2_env).)
- **Git** (Clones the MetAssimulo_2 repository.)

## Examples

```
python3 apps/index.py -p Input/parameters.txt
```

## Evaluation signals

- The Dash web app starts successfully and reports 'Dash is running on http://127.0.0.1:8060/' after `python3 apps/index.py -p Input/parameters.txt`.
- The simulated J-Res spectrum restricted to 0.8–4.5 ppm contains recognizable peaks for high/medium-abundance metabolites such as creatinine and citric acid when compared with the real urine spectrum.
- A Pearson correlation of the aligned skyline projections between simulated and real spectra is computed; for blood spectra with protein background a correlation of ~0.82 is reported as a benchmark of realism.
- The simulated spectrum's peak density is reduced relative to the real spectrum (52 simulated metabolites), which is an expected, quantifiable feature rather than an error.

## Limitations

- Simulated spectra are less densely populated than real spectra because only a subset of metabolites (e.g. 52 for urine) is included; low-abundance features are not reproduced.
- Realism for blood spectra depends on the incorporated broad protein background; without it the approximation is poorer.
- The tool is reimplemented as a Python 3.7 environment (py37_metassimulo_2_env) and requires a conda-managed environment plus PYTHONPATH setup; the Input data must be fetched via gdown from a Google Drive link, so offline or mirrored setups may fail if the link changes.

## Evidence

- [abstract] MetAssimulo 2.0 expands functionality to urine, blood, and cerebral spinal fluid, enhancing the realism of blood spectra by incorporating a broad protein background.: "MetAssimulo 2.0 expands functionality to urine, blood, and cerebral spinal fluid, enhancing the realism of blood spectra by incorporating a broad protein background."
- [abstract] This enhancement enables a closer approximation to real blood spectra, achieving a Pearson correlation of approximately 0.82.: "This enhancement enables a closer approximation to real blood spectra, achieving a Pearson correlation of approximately 0.82."
- [results] The simulated urine J-Res spectrum is less densely populated than the real spectrum (52 metabolites simulated) but accurately reflects features of high and medium abundance metabolites such as creatinine and citric acid.: [evidence span withheld — anchor status: not_found]
- [readme] Dash is running on http://127.0.0.1:8060/: "`Dash is running on http://127.0.0.1:8060/`"
- [methods] conda env create -f environment.yml && conda activate py37_metassimulo_2_env: "conda env create -f environment.yml
conda activate py37_metassimulo_2_env"
