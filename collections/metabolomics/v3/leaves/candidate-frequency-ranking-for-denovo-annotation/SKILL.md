---
name: candidate-frequency-ranking-for-denovo-annotation
description: Use when a diffusion-based or other generative structure-prediction model
  outputs many candidate molecular graphs per mass spectrum and you must distill them
  into a frequency-ranked top-1/top-10 prediction and score it against known ground-truth
  molecules.
when_to_use_negative:
- When evaluating a beam-search baseline such as MSNovelist, which ranks candidates
  by computed log-likelihoods rather than sample frequency
- When the task is chemical formula inference (e.g., SIRIUS, BUDDY, MIST-CF) rather
  than ranking full molecular structures
- When the model emits a single deterministic structure prediction instead of a pool
  of sampled candidates
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: DiffMS
  role: Graph diffusion decoder that samples 100 molecules per spectrum, conditioned
    on the spectrum and formula-derived atom types
  repo: https://github.com/coleygroup/DiffMS
- name: RDKit
  role: Validity and connectivity checks on sampled molecules; Morgan fingerprint
    computation and Tanimoto similarity scoring
- name: MIST
  role: Formula transformer / spectrum encoder providing the fingerprint-based conditioning
    used by the decoder during candidate generation
  repo: https://github.com/coleygroup/MIST
- name: DiGress
  role: Discrete denoising diffusion framework from which the DiffMS decoder codebase
    is adapted
  repo: https://github.com/cvignac/DiGress
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:6b674581a63c5d73d0e3ee79663cce05e38b3d464cd29469b232a243d2f4b65c
  source_papers:
  - doi: 10.48550/arxiv.2502.09571
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coleygroup/DiffMS
  tools:
  - DiGress
  - DiffMS
  - MIST
  - PyTorch
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6956521739130435
n_claims: 23
---

# Candidate-frequency ranking for de novo molecular annotation

## Summary

Converts a generative model's stochastic samples into a ranked de novo molecular annotation by sampling 100 molecules per spectrum with DiffMS's graph diffusion decoder, removing invalid or disconnected molecules, and selecting top-k (k=1, 10) candidates by frequency, then scoring them against ground truth with MassSpecGym de novo generation metrics (top-k accuracy, maximum Tanimoto similarity of RDKit Morgan fingerprints, minimum MCES distance).

## When to use

Use when a diffusion-based or other generative structure-prediction model outputs many candidate molecular graphs per mass spectrum and you must distill them into a frequency-ranked top-1/top-10 prediction and score it against known ground-truth molecules.

## When NOT to use

- When evaluating a beam-search baseline such as MSNovelist, which ranks candidates by computed log-likelihoods rather than sample frequency
- When the task is chemical formula inference (e.g., SIRIUS, BUDDY, MIST-CF) rather than ranking full molecular structures
- When the model emits a single deterministic structure prediction instead of a pool of sampled candidates

## Inputs

- Tandem mass spectra from NPLIB1 or MassSpecGym test sets (processed with the data_processing/00-03 scripts)
- Pretrained DiffMS end-to-end checkpoint (zenodo.org/records/23542112) or separately loaded MIST encoder checkpoint (zenodo.org/records/23303289)
- Chemical formula per spectrum (heavy-atom composition constraining the diffusion decoder's node types)
- Ground-truth molecular structures for the test spectra

## Outputs

- Ranked top-1 and top-10 candidate molecule lists per test spectrum
- Top-k accuracy, top-k maximum Tanimoto similarity (Morgan fingerprints), and top-k minimum MCES distance per dataset
- reproduction_results.csv comparing reproduced metrics to reported Table 1 values

## How to apply

For each test spectrum (NPLIB1 or MassSpecGym), run the DiffMS graph diffusion decoder to sample exactly 100 molecules conditioned on the spectrum and the formula-derived heavy-atom types, loading pretrained end-to-end weights from zenodo.org/records/23542112 via general.load_weights in configs/general/general_default.yaml. Filter the sample pool by removing molecules that RDKit flags as invalid or disconnected. Rank the remaining valid, connected candidates by occurrence frequency to form top-k lists (k=1 and k=10). Score the ranked lists against the ground-truth molecule with top-k accuracy, top-k maximum Tanimoto similarity computed on RDKit Morgan fingerprints, and top-k minimum MCES distance, following MassSpecGym de novo generation metric definitions. Compare the resulting values to the paper's Table 1 DiffMS rows (NPLIB1 top-1 accuracy 8.34%, Tanimoto 0.35, MCES 11.95; MassSpecGym top-1 accuracy 2.30%, Tanimoto 0.28) within stochastic-sampling tolerance.

## Related tools

- **DiffMS** (Graph diffusion decoder that samples 100 molecules per spectrum, conditioned on the spectrum and formula-derived atom types) — https://github.com/coleygroup/DiffMS
- **RDKit** (Validity and connectivity checks on sampled molecules; Morgan fingerprint computation and Tanimoto similarity scoring)
- **MIST** (Formula transformer / spectrum encoder providing the fingerprint-based conditioning used by the decoder during candidate generation) — https://github.com/coleygroup/MIST
- **DiGress** (Discrete denoising diffusion framework from which the DiffMS decoder codebase is adapted) — https://github.com/cvignac/DiGress

## Examples

```
python src/spec2mol_main.py  # with dataset set to 'msg' (MassSpecGym) or 'canopus' (NPLIB1) in configs/config.yaml
```

## Evaluation signals

- Reproduced NPLIB1 top-1 accuracy matches the reported 8.34% (Tanimoto 0.35, MCES 11.95) within stochastic-sampling tolerance
- Reproduced MassSpecGym top-1 accuracy matches the reported 2.30% (Tanimoto 0.28) within tolerance
- Every molecule in the ranked lists passes RDKit validity and connectivity (single connected component) checks before frequency counting
- Both k=1 and k=10 metrics are reported from the same ranked candidate lists
- Compiled values are checked against the reference Table 1 DiffMS rows with both file existence and metric-value checks

## Limitations

- The evaluation is sampling-based, so reproduced metrics vary run-to-run and should only be compared to Table 1 within stochastic tolerance
- De novo generation of the exact molecular structure remains challenging; structurally close matches (high Tanimoto, low MCES) may occur even when top-1 exact accuracy is low
- Frequency-based ranking assumes the diffusion model concentrates samples near the correct structure; low-accuracy regimes (e.g., ~2-8% top-1) mean most spectra have no exact top-1 hit

## Evidence

- [methods] Sampling 100 molecules per spectrum and ranking by frequency: [evidence span withheld — anchor status: not_found]
- [methods] RDKit scoring of ranked candidates: [evidence span withheld — anchor status: not_found]
- [readme] Running the finetuned end-to-end model to generate candidates: "To finetune the end-to-end model on spectra-molecule generation, run spec2mol_main.py"
