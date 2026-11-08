---
name: spectrum-prediction-inference-benchmarking
description: Use when you need to quantify and compare the computational cost (per-molecule
  prediction time) of a learned mass spectral prediction model against published quantum-chemical
  or CFM-EI baselines.
when_to_use_negative:
- You need to benchmark spectral accuracy (recall-at-10, library matching) rather
  than prediction runtime
- The task involves modeling isotope intensity peaks in ion fragments, which NEIMS
  does not support
- The molecules of interest are outside the small-molecule EI regime the NEIMS model
  was trained and evaluated on
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: Tensorflow
  role: Framework in which the NEIMS MLP is constructed; its runtime used for timed
    inference of ECFP fingerprint inputs
  repo: github.com/brain-research/deep-molecular-massspec
- name: RDKit Cheminformatics package
  role: Converts input molecules to additive ECFP fingerprints (length 4096, radius
    2) fed to the model
- name: NEIMS
  role: The benchmarked model itself; a lightweight network predicting EI spectra
    at ~5 ms per molecule
  repo: github.com/brain-research/deep-molecular-massspec
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acscentsci.9b00085
    title: Rapid Prediction of Electron–Ionization Mass Spectrometry Using Neural
      Networks
  source_content_hash: sha256:b359e6d6fb96616942f1986f199f4508712e0f258575c77f9988d9d7a7e0a1db
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - NEIMS
  - RDKit Cheminformatics package
  - Tensorflow
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# spectrum-prediction-inference-benchmarking

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Benchmarks the per-molecule inference runtime of the NEIMS neural network for electron-ionization mass spectral prediction against slower CFM-EI and first-principles QCEIMS reference methods, demonstrating orders-of-magnitude speedup (5 ms vs ~1000 s per molecule).

## When to use

Use when you need to quantify and compare the computational cost (per-molecule prediction time) of a learned mass spectral prediction model against published quantum-chemical or CFM-EI baselines.

## When NOT to use

- You need to benchmark spectral accuracy (recall-at-10, library matching) rather than prediction runtime
- The task involves modeling isotope intensity peaks in ion fragments, which NEIMS does not support
- The molecules of interest are outside the small-molecule EI regime the NEIMS model was trained and evaluated on

## Inputs

- Small-molecule structures (e.g., SDF or SMILES from the NIST 2017 Mass Spectral Library)
- Pretrained NEIMS TensorFlow model weights (massspec_weights from github.com/brain-research/deep-molecular-massspec)
- Published per-molecule runtime figures for CFM-EI (up to ~10 min) and QCEIMS-style first-principles methods (>=1000 s)

## Outputs

- Measured average per-molecule NEIMS prediction time in milliseconds
- runtime_comparison.csv tabulating NEIMS runtime vs CFM-EI and QCEIMS reference runtimes

## How to apply

Load the pretrained NEIMS TensorFlow model checkpoint — the optimal MLP has seven layers of 2000 nodes with residual connections, ReLU activation, and dropout 0.25 — from the deep-molecular-massspec repository. Prepare small-molecule inputs (e.g., from the NIST 2017 Mass Spectral Library) as additive ECFP fingerprints of length 4096 with radius 2 using RDKit. Run inference through the bidirectional prediction head (forward + reverse + gate combination), timing each per-molecule prediction, and compute the average runtime, which should be on the order of 5 ms. Tabulate the measured NEIMS runtime next to published baselines: CFM-EI, which can take up to 10 min for large molecules, and QCEIMS-style first-principles methods requiring 1000 s or more per molecule. Write the comparison to runtime_comparison.csv.

## Related tools

- **Tensorflow** (Framework in which the NEIMS MLP is constructed; its runtime used for timed inference of ECFP fingerprint inputs) — github.com/brain-research/deep-molecular-massspec
- **RDKit Cheminformatics package** (Converts input molecules to additive ECFP fingerprints (length 4096, radius 2) fed to the model)
- **NEIMS** (The benchmarked model itself; a lightweight network predicting EI spectra at ~5 ms per molecule) — github.com/brain-research/deep-molecular-massspec

## Examples

```
python make_spectra_prediction.py --input_file=examples/pentachlorobenzene.sdf --output_file=/tmp/annotated.sdf --weights_dir=$MODEL_WEIGHTS_DIR/massspec_weights
```

## Evaluation signals

- Measured average per-molecule runtime should be on the order of milliseconds (paper reports 5 ms per molecule)
- Measured NEIMS runtime should be several orders of magnitude below the CFM-EI reference (up to 10 min for large molecules) and the >=1000 s QCEIMS reference
- Inference pipeline should reproduce the bidirectional (forward + reverse + gate) prediction head, not a single-direction output
- Fingerprint conversion must use ECFP length 4096 with radius 2 to match the trained model's input dimensionality
- Timing summary must be exported to runtime_comparison.csv with one row per method compared

## Limitations

- NEIMS currently does not have a method to model intensity peaks corresponding to isotopes in ion fragments
- Reported runtimes for CFM-EI and QCEIMS are taken from the literature rather than re-measured on the same hardware
- Runtime depends on CPU hardware; no software version numbers are pinned in the paper (README suggests tensorflow=1.13.2 and Python 3.6)

## Evidence

- [readme] Model weights download and prediction entry point: [evidence span withheld — anchor status: external_doc]
