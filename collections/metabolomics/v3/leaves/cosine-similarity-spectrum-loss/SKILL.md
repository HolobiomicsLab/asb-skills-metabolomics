---
name: cosine-similarity-spectrum-loss
description: Use when training or benchmarking MS/MS spectrum-prediction models (e.g.,
  ICEBERG, SCARF, MassFormer, NEIMS) whose spectra must be compared to ground truth
  via binned-vector cosine similarity.
when_to_use_negative:
- When spectra are already nearly identical and a simple peak-wise RMSE is desired
  instead of angle-based comparison — cosine emphasizes relative intensity patterns
  over absolute scale, which may not suit calibration studies.
- When spectra are recorded at much finer m/z resolution than 0.1 Da and sub-Da peak
  shifts are the phenomenon of interest — binning to 0.1 Da merges such structure.
- When the goal is interpretability of individual fragment attributions rather than
  aggregate spectrum accuracy — cosine over binned vectors does not guarantee the
  fragment explanations are physically correct (equivalent-mass rearrangements may
  score equally well).
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: ICEBERG
  role: The Score module is trained to maximize a vectorized cosine similarity against
    the observed spectrum; predicted spectra are then evaluated with the same metric.
  repo: https://github.com/samgoldman97/ms-pred
- name: SCARF
  role: Baseline spectrum predictor whose predicted spectra are compared with the
    cosine metric to establish benchmark values (~0.726 on NIST20).
- name: MassFormer
  role: Baseline graph-transformer predictor evaluated with the same binned-cosine
    similarity (~0.568 on NPLIB1).
  repo: https://github.com/Roestlab/massformer
- name: spec_pred_eval
  role: Analysis module (analysis/spec_pred_eval.py) implementing the vectorized cosine
    similarity over 0.1-Da-binned spectra vectors for benchmarking.
  repo: https://github.com/samgoldman97/ms-pred
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c04654
    title: Generating Molecular Fragmentation Graphs with Autoregressive Neural Networks
  source_content_hash: sha256:610420e515b407df48247d57995eb66cb4a08555511612d446356376ecc9623d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samgoldman97/ms-pred
  tools:
  - CFM-ID
  - ICEBERG
  - MassFormer
  - PyTorch
  - Python
  - SCARF
  - spec_pred_eval
  - uv
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# cosine-similarity-spectrum-loss

## Summary

Vectorized cosine similarity over 0.1-Da-binned spectra vectors, used both as ICEBERG's training loss for the fragment-intensity Score module and as the evaluation metric for predicted MS/MS spectra on NIST20 and NPLIB1.

## When to use

Use when training or benchmarking MS/MS spectrum-prediction models (e.g., ICEBERG, SCARF, MassFormer, NEIMS) whose spectra must be compared to ground truth via binned-vector cosine similarity.

## When NOT to use

- When spectra are already nearly identical and a simple peak-wise RMSE is desired instead of angle-based comparison — cosine emphasizes relative intensity patterns over absolute scale, which may not suit calibration studies.
- When spectra are recorded at much finer m/z resolution than 0.1 Da and sub-Da peak shifts are the phenomenon of interest — binning to 0.1 Da merges such structure.
- When the goal is interpretability of individual fragment attributions rather than aggregate spectrum accuracy — cosine over binned vectors does not guarantee the fragment explanations are physically correct (equivalent-mass rearrangements may score equally well).

## Inputs

- Predicted spectra from ICEBERG Score module (fragment + hydrogen-shift attributed peaks, serialized as JSON)
- Ground-truth observed MS/MS spectra from NIST20, NPLIB1, or NIST'23 processed datasets (subformulae-annotated)
- Predicted binned spectra from baseline models (SCARF, MassFormer, NEIMS, FixedVocab, CFM-ID)

## Outputs

- Per-molecule cosine similarity scores between predicted and true binned spectra
- Average cosine similarity per model per dataset (e.g., ICEBERG 0.627 on NPLIB1, 0.727 on NIST20)
- Trained ICEBERG Score checkpoint optimized with the cosine loss
- Benchmark CSV table of per-model, per-dataset cosine similarities

## How to apply

Convert each observed and predicted spectrum into a binned intensity vector at 0.1-Da resolution across the m/z range, so that every fragment peak maps to a bin. For training ICEBERG Score, use a vectorized cosine similarity objective that couples all predicted fragment-hydrogen-shift intensities simultaneously, rather than optimizing per-peak losses. For evaluation, apply the same vectorized cosine similarity between predicted and true binned spectra (implemented in analysis/spec_pred_eval.py), then average across the test split per model per dataset. Use the metric to benchmark against baselines (e.g., SCARF 0.726–0.727 on NIST20, MassFormer/FixedVocab ~0.568 on NPLIB1) and to verify ICEBERG reproduces ~0.627 (NPLIB1) and ~0.727 (NIST20).

## Related tools

- **ICEBERG** (The Score module is trained to maximize a vectorized cosine similarity against the observed spectrum; predicted spectra are then evaluated with the same metric.) — https://github.com/samgoldman97/ms-pred
- **SCARF** (Baseline spectrum predictor whose predicted spectra are compared with the cosine metric to establish benchmark values (~0.726 on NIST20).)
- **MassFormer** (Baseline graph-transformer predictor evaluated with the same binned-cosine similarity (~0.568 on NPLIB1).) — https://github.com/Roestlab/massformer
- **spec_pred_eval** (Analysis module (analysis/spec_pred_eval.py) implementing the vectorized cosine similarity over 0.1-Da-binned spectra vectors for benchmarking.) — https://github.com/samgoldman97/ms-pred

## Examples

```
python analysis/spec_pred_eval.py --predictions iceberg_nist20_preds.json --ground-truth nist20_gt.json --bin 0.1 --metric cosine
```

## Evaluation signals

- ICEBERG reproduces the published average cosine similarity of ~0.627 on NPLIB1 and ~0.727 on NIST20 when predictions are evaluated against ground truth with the 0.1-Da binned cosine.
- Baselines fall within reported ranges, e.g., MassFormer and FixedVocab near 0.568 on NPLIB1 and SCARF near 0.726 on NIST20.
- All spectra vectors share identical binning (0.1 Da) and length, so cosine values lie in [0, 1] and per-molecule scores are directly comparable across models.
- The vectorized loss couples all fragment-hydrogen-shift intensities jointly — inspect that the training objective returns one scalar per molecule, not independent per-peak losses.

## Limitations

- The cosine objective provides no guarantee that the generated fragment explanations are the correct physical explanations, since equivalent-mass atom and bond rearrangements can score identically.
- Roughly 1 CPU second per molecule is needed for ICEBERG inference (~10x SCARF), so large-scale cosine evaluation over huge libraries is compute-heavy.
- The vectorized cosine is computed over 0.1-Da bins, so sub-Da peak position information is lost during training and evaluation.
- NPLIB1 support has been retired in the main ms-pred branch; it is only available on the iceberg_1.0 branch.

## Evidence

- [methods] cosine similarity with 0.1-Da binned vectors: [evidence span withheld — anchor status: not_found]
- [discussion] ICEBERG Score trained to maximize vectorized cosine: "our decision to train ICEBERG Score to maximize a vectorized cosine similarity"
- [results] benchmark values on NPLIB1 and NIST20: [evidence span withheld — anchor status: not_found]
- [discussion] cosine similarity evaluation on both prediction and retrieval settings: "yielding especially strong improvements for natural product molecules under evaluation settings of both spectrum prediction and retrieval"
- [discussion] no guarantee of correct physical fragment explanations: "there are no guarantees that they are the correct physical explanations"
