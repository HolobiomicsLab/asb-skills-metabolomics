---
name: dataset-frequency-ranking-postprocessing
description: Use when a generative model produces multiple candidate molecular graphs
  per spectrum and you need deterministic top-k predictions with accuracy, MCES-distance,
  and Tanimoto-similarity metrics on benchmarks like NPLIB1 or MassSpecGym.
when_to_use_negative:
- Evaluating autoregressive/SMILES baselines such as MSNovelist, where ranking is
  by beam-search log-likelihood rather than sampled-structure frequency
- When you need full candidate diversity rather than the most frequent structures
  — frequency ranking collapses 100 samples to top-k
edam_operation: http://edamontology.org/operation_1812
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3473
tools:
- name: DiffMS
  role: Generative model whose sampled molecules per spectrum are filtered and frequency-ranked
  repo: https://github.com/coleygroup/DiffMS
- name: RDKit
  role: Validity/connectedness filtering of sampled structures and Tanimoto similarity
    computation (rdkit=2024.09.4 in the diffms environment)
  repo: https://github.com/coleygroup/DiffMS
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:24168e3cb2fd73e098205b8cfb5382883cf96ba2c657fda20006ae85c0ea39eb
  source_papers:
  - doi: 10.48550/arxiv.2502.09571
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coleygroup/DiffMS
  tools:
  - DiffMS
  - MIST
  - PyTorch
  - RDKit
  - SIRIUS
  edam_operation: http://edamontology.org/operation_1812
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9047619047619048
n_claims: 21
---

# dataset-frequency-ranking-postprocessing

## Summary

Post-generation evaluation procedure for de novo molecule generation from mass spectra: sample many candidate molecules per spectrum (100 per spectrum in DiffMS), filter invalid/disconnected structures, and rank by frequency to produce top-k predictions scored against ground truth.

## When to use

Use when a generative model produces multiple candidate molecular graphs per spectrum and you need deterministic top-k predictions with accuracy, MCES-distance, and Tanimoto-similarity metrics on benchmarks like NPLIB1 or MassSpecGym.

## When NOT to use

- Evaluating autoregressive/SMILES baselines such as MSNovelist, where ranking is by beam-search log-likelihood rather than sampled-structure frequency
- When you need full candidate diversity rather than the most frequent structures — frequency ranking collapses 100 samples to top-k

## Inputs

- 100 sampled molecules per spectrum from the diffusion decoder (each as an RDKit-parseable molecular graph/SMILES)
- Ground-truth molecules for the benchmark dataset (NPLIB1/CANOPUS or MassSpecGym spectra-molecule pairs)

## Outputs

- Top-1 and Top-10 accuracy per dataset
- Top-1 and Top-10 minimum MCES distance
- Top-1 and Top-10 maximum Tanimoto similarity
- Per-spectrum top-k molecule lists (e.g., prior_ablation_metrics.csv for the ablation task)

## How to apply

For each spectrum, sample 100 molecules from the generative model (DiffMS's graph diffusion decoder generates molecular graphs as atom-feature and adjacency tensors). Remove molecules that RDKit cannot parse (invalid) or that are disconnected, then count exact structures and identify the top-k molecules by frequency to serve as Top-1 and Top-10 predictions. Evaluate these against the ground-truth molecule using Top-1/Top-10 accuracy, Top-1/Top-10 minimum MCES distance, and Top-1/Top-10 maximum Tanimoto similarity. This frequency-based ranking is the DiffMS counterpart to beam-search-with-log-likelihood ranking used for autoregressive baselines like MSNovelist.

## Related tools

- **DiffMS** (Generative model whose sampled molecules per spectrum are filtered and frequency-ranked) — https://github.com/coleygroup/DiffMS
- **RDKit** (Validity/connectedness filtering of sampled structures and Tanimoto similarity computation (rdkit=2024.09.4 in the diffms environment)) — https://github.com/coleygroup/DiffMS

## Examples

```
python src/spec2mol_main.py dataset=canopus general.load_weights=<checkpoint>; then sample 100 molecules/spectrum, drop invalid/disconnected via RDKit, rank top-k by frequency, and write prior_ablation_metrics.csv
```

## Evaluation signals

- Every molecule counted in the ranking passes RDKit sanitization and is a single connected component
- Top-k lists contain at most k distinct structures, ordered by descending sample frequency
- Ablation comparison reproduces the paper's ordering: the dataset-marginal edge-type prior yields the best accuracy and lowest MCES versus empty and fully-connected single-bond priors
- Metrics match Table 6-style structure: {Top-1/Top-10 accuracy, Top-1/Top-10 min MCES, Top-1/Top-10 max Tanimoto}

## Limitations

- Frequency ranking collapses 100 sampled structures to a few top-k predictions; structurally close but non-identical matches are captured by MCES/Tanimoto, not exact accuracy
- De novo generation of the exact molecular structure remains challenging; results depend on the removal of invalid or disconnected samples, which varies with model quality

## Evidence

- [methods] sample 100 molecules per spectrum, filter and rank by frequency: [evidence span withheld — anchor status: not_found]
- [other] evaluation metrics against ground truth: [evidence span withheld — anchor status: not_found]
- [methods] baseline ranking differs for MSNovelist: "beam search (with a width of 100) and subsequently computed log-likelihoods are used for ranking"
- [readme] RDKit environment for validity checks: "conda create -y -c conda-forge -n diffms rdkit=2024.09.4 python=3.9"
