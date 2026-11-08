---
name: architecture-depth-comparison-analysis
description: Use when selecting a GNN architecture or network depth for tandem MS/MS
  spectral prediction, and validation-split performance (median cosine similarity)
  must drive the choice before final benchmarking.
when_to_use_negative:
- When benchmarking FIORA against ICEBERG or CFM-ID on CASMI 2016/2022 test sets —
  that requires final-test evaluation of the already-selected model, not a validation
  model-selection sweep
- When the task is predicting spectra with an existing trained model (fiora-predict)
  rather than choosing architecture/depth
- When CFM-ID comparability is the goal — CFM-ID was trained on METLIN17 and retraining
  with the FIORA dataset was found computationally infeasible, so this sweep's training
  regime does not transfer
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: FIORA
  role: Trained per architecture/depth configuration with fiora-train (200 epochs,
    ADAM, weighted MSE, plateau-reducing LR scheduler) and evaluated on the validation
    split with fiora-eval reporting spectral_sqrt_cosine scores
  repo: https://github.com/BAMeScience/fiora
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:f08a92a02d71ac31decaffac0e53122632cd1a2c9ecf024a25017a6728ef210a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - FIORA
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# architecture-depth-comparison-analysis

## Summary

Systematic model-selection sweep over graph neural network architectures (GCN, RGCN, GAT, Graph Transformer) and graph depths (0-7) for FIORA spectral prediction, scored by median spectral_sqrt_cosine similarity on the validation split of the merged NIST'17 + MS-DIAL library.

## When to use

Use when selecting a GNN architecture or network depth for tandem MS/MS spectral prediction, and validation-split performance (median cosine similarity) must drive the choice before final benchmarking.

## When NOT to use

- When benchmarking FIORA against ICEBERG or CFM-ID on CASMI 2016/2022 test sets — that requires final-test evaluation of the already-selected model, not a validation model-selection sweep
- When the task is predicting spectra with an existing trained model (fiora-predict) rather than choosing architecture/depth
- When CFM-ID comparability is the goal — CFM-ID was trained on METLIN17 and retraining with the FIORA dataset was found computationally infeasible, so this sweep's training regime does not transfer

## Inputs

- Merged NIST'17 + MS-DIAL training split as csv with header columns Name, SMILES, Precursor_type, CE, Instrument_type
- Validation split spectra (20% of molecular structures, disjoint from the 80% training compounds)
- Per-architecture/depth checkpoint files (e.g., checkpoints/fiora.pt)

## Outputs

- Per-epoch training history files (--history-out, .json or .csv) for each architecture/depth configuration
- validation_eval.csv per run with spectral_sqrt_cosine summary scores
- Median spectral_sqrt_cosine per architecture/depth with bootstrapped 95% confidence intervals
- Plot of median cosine similarity versus graph depth for GCN, RGCN, GAT, and Graph Transformer, plus the selected configuration (6-layer RGCN)

## How to apply

Assemble the merged NIST'17 + MS-DIAL training library as a csv with header columns Name, SMILES, Precursor_type, CE, Instrument_type, then train one FIORA checkpoint per architecture (GCN, RGCN, GAT, Graph Transformer) and depth 0-7 using fiora-train on cuda:0 with 200 epochs, the ADAM optimizer, a weighted MSE loss, and a plateau-reducing learning-rate scheduler, persisting per-epoch history via --history-out. Evaluate each checkpoint on the validation split with fiora-eval (--device cuda:0, --output-dir), which reports spectral_sqrt_cosine summary scores and writes validation_eval.csv. Aggregate the median spectral_sqrt_cosine over validation spectra per architecture/depth cell and compute 95% confidence intervals by bootstrapping compounds. Plot median cosine similarity versus graph depth for the four architectures; the decision rule is to select the configuration with the highest median validation cosine — for FIORA this identified convolutional architectures (GCN, RGCN) over attention-based networks (GAT, Transformer), with the 6-layer RGCN selected for subsequent benchmarking.

## Related tools

- **FIORA** (Trained per architecture/depth configuration with fiora-train (200 epochs, ADAM, weighted MSE, plateau-reducing LR scheduler) and evaluated on the validation split with fiora-eval reporting spectral_sqrt_cosine scores) — https://github.com/BAMeScience/fiora

## Examples

```
fiora-train -i nist_msdial_library.csv -o checkpoints/fiora_rgcn6.pt --device cuda:0 --history-out checkpoints/fiora_rgcn6_history.json; fiora-eval -m checkpoints/fiora_rgcn6.pt --device cuda:0 --output-dir eval/rgcn6
```

## Evaluation signals

- Every architecture/depth combination (4 architectures x depths 0-7) has a checkpoint, a history file, and a validation_eval.csv with spectral_sqrt_cosine summaries
- Aggregated medians show convolutional architectures (GCN, RGCN) outperforming attention-based networks (GAT, Transformer) on validation median cosine similarity
- The 6-layer RGCN achieves the highest median spectral_sqrt_cosine on the validation split and is the recorded selection for benchmarking
- 95% confidence intervals computed by bootstrapping compounds are non-overlapping or clearly ordered between the best and runner-up configurations
- Validation compounds are disjoint from training compounds (80/20 molecular-structure split preserved), so scores reflect true generalization

## Limitations

- FIORA was trained on a relatively small dataset of ~10,000 unique compounds, limited by lack of experimental metadata, especially collision energies, which caps the reliability of the selection curves
- Exact model specification is only in the Supplementary Information, not the main text
- Hyperparameters were not specifically tuned for the RT and CCS prediction task, so the sweep scores only spectral prediction quality

## Evidence

- [discussion] Training regime for each sweep configuration: "The training process consisted of 200 epochs using the ADAM optimizer, a weighted MSE loss and a scheduler that reduces the learning rate upon reaching a plateau in validation loss."
- [discussion] Data split underlying the validation evaluation: "FIORA was trained on 80% of the molecular structures, with the remaining 20% split evenly between validation and testing."
- [readme] fiora-train invocation pattern from README: "fiora-train -i resources/data/msnlib/library.csv -o checkpoints/fiora.pt --device cuda:0"
- [readme] History persistence option from README: "To persist per-epoch training history, add `--history-out` (supports `.json` or `.csv`)"
