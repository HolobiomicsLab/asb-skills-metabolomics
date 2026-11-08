---
name: beam-search-decoding-hyperparameter-tuning
description: Use when evaluating a finetuned/aligned seq2seq molecular generator whose
  output is SELFIES/SMILES and you must sweep decoding settings (beam width, temperature,
  number of samples) before reporting benchmark metrics.
when_to_use_negative:
- The model is diffusion-based (DiffMS, FRIGID) and generates via iterative denoising
  rather than an autoregressive beam decoder — use its own sampler and re-ranking
  protocol instead
- You are tuning the MIST fingerprint encoder or the pretraining/alignment objectives
  rather than the decoding-time sampler of a frozen checkpoint
- Only a single pre-published decoding configuration is required and no benchmark
  re-ranking or hyperparameter sweep is being reproduced
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MS-BART
  role: BART-Base autoregressive model whose beam-search multinomial decoding (beam
    width up to 100, temperature sweep) is being tuned and evaluated
  repo: github.com/OpenDFM/MS-BART
- name: RDKit
  role: Computes Morgan fingerprints (radius 2, 2048 bits) used to score decoded molecules
    by Tanimoto similarity for metric reporting and re-ranking
- name: MIST
  role: Supplies the predicted 4096-bit fingerprints that condition decoding quality;
    its accuracy bounds the achievable decoding performance
  repo: https://github.com/coleygroup/MIST
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:12e8648504370a0ad172041ab7e98538bafdd6334deb8df1cc8e5b9e060882b3
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BART-Base
  - MIST
  - MS-BART
  - MassSpecGym
  - NPLIB1
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# Beam-search decoding hyperparameter tuning

## Summary

Tunes the autoregressive decoding of MS-BART (a BART-Base seq2seq molecular structure elucidator) over beam widths and sampling temperature so that Top-1/Top-10 InChIKey accuracy, MCES distance, and Tanimoto similarity are maximized on MassSpecGym and NPLIB1 test folds.

## When to use

Use when evaluating a finetuned/aligned seq2seq molecular generator whose output is SELFIES/SMILES and you must sweep decoding settings (beam width, temperature, number of samples) before reporting benchmark metrics.

## When NOT to use

- The model is diffusion-based (DiffMS, FRIGID) and generates via iterative denoising rather than an autoregressive beam decoder — use its own sampler and re-ranking protocol instead
- You are tuning the MIST fingerprint encoder or the pretraining/alignment objectives rather than the decoding-time sampler of a frozen checkpoint
- Only a single pre-published decoding configuration is required and no benchmark re-ranking or hyperparameter sweep is being reproduced

## Inputs

- Aligned/finetuned MS-BART (BART-Base) model weights, e.g. data/MassSpecGym/model-weights
- MassSpecGym and NPLIB1 test-fold spectra with chemical formula and metadata
- MassSpecGym evaluation script (requires test-path filename fix and NUM_BEAMS/NUM_BEAM fix)

## Outputs

- Top-1/Top-10 accuracy by full InChIKey exact match per decoding configuration
- Top-1/Top-10 minimum MCES distance and maximum Tanimoto similarity (Morgan radius 2, 2048 bits)
- Temperature-ablation table (Appendix D style) and a Table-1-style results_table.csv against baselines (Spec2Mol, MIST+Neuraldecipher, MIST+MSNovelist, MADGEN, DiffMS)

## How to apply

Decode the aligned MS-BART checkpoint on the complete MassSpecGym and NPLIB1 test folds using beam-search multinomial sampling, sweeping beam widths up to 100 and running a temperature ablation (paper Appendix D) to select the operating point. Before running, patch the known MassSpecGym eval-script bugs: the incorrect test-path filename and the NUM_BEAMS/NUM_BEAM variable mismatch, otherwise the sweep silently uses wrong beams or wrong data. Score each setting with Top-1/Top-10 accuracy by full InChIKey exact match, minimum MCES distance, and maximum Tanimoto similarity computed from RDKit Morgan fingerprints (radius 2, 2048 bits). For a like-for-like comparison against DiffMS, switch from beam search to sampling 100 molecules per spectrum and re-rank candidates by formula distance (Eq. 6) with log-probability tie-breaking. Validate the chosen configuration by checking reproduced metrics against reported values (e.g., NPLIB1 Top-1 7.45%, MCES 9.66, Tanimoto 0.44; MassSpecGym Top-1 1.07%).

## Related tools

- **MS-BART** (BART-Base autoregressive model whose beam-search multinomial decoding (beam width up to 100, temperature sweep) is being tuned and evaluated) — github.com/OpenDFM/MS-BART
- **RDKit** (Computes Morgan fingerprints (radius 2, 2048 bits) used to score decoded molecules by Tanimoto similarity for metric reporting and re-ranking)
- **MIST** (Supplies the predicted 4096-bit fingerprints that condition decoding quality; its accuracy bounds the achievable decoding performance) — https://github.com/coleygroup/MIST

## Evaluation signals

- Beam-width sweep completes with widths up to 100 and a temperature ablation is reported (mirroring Appendix D)
- MassSpecGym eval script bugs are fixed first: test-path filename corrected and NUM_BEAMS/NUM_BEAM variable mismatch resolved
- Reproduced metrics land near reported values: NPLIB1 Top-1 accuracy 7.45%, MCES 9.66, Tanimoto 0.44; MassSpecGym Top-1 accuracy 1.07%, MCES 16.47, Tanimoto 0.28
- For the DiffMS comparison exactly 100 molecules per spectrum are sampled and re-ranked by formula distance (Eq. 6) with log-probability tie-breaking
- Final numbers compiled for all baselines (Spec2Mol, MIST+Neuraldecipher, MIST+MSNovelist, MADGEN, DiffMS) match the 12-key-metric Table-1 layout saved as results_table.csv

## Limitations

- Formulas are used only for re-ranking, not in model training, so decoding cannot exploit formula constraints directly (deferred to future work in the paper)
- Decoding quality is capped by MIST fingerprint-prediction accuracy; inaccurate predicted fingerprints propagate into worse decoded structures
- Earlier MIST MassSpecGym encoders used test-set chemical-formula information in training; only revised (e.g., 4M-aug) encoders give a leakage-free comparison, so beam-search results are encoder-sensitive

## Evidence

- [supplementary] Evaluate the aligned model on the complete MassSpecGym and NPLIB1 test folds using beam-search multinomial sampling: [evidence span withheld — anchor status: not_found]
- [methods] sample 100 molecules per spectrum and re-rank by formula distance with tie-breaking: [evidence span withheld — anchor status: not_found]
- [other] correcting the known MSG eval script issues: [evidence span withheld — anchor status: not_found]
- [discussion] The accuracy of these fingerprint predictions significantly impacts the overall performance.: "The accuracy of these fingerprint predictions significantly impacts the overall performance."
