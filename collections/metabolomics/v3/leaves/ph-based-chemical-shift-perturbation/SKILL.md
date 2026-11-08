---
name: ph-based-chemical-shift-perturbation
description: Use when generating simulated 1D or 2D 1H NMR spectra (e.g. urine J-Res)
  whose peak positions must track pH variation, or when diagnosing why simulated metabolite
  peaks misalign with real reference spectra.
when_to_use_negative:
- You already have real experimental NMR spectra and only need processing/quantification
  — the perturbation only applies to simulated spectra produced by MetAssimulo 2.0.
- The matrix of interest is not urine, blood, or cerebral spinal fluid — MetAssimulo
  2.0's pH perturbation models these three biological matrices.
- pH is unknown or poorly constrained for the sample; shifting resonance positions
  to an arbitrary pH would degrade, not improve, realism.
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: MetAssimulo 2.0
  role: Web application that performs the pH-dependent adjustment of metabolite resonance
    positions as part of simulating 1D and 2D 1H NMR spectra for urine, blood, and
    CSF
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: Anaconda
  role: Manages the py37_metassimulo_2_env environment in which the MetAssimulo 2.0
    simulation, including the pH-based shift adjustment, is executed
- name: gdown
  role: Downloads the Input.zip containing parameter files and reference spectra required
    to run the pH-perturbed simulation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf045
    title: 'MetAssimulo 2.0: a web app for simulating realistic 1D and 2D metabolomic
      1H NMR spectra'
  source_content_hash: sha256:82ad40d49c9d1d872f10ff82db98e364e01b07fad6bf252097ca11bc60b6f9b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yanyan5420/MetAssimulo_2.git
  tools:
  - Anaconda
  - MetAssimulo 2.0
  - conda
  - gdown
  - pip
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# pH-based chemical shift perturbation

## Summary

A simulation step within MetAssimulo 2.0 in which each metabolite's 1H resonance positions are adjusted according to the sample matrix's pH before synthesising the NMR spectrum, so that simulated urine, blood, or CSF spectra reflect the pH-dependent chemical-shift behaviour observed in real spectra.

## When to use

Use when generating simulated 1D or 2D 1H NMR spectra (e.g. urine J-Res) whose peak positions must track pH variation, or when diagnosing why simulated metabolite peaks misalign with real reference spectra.

## When NOT to use

- You already have real experimental NMR spectra and only need processing/quantification — the perturbation only applies to simulated spectra produced by MetAssimulo 2.0.
- The matrix of interest is not urine, blood, or cerebral spinal fluid — MetAssimulo 2.0's pH perturbation models these three biological matrices.
- pH is unknown or poorly constrained for the sample; shifting resonance positions to an arbitrary pH would degrade, not improve, realism.

## Inputs

- MetAssimulo 2.0 parameter file (Input/parameters.txt) specifying metabolite list, matrix type (urine/blood/CSF), and pH
- Metabolite 1H chemical-shift reference data deposited at MetaboLights MTBLS12081
- Real reference NMR spectra (e.g. urine J-Res spectrum) for comparison

## Outputs

- pH-adjusted simulated 1D or 2D 1H NMR spectrum (e.g. 2D J-Res spectrum of urine)
- Skyline/1D projection of the simulated J-Res spectrum for overlay with real spectra

## How to apply

Configure MetAssimulo 2.0 (Python web app launched via `python3 apps/index.py -p Input/parameters.txt`) with the target matrix — urine, blood, or CSF — and supply metabolite resonance data together with a pH specification so that each metabolite's chemical shifts are recomputed at that pH rather than held at nominal values. For the urine J-Res example, the perturbation is applied to the 52 simulated metabolites before synthesising the 2D J-Res spectrum, ensuring peaks such as creatinine and citric acid land at positions comparable to the real MTBLS12081 reference. The perturbation is a preprocessing step inside the simulation pipeline, not a post-hoc peak shifting routine, so it must be set in the input parameters file before running the web app at http://127.0.0.1:8060/. Judge correctness by projecting the simulated J-Res onto its 1D skyline, restricting to the 0.8–4.5 ppm window, and overlaying on the real spectrum to confirm that high- and medium-abundance peaks align in both position and approximate intensity.

## Related tools

- **MetAssimulo 2.0** (Web application that performs the pH-dependent adjustment of metabolite resonance positions as part of simulating 1D and 2D 1H NMR spectra for urine, blood, and CSF) — https://github.com/yanyan5420/MetAssimulo_2.git
- **Anaconda** (Manages the py37_metassimulo_2_env environment in which the MetAssimulo 2.0 simulation, including the pH-based shift adjustment, is executed)
- **gdown** (Downloads the Input.zip containing parameter files and reference spectra required to run the pH-perturbed simulation)

## Examples

```
python3 apps/index.py -p Input/parameters.txt
```

## Evaluation signals

- The simulated urine J-Res spectrum contains the expected metabolite peaks (e.g. creatinine, citric acid) after pH adjustment, and its skyline projection restricted to 0.8–4.5 ppm shows peaks that coincide with the real urine spectrum
- Simulated blood spectra incorporating the pH-perturbed shifts and protein background achieve a Pearson correlation of approximately 0.82 with real blood spectra
- The generated spectrum reflects the matrix-specific pH rather than nominal chemical shifts, i.e. changing the pH parameter changes peak positions in the output
- Output artifacts include the simulated spectrum/projection and an overlaid comparison figure whose metabolite peaks align with the MTBLS12081 reference

## Limitations

- The simulated urine J-Res spectrum is less densely populated than the real spectrum because only 52 metabolites are simulated, so low-abundance peaks will be absent even with correct pH adjustment
- Functionality is limited to urine, blood, and cerebral spinal fluid matrices; other sample types are not supported by MetAssimulo 2.0
- Achieved Pearson correlation for blood (~0.82) indicates residual mismatch between simulated and real spectra even with the protein background and pH adjustments

## Evidence

- [results] pH-based chemical-shift adjustment of each metabolite's resonance positions: [evidence span withheld — anchor status: not_found]
- [abstract] MetAssimulo 2.0 expands functionality to urine, blood, and cerebral spinal fluid: "MetAssimulo 2.0 expands functionality to urine, blood, and cerebral spinal fluid, enhancing the realism of blood spectra"
- [abstract] The relevant NMR spectra for metabolites are deposited in MetaboLights with accession number MTBLS12081.: "The relevant NMR spectra for metabolites are deposited in MetaboLights with accession number MTBLS12081."
- [readme] python3 apps/index.py -p Input/parameters.txt: "python3 apps/index.py -p Input/parameters.txt"
- [abstract] achieving a Pearson correlation of approximately 0.82: "This enhancement enables a closer approximation to real blood spectra, achieving a Pearson correlation of approximately 0.82."
