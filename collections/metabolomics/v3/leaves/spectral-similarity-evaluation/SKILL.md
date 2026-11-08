---
name: spectral-similarity-evaluation
description: Use when you have one or more trained FIORA checkpoints and must score
  predicted vs. validation spectra with spectral_sqrt_cosine to compare architectures
  or depths before benchmarking.
when_to_use_negative:
- Input is a final benchmark evaluation on CASMI 2016/2022 or held-out test spectra
  rather than a validation-split model-selection comparison
- You only need forward MS/MS prediction (fiora-predict with the default model) and
  have no checkpoints to compare
- The prediction task is RT or CCS estimation, whose hyperparameters were not tuned
  and remain prototype-stage in FIORA
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: FIORA
  role: Trained per architecture/depth via fiora-train and scored on the validation
    split via fiora-eval using the spectral_sqrt_cosine metric
  repo: https://github.com/BAMeScience/fiora
- name: ICEBERG
  role: State-of-the-art fragmentation baseline against which the selected FIORA model's
    prediction quality is ultimately compared (not part of the model-selection loop)
  repo: https://github.com/samgoldman97/ms-pred
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:74de068df56da238ad5d7c2b084a1a116d972eb08beec495163eaca5a556f741
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - FIORA
  - ICEBERG
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# spectral-similarity-evaluation

## Summary

Evaluate trained FIORA spectral prediction checkpoints on a held-out validation split using the spectral_sqrt_cosine metric, aggregating median cosine similarity to drive model selection across GNN architectures (GCN, RGCN, GAT, Graph Transformer) and graph depths (0–7).

## When to use

Use when you have one or more trained FIORA checkpoints and must score predicted vs. validation spectra with spectral_sqrt_cosine to compare architectures or depths before benchmarking.

## When NOT to use

- Input is a final benchmark evaluation on CASMI 2016/2022 or held-out test spectra rather than a validation-split model-selection comparison
- You only need forward MS/MS prediction (fiora-predict with the default model) and have no checkpoints to compare
- The prediction task is RT or CCS estimation, whose hyperparameters were not tuned and remain prototype-stage in FIORA

## Inputs

- Trained FIORA checkpoints (e.g., checkpoints/fiora.pt per architecture/depth configuration)
- Validation-split spectra library csv with columns Name, SMILES, Precursor_type, CE, Instrument_type
- Training history files (--history-out . or .csv) from fiora-train

## Outputs

- validation_eval.csv per evaluated checkpoint with spectral_sqrt_cosine scores
- Median spectral_sqrt_cosine per architecture/depth with 95% confidence intervals (bootstrapped over compounds)
- Model-selection curves (median cosine similarity vs. graph depth per architecture) and selected best checkpoint (6-layer RGCN)

## How to apply

Assemble the merged NIST'17+MS-DIAL training split as a csv with header columns Name, SMILES, Precursor_type, CE, and Instrument_type, keeping FIORA's 80/10/10 train/validation/test split over molecular structures (test compounds removed from training). For each candidate configuration (GCN, RGCN, GAT, Graph Transformer at depths 0–7), run fiora-train for 200 epochs with the ADAM optimizer, a weighted MSE loss, and a plateau-reducing learning-rate scheduler on cuda:0, persisting per-epoch metrics via --history-out. Then evaluate each checkpoint with fiora-eval (--device cuda:0), which reports spectral_sqrt_cosine summary scores and writes validation_eval.csv per run. Aggregate the median spectral_sqrt_cosine over validation spectra per configuration and compute 95% confidence intervals (e.g., by bootstrapping compounds) to select the best model; in the reference study the 6-layer RGCN maximized median validation cosine similarity.

## Related tools

- **FIORA** (Trained per architecture/depth via fiora-train and scored on the validation split via fiora-eval using the spectral_sqrt_cosine metric) — https://github.com/BAMeScience/fiora
- **ICEBERG** (State-of-the-art fragmentation baseline against which the selected FIORA model's prediction quality is ultimately compared (not part of the model-selection loop)) — https://github.com/samgoldman97/ms-pred

## Examples

```
fiora-eval -m checkpoints/fiora.pt --device cuda:0 --output-dir eval/rgcn_depth6 && fiora-train -i library.csv -o checkpoints/fiora.pt --device cuda:0 --history-out checkpoints/fiora_history.
```

## Evaluation signals

- validation_eval.csv is written for every evaluated checkpoint and contains spectral_sqrt_cosine summary scores
- Median spectral_sqrt_cosine on the validation split for the 6-layer RGCN is the highest among all 32 architecture/depth combinations (4 architectures × depths 0–7)
- Convolutional architectures (GCN, RGCN) show higher median cosine similarity than attention-based ones (GAT, Graph Transformer), matching the reported selection curves
- Per-epoch history files (--history-out) confirm 200 epochs with decreasing validation loss and plateau-reduced learning rate
- 95% bootstrap confidence intervals over compounds do not overlap between the selected model and the runner-up (or differences are qualitatively consistent across resamples)

## Limitations

- NIST'17 is a commercial library requiring a license, limiting full public reproduction of the merged training split
- The homogeneity of NIST'17 data may limit generalization of the selected model to more diverse experimental setups
- CFM-ID comparison baselines were trained on METLIN17, so train/test compound separation is not guaranteed for those comparisons

## Evidence

- [discussion] 200-epoch training protocol with weighted MSE loss and LR scheduler: "The training process consisted of 200 epochs using the ADAM optimizer, a weighted MSE loss and a scheduler that reduces the learning rate upon reaching a plateau in validation loss."
- [discussion] 80/20 molecular-structure split underlying the validation set: "FIORA was trained on 80% of the molecular structures, with the remaining 20% split evenly between validation and testing."
- [readme] history persistence flag in fiora-train CLI: "To persist per-epoch training history, add `--history-out` (supports `.` or `.csv`):"
- [discussion] test-compound removal preserving evaluation integrity: "To preserve the integrity of the test results, all compounds from the test sets were removed from the training data for FIORA and ICEBERG."
