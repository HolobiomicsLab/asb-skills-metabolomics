---
name: spectral-overlap-comparison-urine-metabolites
description: Use when you have a simulated metabolomic 1H NMR (1D or 2D J-Res) urine
  spectrum and a real deposited reference spectrum and need to assess how well simulated
  peaks reproduce real high/medium-abundance metabolites such as creatinine and citric
  acid.
when_to_use_negative:
- The matrix of interest is blood or CSF — the comparison markers (creatinine, citric
  acid) and 0.8–4.5 ppm urine workflow are urine-specific, though MetAssimulo 2.0
  also supports blood and cerebral spinal fluid
- You need a densely populated simulated spectrum matching every real peak — only
  52 metabolites are simulated, so the simulated spectrum is intentionally less densely
  populated than the real one
- The task is 1H-NMR spectral processing of raw FIDs (phasing, baseline correction)
  rather than simulation-vs-real comparison — this skill assumes pre-processed deposited
  spectra
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: MetAssimulo 2.0
  role: Generates the simulated urine J-Res spectrum with pH-based chemical-shift
    adjustment of 52 metabolites before comparison against the real spectrum
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: Python
  role: Runs the MetAssimulo 2.0 Dash web app and performs skyline projection, 0.8–4.5
    ppm restriction, normalisation and Pearson-correlation computation
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: gdown
  role: Downloads the Input.zip bundle containing simulation parameters and metabolite
    spectra from Google Drive
- name: conda
  role: Creates and activates the py37_metassimulo_2_env environment required to run
    the comparison workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf045
    title: 'MetAssimulo 2.0: a web app for simulating realistic 1D and 2D metabolomic
      1H NMR spectra'
  source_content_hash: sha256:f9a26b2ec9ef42ed08227995f0d294672db3feb245e7e148e40fa7c094c95caf
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
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# spectral-overlap-comparison-urine-metabolites

## Summary

Comparison skill that overplots a MetAssimulo 2.0-simulated 2D J-Res urine spectrum (skyline-projected to 1D) on a real urine J-Res reference spectrum (MTBLS12081) over 0.8–4.5 ppm, then quantifies agreement via peak coincidence and Pearson correlation of aligned projections.

## When to use

Use when you have a simulated metabolomic 1H NMR (1D or 2D J-Res) urine spectrum and a real deposited reference spectrum and need to assess how well simulated peaks reproduce real high/medium-abundance metabolites such as creatinine and citric acid.

## When NOT to use

- The matrix of interest is blood or CSF — the comparison markers (creatinine, citric acid) and 0.8–4.5 ppm urine workflow are urine-specific, though MetAssimulo 2.0 also supports blood and cerebral spinal fluid
- You need a densely populated simulated spectrum matching every real peak — only 52 metabolites are simulated, so the simulated spectrum is intentionally less densely populated than the real one
- The task is 1H-NMR spectral processing of raw FIDs (phasing, baseline correction) rather than simulation-vs-real comparison — this skill assumes pre-processed deposited spectra

## Inputs

- MetAssimulo 2.0 input parameters file (Input/parameters.txt from the downloaded Input.zip)
- Metabolite reference NMR spectra bundled in Input.zip (gdown Google Drive download)
- Real human urine J-Res spectrum from the MetaboLights MTBLS12081 deposit

## Outputs

- Simulated 2D J-Res urine spectrum and its skyline (J-dimension) projection
- Overlaid comparison figure of projected simulated vs real urine spectrum (0.8–4.5 ppm)
- Similarity metrics (Pearson correlation of aligned projections) and reduced spectral density estimate

## How to apply

Configure MetAssimulo 2.0 (started via `python3 apps/index.py -p Input/parameters.txt`, Dash at http://127.0.0.1:8060) for the urine matrix with 52 metabolites, enabling pH-based chemical-shift adjustment of resonance positions, and generate the 2D J-Res spectrum. Compute the skyline projection along the vertical J-dimension to obtain a 1D-like profile. Restrict both the simulated and the real urine J-Res spectra (downloaded from the Input.zip bundle / MetaboLights MTBLS12081 deposit) to the 0.8–4.5 ppm window and normalise/rescale intensities so they are directly comparable. Overplot the projected simulated spectrum on the real spectrum and quantify which metabolite peaks coincide, focusing on creatinine and citric acid as the high/medium-abundance markers reported in section 3.2.1. Compute a similarity metric such as Pearson correlation of the aligned projections (the paper reports ~0.82 for the analogous blood-spectrum comparison) and record the reduced spectral density of the simulated relative to the real spectrum.

## Related tools

- **MetAssimulo 2.0** (Generates the simulated urine J-Res spectrum with pH-based chemical-shift adjustment of 52 metabolites before comparison against the real spectrum) — https://github.com/yanyan5420/MetAssimulo_2.git
- **Python** (Runs the MetAssimulo 2.0 Dash web app and performs skyline projection, 0.8–4.5 ppm restriction, normalisation and Pearson-correlation computation) — https://github.com/yanyan5420/MetAssimulo_2.git
- **gdown** (Downloads the Input.zip bundle containing simulation parameters and metabolite spectra from Google Drive)
- **conda** (Creates and activates the py37_metassimulo_2_env environment required to run the comparison workflow)

## Examples

```
python3 apps/index.py -p Input/parameters.txt
```

## Evaluation signals

- Peaks of high/medium-abundance metabolites — creatinine and citric acid — in the projected simulated spectrum coincide with the real urine J-Res spectrum
- The simulated spectrum should be visibly less densely populated than the real spectrum, consistent with only 52 metabolites being simulated
- Both spectra are restricted to the identical 0.8–4.5 ppm window and intensity-normalised before comparison
- A Pearson correlation of the aligned skyline projections is computed and recorded alongside the reduced spectral density ratio
- The Dash server confirms 'Dash is running on http://127.0.0.1:8060/' before simulation, indicating the environment was set up correctly

## Limitations

- Only 52 metabolites are simulated for urine, so the simulated J-Res spectrum is less densely populated than a real urine spectrum and low-abundance metabolites will not overlap
- The comparison accuracy depends on pH-based chemical-shift adjustment being enabled; without it resonance positions may be misaligned with the real spectrum
- The paper's quantitative benchmark (~0.82 Pearson correlation) was established for blood spectra with a protein background, not for the urine J-Res comparison, so no fixed correlation threshold exists for this skill

## Evidence

- [abstract] MetAssimulo 2.0 expands functionality to urine, blood, and cerebral spinal fluid, enhancing the realism of blood spectra by incorporating a broad protein background.: "MetAssimulo 2.0 expands functionality to urine, blood, and cerebral spinal fluid, enhancing the realism of blood spectra by incorporating a broad protein background."
- [abstract] The relevant NMR spectra for metabolites are deposited in MetaboLights with accession number MTBLS12081.: "The relevant NMR spectra for metabolites are deposited in MetaboLights with accession number MTBLS12081."
- [readme] pip install gdown; gdown https://drive.google.com/uc?id=12k-nbIcoME5GSkD8u1iYUcNzg6l-7msW; unzip Input.zip: "pip install gdown
gdown https://drive.google.com/uc?id=12k-nbIcoME5GSkD8u1iYUcNzg6l-7msW
unzip Input.zip"
- [readme] conda env create -f environment.yml; conda activate py37_metassimulo_2_env: "conda env create -f environment.yml
conda activate py37_metassimulo_2_env"
- [abstract] This enhancement enables a closer approximation to real blood spectra, achieving a Pearson correlation of approximately 0.82.: "This enhancement enables a closer approximation to real blood spectra, achieving a Pearson correlation of approximately 0.82."
