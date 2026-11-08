---
name: tandem-ms-spectrum-inference
description: Use when you have a molecular structure (e.g., a candidate from NIST20
  or PubChem) and need its predicted MS/MS spectrum for C2MS prediction or to augment
  library-based MS2C retrieval for compounds missing from spectral libraries.
when_to_use_negative:
- The molecule's spectrum is already in the NIST20/NIST23 spectral library — direct
  library matching is preferable, since C2MS prediction targets compounds missing
  from libraries
- The goal is MS2C (spectrum-to-compound) identification from an unknown spectrum
  rather than compound-to-spectrum (C2MS) prediction
- Spectra come from instrument platforms or precursor adducts outside the validated
  setting (merged [M+H]+ Orbitrap HCD spectra), since performance has not been evaluated
  under broader conditions
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: FraGNNet
  role: The probabilistic C2MS model that generates the fragmentation DAG and predicts
    the fragment/formula distribution underlying the spectrum
  repo: github.com/FraGNNet/fragnnet
- name: Cython
  role: Implements the recursive bond-breaking fragmentation algorithm that generates
    plausible molecule fragments
- name: PyTorch Geometric
  role: Implements the Molecule GNN and Fragment GNN graph neural network modules
    used for fragment distribution prediction
- name: RDKit
  role: Data preprocessing and molecule featurization of input structures into molecular
    graphs
- name: PyTorch Lightning
  role: Training/inference framework for the FraGNNet model and baselines
  repo: https://github.com/Lightning-AI/pytorch-lightning
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_content_hash: sha256:87a346b5c973864891dae069a2020a486848798678c57d0cd6ff2ff194aafe32
  source_papers:
  - doi: 10.48550/arxiv.2404.02360
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cython
  - FraGNNet
  - GrAFF-MS
  - PyTorch
  - PyTorch Geometric
  - PyTorch Lightning
  - Python
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# tandem-ms-spectrum-inference

## Summary

Predicts tandem MS/MS spectra directly from molecular structures using FraGNNet, a deep probabilistic GNN that combines a recursive bond-breaking fragmentation DAG with a two-stage graph neural network to induce a fragment distribution converted into a spectrum via formula masses.

## When to use

Use when you have a molecular structure (e.g., a candidate from NIST20 or PubChem) and need its predicted MS/MS spectrum for C2MS prediction or to augment library-based MS2C retrieval for compounds missing from spectral libraries.

## When NOT to use

- The molecule's spectrum is already in the NIST20/NIST23 spectral library — direct library matching is preferable, since C2MS prediction targets compounds missing from libraries
- The goal is MS2C (spectrum-to-compound) identification from an unknown spectrum rather than compound-to-spectrum (C2MS) prediction
- Spectra come from instrument platforms or precursor adducts outside the validated setting (merged [M+H]+ Orbitrap HCD spectra), since performance has not been evaluated under broader conditions

## Inputs

- Molecular structures (molecular graphs featurized from SMILES/InChI via RDKit)
- Trained FraGNNet-D3 or FraGNNet-D4 model checkpoints (github.com/FraGNNet/fragnnet)
- Fragmentation DAG hyperparameters: depth d in {3,4}, hydrogen tolerance j=4
- Held-out test spectra for evaluation (e.g., NIST20 MS/MS, [M+H]+ Orbitrap HCD)

## Outputs

- Predicted MS/MS spectrum as a set of peak masses with probabilities derived from the formula distribution P_theta(f)
- Latent formula distribution over molecule fragments (interpretable fragment/formula annotations)
- Comparison metrics against real spectra: C_BIN (0.01 Da bins) and C_HUN (10 ppm tolerance) cosine similarities
- Results table per model per split (e.g., spectrum_prediction_benchmark.csv)

## How to apply

First, generate the approximate fragmentation DAG M_hat(G, d, j) for the input molecular graph using the recursive bond-breaking algorithm (implemented in Cython), with depth d in {3,4} and hydrogen tolerance j=4, which the paper reports captures most total peak intensity in real MS/MS data. Then run the trained FraGNNet model — a Molecule GNN (g_theta^MOL) over the input molecular graph followed by a Fragment GNN (g_theta^FRAG) over the DAG — to predict a distribution over molecule fragments. The fragment distribution induces a formula distribution P_theta(f), which is converted to peak positions using exact formula masses, yielding a high-mass-accuracy predicted spectrum rather than a binned output. Evaluate against real spectra using binned cosine similarity C_BIN (0.01 Da bins) and Hungarian cosine similarity C_HUN (10 ppm peak-matching tolerance), and treat fragments whose masses fall far from predicted support as Outside-of-Support (OS) peaks.

## Related tools

- **FraGNNet** (The probabilistic C2MS model that generates the fragmentation DAG and predicts the fragment/formula distribution underlying the spectrum) — github.com/FraGNNet/fragnnet
- **Cython** (Implements the recursive bond-breaking fragmentation algorithm that generates plausible molecule fragments)
- **PyTorch Geometric** (Implements the Molecule GNN and Fragment GNN graph neural network modules used for fragment distribution prediction)
- **RDKit** (Data preprocessing and molecule featurization of input structures into molecular graphs)
- **PyTorch Lightning** (Training/inference framework for the FraGNNet model and baselines) — https://github.com/Lightning-AI/pytorch-lightning

## Examples

```
bash env/env_setup_gpu_py311_cu121_250602.sh && conda activate FRAGNNET-GPU && pip install -e ".[dev]"
```

## Evaluation signals

- FraGNNet-D4 should reach C_BIN ≈ 0.736 ± 0.002 on the NIST20 InChIKey split and ≈ 0.709 ± 0.002 on the scaffold split, outperforming ICEBERG, GrAFF-MS, MassFormer, and NEIMS baselines
- Formula masses of predicted peaks should match observed peaks within tight tolerance (annotations more than 10 ppm away from the peak mass are excluded), reflecting the model's high mass accuracy
- The fragmentation DAG with d in {3,4} and j=4 should capture most of the total peak intensity of real MS/MS data (low OS-peak intensity)
- Marginalized formula distribution P_theta(f) should have a median support size of roughly 679 formulae per test molecule (vs ~10,000 for GrAFF-MS's fixed library)
- Repeated evaluation over 5 random seeds should yield low standard deviation (mean ± std) per model per split

## Limitations

- The fragmentation algorithm depends on recursive edge-removal operations that are not parallelized, limiting runtime
- The algorithm fails to account for some fragments, evidenced by unexplained OS peaks in the spectra
- Unexplained peaks may arise from complex chemistry such as cyclizations, which dramatically expands the fragment search space and is challenging to model
- Performance has not been evaluated on unmerged spectra or under broader experimental conditions (different instrument platforms and precursor adducts)

## Evidence

- [methods] calculating M_hat(G,d,j) with d in {3,4} and j=4 can effectively capture most of the total peak intensity in real MS/MS data: [evidence span withheld — anchor status: not_found]
- [methods] The recursive fragmentation algorithm was implemented in Cython: [evidence span withheld — anchor status: not_found]
- [readme] FraGNNet is a deep probabilistic graph neural network (GNN) model for predicting tandem mass spectra (MS/MS) from molecular structures.: [evidence span withheld — anchor status: external_doc]
