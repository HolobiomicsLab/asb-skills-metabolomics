---
name: scaffold-conditioned-de-novo-graph-generation
description: Use when you have an MS/MS spectrum plus a candidate scaffold (predicted
  or oracle) and need to generate full candidate molecular structures for top-k accuracy
  or Tanimoto-based evaluation.
when_to_use_negative:
- You do not have a scaffold (predicted or oracle) and cannot run scaffold retrieval
  first — the generator is scaffold-conditioned and requires a starting scaffold.
- Your spectra are not MS/MS tandem spectra preprocessed with intensity normalization
  and low-intensity peak removal; MADGEN was built for NIST23, CANOPUS, and MassSpecGym-style
  MS/MS data.
- NIST23 data is needed but not separately licensed — the NIST dataset is commercially
  available and cannot be pulled from the public Zenodo records.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MADGEN
  role: 'Scaffold-conditioned generation model: Markov bridge generation with classifier-free
    guidance from the mass spectrum, run via sample.py'
  repo: https://github.com/HassounLab/MADGEN
- name: RDKit
  role: Extracts scaffolds from molecular graph representations to build the oracle
    lookup table and computes scaffold coverage during evaluation
  repo: https://github.com/HassounLab/MADGEN
- name: PyTorch
  role: Runs the checkpoint-based sampling and attention-based generation under CUDA
  repo: https://github.com/HassounLab/MADGEN
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:693efd82e789062dea7a679a19c79d184c06b1fda058bc6ea4847a70a00ad3f3
  source_papers:
  - doi: 10.48550/arxiv.2501.01950
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HassounLab/MADGEN
  tools:
  - MADGEN
  - PyTorch
  - Python
  - RDKit
  - conda
  - pip
  - wget
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5652173913043478
n_claims: 23
---

# scaffold-conditioned-de-novo-graph-generation

## Summary

Generate de novo molecular structures from MS/MS spectra by starting from a retrieved scaffold and using spectrum-guided attention to complete the molecule (MADGEN's Markov bridge generation with classifier-free spectral guidance).

## When to use

Use when you have an MS/MS spectrum plus a candidate scaffold (predicted or oracle) and need to generate full candidate molecular structures for top-k accuracy or Tanimoto-based evaluation.

## When NOT to use

- You do not have a scaffold (predicted or oracle) and cannot run scaffold retrieval first — the generator is scaffold-conditioned and requires a starting scaffold.
- Your spectra are not MS/MS tandem spectra preprocessed with intensity normalization and low-intensity peak removal; MADGEN was built for NIST23, CANOPUS, and MassSpecGym-style MS/MS data.
- NIST23 data is needed but not separately licensed — the NIST dataset is commercially available and cannot be pulled from the public Zenodo records.

## Inputs

- Processed MS/MS dataset pickle (msgym.pkl or canopus.pkl from Zenodo record 15036069)
- Pre-trained MADGEN checkpoint for the matching dataset (Zenodo record 17115068)
- Scaffold source: RDKit-extracted scaffold lookup table keyed by spectrum and chemical formula (oracle), or pre-computed predictive retrieval ranks (ranks_msgym_pred.pkl / ranks_canopus_pred.pkl)
- YAML config for the dataset (configs/msgym.yaml, configs/canopus.yaml, configs/nist.yaml)

## Outputs

- CSV of generation predictions with columns true, pred, scaffold
- Top-K accuracy (InChI key matching), Maximum Tanimoto Similarity (Morgan radius=2, 2048 bits), and scaffold coverage metrics
- Aggregated per-dataset/retriever-setting results table (madgen_accuracy_results.csv)

## How to apply

Run sample.py with the dataset config (e.g. configs/msgym.yaml), the dataset-matched checkpoint from Zenodo record 17115068, --mode test, --n_steps 100, and --sampling_seed 42; the model starts generation from the retrieved scaffold and guides an attention-based generative model with the MS/MS spectrum via classifier-free guidance. For the oracle setting, seed the scaffold from an RDKit-extracted lookup table keyed by spectrum and chemical formula; for the predictive setting, start from scaffolds ranked by the pre-computed ranks_msgym_pred.pkl / ranks_canopus_pred.pkl files. Assemble generated candidates into a CSV with true, pred, and scaffold columns (pred repeated for multiple samples, e.g. n_samples 50). Evaluate with evaluation_generation.py to compute Top-K accuracy via InChI key matching, Maximum Tanimoto Similarity with Morgan fingerprints (radius 2, 2048 bits), and scaffold coverage. Scaffold conditioning is expected to dominate accuracy: with the oracle retriever MADGEN reaches 49.0% top-1 on NIST23 versus 4.6% with the predictive retriever, so always record which retriever setting produced the scaffold.

## Related tools

- **MADGEN** (Scaffold-conditioned generation model: Markov bridge generation with classifier-free guidance from the mass spectrum, run via sample.py) — https://github.com/HassounLab/MADGEN
- **RDKit** (Extracts scaffolds from molecular graph representations to build the oracle lookup table and computes scaffold coverage during evaluation) — https://github.com/HassounLab/MADGEN
- **PyTorch** (Runs the checkpoint-based sampling and attention-based generation under CUDA) — https://github.com/HassounLab/MADGEN

## Examples

```
CUDA_VISIBLE_DEVICES=1 python sample.py --config configs/msgym.yaml --checkpoint checkpoints/msgym_best.pt --model Madgen --mode test --n_steps 100 --sampling_seed 42
```

## Evaluation signals

- Reproduced top-1 accuracy should match published values, e.g. 49.0% top-1 on NIST23 with the oracle retriever and 2.10% top-1 on CANOPUS/MassSpecGym with the predictive retriever.
- Oracle vs predictive gap must be large (NIST23: 49.0% top-1 oracle vs 4.6% top-1 predictive); a small gap indicates the wrong scaffold source was wired in.
- Prediction CSV must parse with exactly the columns true, pred, scaffold and pred rows repeated for multiple samples.
- Sampling must use the fixed settings (--n_steps 100, --sampling_seed 42) for reproducible regeneration.
- Evaluation must report Top-K accuracy via InChI key matching and Tanimoto similarity using Morgan fingerprints radius=2, 2048 bits.

## Limitations

- Generation accuracy depends dramatically on scaffold correctness: with the predictive retriever top-1 accuracy collapses (e.g. 4.6% on NIST23).
- NIST23 is commercially licensed and must be acquired separately from NIST, so full three-dataset reproduction requires licensing.
- Only MSGym and CANOPUS pre-trained checkpoints are publicly hosted on Zenodo record 17115068; the NIST checkpoint requires the licensed data.
- MSNovelist baseline results are limited to accuracy metrics; other measures (similarity, MCES) are not available for comparison.

## Evidence

- [results] Oracle vs predictive accuracy finding: [evidence span withheld — anchor status: not_found]
