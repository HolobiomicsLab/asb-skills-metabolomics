---
name: sampling-temperature-sensitivity-analysis
description: Use when you need to test whether a generative molecular-structure decoder's
  reported benchmark metrics depend on the sampling temperature hyperparameter, e.g.
  reproducing Table 8 of the MS-BART paper.
when_to_use_negative:
- Evaluating a deterministic (greedy/beam-only) decoding configuration where no sampling
  temperature is involved — the ablation has no varying knob.
- Running on the restricted NIST23 dataset, which the paper excludes due to commercial
  licensing.
- Assessing sensitivity to a non-decoding hyperparameter (e.g. fingerprint bit threshold
  epsilon or adduct filtering), which needs a different ablation protocol.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3418
- http://edamontology.org/topic_0091
tools:
- name: MS-Bart
  role: Generative decoder whose sampled SELFIES outputs are evaluated across temperatures
  repo: github.com/OpenDFM/MS-BART
- name: MIST
  role: Produces the 4096-bit fingerprint predictions from spectra that serve as decoder
    input
  repo: github.com/coleygroup/FRIGID
- name: RDKit
  role: Computes Morgan fingerprints (radius 2, 2048 bits) for Tanimoto similarity
    from decoded structures
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:23ed7eb3abc74f79f69d6ecee2670ae57c9a833126096a0d2cfd8e2e12eee70a
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MIST
  - MS-Bart
  - RDKit
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3418
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# sampling-temperature-sensitivity-analysis

## Summary

A robustness-ablation skill that decodes a generative structure-elucidation model (e.g. MS-Bart on MassSpecGym) at several sampling temperatures and checks whether Top-1/Top-10 accuracy, MCES distance and Tanimoto similarity metrics remain stable across temperatures.

## When to use

Use when you need to test whether a generative molecular-structure decoder's reported benchmark metrics depend on the sampling temperature hyperparameter, e.g. reproducing Table 8 of the MS-BART paper.

## When NOT to use

- Evaluating a deterministic (greedy/beam-only) decoding configuration where no sampling temperature is involved — the ablation has no varying knob.
- Running on the restricted NIST23 dataset, which the paper excludes due to commercial licensing.
- Assessing sensitivity to a non-decoding hyperparameter (e.g. fingerprint bit threshold epsilon or adduct filtering), which needs a different ablation protocol.

## Inputs

- Final MS-BART model weights trained on MassSpecGym (model-weights from Figshare release)
- MassSpecGym test split with MIST-predicted 4096-bit Morgan fingerprints, thresholded at epsilon=0.11, paired with SELFIES targets
- Molecular formulae associated with each test spectrum (used for candidate re-ranking)

## Outputs

- table8_temperature_sensitivity.csv with one row per sampling temperature (0.2, 0.4, 0.8) and columns Top-1/Top-10 accuracy, Top-k MCES, Top-k Tanimoto
- Per-spectrum top-100 candidate SELFIES lists and their re-ranked orderings

## How to apply

Load the final MS-Bart checkpoint trained on MassSpecGym together with the tokenized MIST-predicted fingerprint inputs from the test split of the fingerprint-SELFIES table at bit threshold epsilon=0.11. For each sampling temperature in {0.2, 0.4, 0.8}, generate 100 candidate SELFIES molecules per spectrum using beam-search multinomial sampling with the fully optimized (pretrain-FT-rank) model. Re-rank the 100 candidates per spectrum by formula distance (sum of absolute atom-count differences), breaking ties by estimated log-probability, then select top-k predictions. Convert SELFIES to structures and compute Top-1/Top-10 exact-match accuracy via full InChIKey comparison, Top-k maximum Tanimoto similarity with Morgan fingerprints (radius 2, 2048 bits, RDKit), and Top-k minimum MCES graph distance. Assemble the per-temperature metric rows into a table (e.g. table8_temperature_sensitivity.csv) and verify the metrics are essentially constant across temperatures.

## Related tools

- **MS-Bart** (Generative decoder whose sampled SELFIES outputs are evaluated across temperatures) — github.com/OpenDFM/MS-BART
- **MIST** (Produces the 4096-bit fingerprint predictions from spectra that serve as decoder input) — github.com/coleygroup/FRIGID
- **RDKit** (Computes Morgan fingerprints (radius 2, 2048 bits) for Tanimoto similarity from decoded structures)

## Examples

```
for T in 0.2 0.4 0.8; do python generate.py --model data/MassSpecGym/model-weights --split test --eps 0.11 --temperature $T --num_candidates 100; done
```

## Evaluation signals

- Top-1/Top-10 accuracy, MCES and Tanimoto values are essentially constant across the three temperature rows (matching the paper's 'remarkable stability' claim).
- Each temperature condition produces exactly 100 candidate SELFIES per spectrum before re-ranking.
- Exact-match accuracy agrees between InChIKey comparison of decoded structures and the canonical Top-k definition used elsewhere in the paper.
- Metric definitions match Table 8: Top-k maximum Tanimoto and Top-k minimum MCES over the ranked candidates.

## Limitations

- MS-Bart depends on an external MIST model for fingerprint prediction; the accuracy of these predictions limits overall performance and thus any temperature conclusions.
- Formula information is used only for re-ranking and not for model training, so robustness results reflect the decoding stage rather than the full pipeline.
- Beam-search multinomial sampling with 100 candidates is computationally heavier than single-sample decoding.

## Evidence

- [methods] temperature rows of metrics: [evidence span withheld — anchor status: not_found]
- [results] formula re-ranking: [evidence span withheld — anchor status: not_found]
- [supplementary] remarkable stability claim: [evidence span withheld — anchor status: not_found]
- [methods] evaluation metrics: [evidence span withheld — anchor status: not_found]
