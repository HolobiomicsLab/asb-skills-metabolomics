---
name: graph-neural-network-training
description: Use when you have a curated spectral library csv (Name, SMILES, Precursor_type,
  CE, Instrument_type) and must train a bond-cleavage GNN and pick architecture/depth
  via validation spectral_sqrt_cosine.
when_to_use_negative:
- You only need inference on candidate compounds — use fiora-predict with the packaged
  default model instead of retraining
- The input library has not passed the paper's quality filters (e.g., inconsistent
  SMILES/MOL/InChiKey entries, missing collision energies, precursor intensity above
  90%)
- You need multi-stage (MSn) fragmentation prediction — FIORA is single-step and this
  skill does not cover multi-step fragmentation
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: FIORA
  role: Graph neural network framework whose fiora-train and fiora-eval CLI scripts
    are used to train architecture/depth candidates and score them on validation spectra
  repo: https://github.com/BAMeScience/fiora
- name: ICEBERG
  role: Benchmark baseline implemented in ms-pred that FIORA-trained models are compared
    against in prediction quality
  repo: https://github.com/samgoldman97/ms-pred
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:d7fc93d72e309d4ef3dec1a217619e1920039b98b94ae3f235a89d1f70c72c11
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
yes_rate: 0.7692307692307693
n_claims: 13
---

# graph-neural-network-training

## Summary

Train FIORA's molecular-graph neural networks on merged spectral-library data (e.g., NIST'17 + MS-DIAL, or MSnLib) to predict tandem MS/MS fragment intensities, RT and CCS, and select the best architecture and depth (convolutional GCN/RGCN vs. attention GAT/Transformer, depths 0–7) by validation cosine similarity.

## When to use

Use when you have a curated spectral library csv (Name, SMILES, Precursor_type, CE, Instrument_type) and must train a bond-cleavage GNN and pick architecture/depth via validation spectral_sqrt_cosine.

## When NOT to use

- You only need inference on candidate compounds — use fiora-predict with the packaged default model instead of retraining
- The input library has not passed the paper's quality filters (e.g., inconsistent SMILES/MOL/InChiKey entries, missing collision energies, precursor intensity above 90%)
- You need multi-stage (MSn) fragmentation prediction — FIORA is single-step and this skill does not cover multi-step fragmentation

## Inputs

- Training-library csv with header columns Name, SMILES, Precursor_type, CE, Instrument_type (merged NIST'17 + MS-DIAL split, or MSnLib library.csv)
- Molecular structures parsed as molecular graphs with bond-level fragmentation labels
- GPU device (e.g., cuda:0)

## Outputs

- Per-configuration model checkpoint (checkpoints/fiora.pt)
- Per-epoch training history (--history-out, . or .csv)
- validation_eval.csv with spectral_sqrt_cosine summary scores per run
- Median validation cosine similarity with 95% confidence intervals per architecture/depth, and the selected model (6-layer RGCN)

## How to apply

Assemble the training library as a csv with the required header columns Name, SMILES, Precursor_type, CE, and Instrument_type, after applying the paper's filters (ionization types [M+H]+ and [M-H]-, max weight 1000 Da, max CE 100 eV, etc.). Split 80% of molecular structures for training, with the remaining 20% split evenly between validation and testing, keeping test compounds out of training. For each candidate (GCN, RGCN, GAT, Graph Transformer; graph depth 0–7) run fiora-train with 200 epochs, the ADAM optimizer, a weighted MSE loss, and a plateau-reducing learning-rate scheduler on cuda:0, persisting per-epoch history with --history-out. Evaluate each checkpoint with fiora-eval on the validation split to obtain spectral_sqrt_cosine summaries, aggregate median cosine similarity with bootstrapped 95% CIs, and select the top configuration (the paper found the 6-layer RGCN best). The rationale is that convolutional message passing over the local bond neighborhood outperformed pooled/attention-based embeddings for fragmentation-event prediction.

## Related tools

- **FIORA** (Graph neural network framework whose fiora-train and fiora-eval CLI scripts are used to train architecture/depth candidates and score them on validation spectra) — https://github.com/BAMeScience/fiora
- **ICEBERG** (Benchmark baseline implemented in ms-pred that FIORA-trained models are compared against in prediction quality) — https://github.com/samgoldman97/ms-pred

## Examples

```
fiora-train -i resources/data/msnlib/library.csv -o checkpoints/fiora.pt --device cuda:0 --instruments HCD --precursor-modes "[M+H]+,[M-H]-,[M]+,[M]-" --history-out checkpoints/fiora_history.
```

## Evaluation signals

- The 6-layer RGCN checkpoint achieves the highest median spectral_sqrt_cosine on the validation split among all architecture/depth combinations, reproducing Fig. 2 selection curves
- Convolutional architectures (GCN, RGCN) outperform attention-based models (GAT, Graph Transformer) at matched depths
- Per-epoch history files exist for every configuration and validation loss plateaus trigger the learning-rate scheduler within 200 epochs
- Bootstrapped 95% confidence intervals (resampling compounds) do not invert the ranking between the top and runner-up configurations
- Test-set compounds are disjoint from the training split (80/10/10 structure split with all test-compound spectra removed from training)

## Limitations

- FIORA was trained on a relatively small dataset of ~10,000 unique compounds, limited by lack of experimental metadata, especially collision energies
- Hyperparameters were not specifically tuned for the RT and CCS prediction tasks, which remain in prototype stage
- NIST'17 is a commercial library requiring a license and its homogeneity may limit generalization to more diverse experimental setups
- Exact model specification is only in the Supplementary Information, not the main text

## Evidence

- [discussion] The training process consisted of 200 epochs using the ADAM optimizer, a weighted MSE loss and a scheduler that reduces the learning rate upon reaching a plateau in validation loss.: "The training process consisted of 200 epochs using the ADAM optimizer, a weighted MSE loss and a scheduler that reduces the learning rate upon reaching a plateau in validation loss."
- [discussion] FIORA was trained on 80% of the molecular structures, with the remaining 20% split evenly between validation and testing.: "FIORA was trained on 80% of the molecular structures, with the remaining 20% split evenly between validation and testing."
- [readme] To (re)train a new FIORA model, use `fiora-train`.: "To (re)train a new FIORA model, use `fiora-train`. For example, to train on MSnLib with the same parameters used for the v1.0 release"
- [readme] To persist per-epoch training history, add `--history-out` (supports `.` or `.csv`): "To persist per-epoch training history, add `--history-out` (supports `.` or `.csv`)"
- [discussion] we filtered for ionization types [M+H]+ and [M-H]- (in case of MSnLib [M]+ and [M]- ions were considered as well): "we filtered for ionization types [M+H]+ and [M-H]- (in case of MSnLib [M]+ and [M]- ions were considered as well)"
