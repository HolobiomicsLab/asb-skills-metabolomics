---
name: tandem-ms-spectrum-prediction
description: Use when reference MS/MS spectra are unavailable for candidate compounds
  in non-targeted metabolomics, or when validating putative compound annotations by
  comparing predicted vs. measured spectra for [M+H]+ / [M-H]- precursors.
when_to_use_negative:
- Multi-stage (MSn) fragmentation is required — FIORA's single-step fragmentation
  is its biggest limitation and multi-step fragmentation is not implemented
- Predicting retention time or CCS on new instruments — RT/CCS prediction is prototype-stage
  and hyperparameters were not tuned for these tasks
- The benchmark requires strictly train/test-separated baselines for CFM-ID — CFM-ID
  was trained on METLIN17, so compound separation is not guaranteed, and retraining
  it on the FIORA dataset was computationally infeasible
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: FIORA
  role: 'Primary tool: graph neural network predicting MS/MS spectra, RT, and CCS
    via bond-cleavage-level modeling; provides fiora-predict, fiora-train, and fiora-eval
    CLIs'
  repo: https://github.com/BAMeScience/fiora
- name: ICEBERG
  role: Baseline fragmentation algorithm used to benchmark FIORA's prediction quality
  repo: https://github.com/samgoldman97/ms-pred
- name: CFM-ID
  role: Baseline ML fragmentation predictor run at the closest matching collision
    energy (10/20/40 eV) via its Docker Hub container for comparison
  repo: https://hub.docker.com/r/wishartlab/cfmid
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:9b26687ccb6c05896f366edd58ff3b64cd05a45517f7185c48e16c7636bbc7c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - CFM-ID
  - FIORA
  - ICEBERG
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# tandem-ms-spectrum-prediction

## Summary

Predict tandem mass spectra (MS/MS) in silico with FIORA, a graph neural network that models bond cleavages via the local molecular neighborhood of each bond, and benchmark prediction quality (spectral_sqrt_cosine similarity) against state-of-the-art methods such as ICEBERG and CFM-ID.

## When to use

Use when reference MS/MS spectra are unavailable for candidate compounds in non-targeted metabolomics, or when validating putative compound annotations by comparing predicted vs. measured spectra for [M+H]+ / [M-H]- precursors.

## When NOT to use

- Multi-stage (MSn) fragmentation is required — FIORA's single-step fragmentation is its biggest limitation and multi-step fragmentation is not implemented
- Predicting retention time or CCS on new instruments — RT/CCS prediction is prototype-stage and hyperparameters were not tuned for these tasks
- The benchmark requires strictly train/test-separated baselines for CFM-ID — CFM-ID was trained on METLIN17, so compound separation is not guaranteed, and retraining it on the FIORA dataset was computationally infeasible

## Inputs

- CSV query file with columns Name, SMILES, Precursor_type, CE, Instrument_type
- Trained FIORA checkpoint (e.g., checkpoints/fiora.pt) or default packaged model under fiora/resources/models
- Spectral library split CSV (NIST'17 + MS-DIAL merged) for training/validation
- MGF experimental spectra for evaluation

## Outputs

- Predicted spectra in mgf or msp format
- validation_eval.csv / test_eval.csv with per-compound spectral_sqrt_cosine scores
- Median cosine similarity summaries per superclass or architecture/depth configuration
- Optional training history (./.csv) via --history-out

## How to apply

Assemble an input csv with header columns Name, SMILES, Precursor_type, CE, Instrument_type and run fiora-predict with the default packaged model under fiora/resources/models to emit predicted spectra in mgf or msp format; a trained checkpoint can be supplied via fiora-train (200 epochs, ADAM optimizer, weighted MSE loss, plateau-reducing LR scheduler on cuda:0, RGCN with depth 6 being the selected architecture). Evaluate predictions with fiora-eval (-m checkpoint --device cuda:0 --output-dir), which reports per-compound spectral_sqrt_cosine similarity and writes validation_eval.csv / test_eval.csv. Aggregate quality by median cosine similarity — FIORA reaches medians well above 0.7 for most compound superclasses, with organohalogen (0.63) and hydrocarbon derivatives (0.66) as weak spots. For CASMI-style challenges with stepped collision energies, predict at all 3 CEs and merge into a single 'stepped' spectrum before comparison.

## Related tools

- **FIORA** (Primary tool: graph neural network predicting MS/MS spectra, RT, and CCS via bond-cleavage-level modeling; provides fiora-predict, fiora-train, and fiora-eval CLIs) — https://github.com/BAMeScience/fiora
- **ICEBERG** (Baseline fragmentation algorithm used to benchmark FIORA's prediction quality) — https://github.com/samgoldman97/ms-pred
- **CFM-ID** (Baseline ML fragmentation predictor run at the closest matching collision energy (10/20/40 eV) via its Docker Hub container for comparison) — https://hub.docker.com/r/wishartlab/cfmid

## Examples

```
fiora-predict -i examples/example_input.csv -o examples/example_spec.mgf
```

## Evaluation signals

- Per-compound spectral_sqrt_cosine similarity values are written to validation_eval.csv / test_eval.csv and median cosine similarity exceeds ~0.7 for most compound superclasses (organohalogens ~0.63, hydrocarbon derivatives ~0.66)
- Architecture/depth sweep confirms the 6-layer RGCN achieves the highest median spectral_sqrt_cosine on the validation split, with convolutional architectures (GCN, RGCN) outperforming GAT and Graph Transformer
- Default-model predictions match the expected output (examples/expected_output.mgf), verified automatically by running pytest
- Test-set compounds are removed from training data (80/10/10 split of molecular structures) to preserve benchmark integrity
- For CASMI-style stepped-CE data, the merged 'stepped' spectrum built from all 3 predicted collision energies is used instead of a single-CE prediction

## Limitations

- Single-step fragmentation is the biggest limitation of FIORA; multi-step fragmentation is not implemented
- FIORA was trained on a relatively small dataset of ~10,000 unique compounds, limited by lack of experimental metadata, especially collision energies
- Homogeneity of NIST'17 data may limit generalization to more diverse experimental setups
- RT and CCS prediction is prototype-stage with untuned hyperparameters
- NIST'17 is commercial and does not provide RT/CCS values, restricting public reuse

## Evidence

- [abstract] Graph neural network models bond cleavages via local molecular neighborhood: "utilizing the molecular neighborhood of bonds to learn breaking patterns and derive fragment ion probabilities"
- [abstract] FIORA surpasses ICEBERG and CFM-ID in prediction quality: "FIORA not only surpasses state-of-the-art fragmentation algorithms, ICEBERG and CFM-ID, in prediction quality"
- [discussion] Training protocol: 200 epochs, ADAM, weighted MSE loss, plateau-reducing scheduler: "The training process consisted of 200 epochs using the ADAM optimizer, a weighted MSE loss and a scheduler that reduces the learning rate upon reaching a plateau in validation loss."
- [readme] Input format with required header columns: "Input files are expected to be in csv format. With a header defining the columns: "Name", "SMILES", "Precursor_type", "CE", "Instrument_type""
- [readme] Default model is packaged and auto-selected by the CLI: "Default models are packaged under `fiora/resources/models`. The CLI uses these automatically when `--model default` is selected."
