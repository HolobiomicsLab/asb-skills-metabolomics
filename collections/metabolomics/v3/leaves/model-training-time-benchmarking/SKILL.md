---
name: model-training-time-benchmarking
description: Use when deciding whether to merge multi-collision-energy MS/MS spectra
  before CNN training, by comparing wall-clock epoch times between spectral-representation
  variants.
when_to_use_negative:
- The goal is measuring annotation accuracy or embedding quality (Euclidean distance,
  cosine similarity) rather than training cost — use the spectral-representation ablation
  skill instead.
- Inputs are already merged spectra and the only remaining variant is merged-plus-neutral-loss;
  epoch-time differences then reflect neutral-loss processing, not the merge decision
  itself.
- Training on external benchmarks (CASMI 2016/2022, ARUS) where the original timing
  claim on the paper's test split does not directly apply.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PyTorch
  role: Runs the CNN training whose epoch time is benchmarked under each spectral-representation
    variant
  repo: https://github.com/faizanurv/ChemEmbed
- name: ChemEmbed
  role: Provides the preprocessing pipeline (MSP to DataFrames, spectral merging)
    and CNN training code whose runtime is measured
  repo: https://github.com/faizanurv/ChemEmbed
- name: pandas >= 1.0.0
  role: Holds the preprocessed spectral DataFrames fed to the timed training loop
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.02.07.637102
    title: 'ChemEmbed: A deep learning framework for metabolite identification using
      enhanced MS/MS data and multidimensional molecular embeddings'
  source_content_hash: sha256:68196807e338fc96148d90b94a0930e19e62e00585771413594f6eb18568da14
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/massspecdl/ChemEmbed
  repo_url: https://github.com/faizanurv/ChemEmbed
  tools:
  - ChemEmbed
  - Mol2vec
  - PyTorch
  - RDKit
  - numpy >=
  - pandas >= 1.0.0
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# model-training-time-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark computational cost of CNN training under different spectral-representation schemes by timing one full epoch with merged spectra versus individual collision-energy spectra, quantifying the ~17x speed-up from merged inputs.

## When to use

Use when deciding whether to merge multi-collision-energy MS/MS spectra before CNN training, by comparing wall-clock epoch times between spectral-representation variants.

## When NOT to use

- The goal is measuring annotation accuracy or embedding quality (Euclidean distance, cosine similarity) rather than training cost — use the spectral-representation ablation skill instead.
- Inputs are already merged spectra and the only remaining variant is merged-plus-neutral-loss; epoch-time differences then reflect neutral-loss processing, not the merge decision itself.
- Training on external benchmarks (CASMI 2016/2022, ARUS) where the original timing claim on the paper's test split does not directly apply.

## Inputs

- Preprocessed MSP-derived spectra DataFrames (individual collision-energy spectra variant)
- Preprocessed MSP-derived spectra DataFrames (merged spectra variant, e.g. merged collision energies with or without neutral losses)
- PyTorch CNN training configuration (model_positive.bin / model_negative.bin architecture, batch size, optimizer)

## Outputs

- Wall-clock epoch time per spectral-representation variant (minutes)
- Epoch-time ratio between variants (paper reference: ~17x)
- Comparison supporting decision on spectral representation for training

## How to apply

Prepare two training configurations that differ only in spectral representation: one feeding individual collision-energy spectra per molecule and one feeding a single merged spectrum (collision energies combined, intensities binarized). Time one full training epoch under each configuration with identical hardware, batch size, and optimizer settings; for ChemEmbed this means PyTorch CNN training on the preprocessed MSP-derived DataFrames. Compute the epoch-time ratio; in the paper's experiment individual spectra took over 95 minutes per epoch versus 5.5 minutes with merged spectra (about a 17x ratio), because the higher sparsity of individual spectra complicates learning and multiplies the number of training samples. Treat a ratio close to 17x as confirmation that merged spectra reproduce the paper's computational-efficiency finding, and use the wall-clock measurements to inform whether the added embedding-quality benefit (mean Euclidean distance improving from 40 to 24.5 to 23.1) justifies the cost of individual or merged-plus-neutral-loss variants.

## Related tools

- **PyTorch** (Runs the CNN training whose epoch time is benchmarked under each spectral-representation variant) — https://github.com/faizanurv/ChemEmbed
- **ChemEmbed** (Provides the preprocessing pipeline (MSP to DataFrames, spectral merging) and CNN training code whose runtime is measured) — https://github.com/faizanurv/ChemEmbed
- **pandas >= 1.0.0** (Holds the preprocessed spectral DataFrames fed to the timed training loop)

## Examples

```
chemembed --input_file_type without_smiles --adduct "+" --msp_file_positive merged_spectra.msp --model_path_positive data_model_files/model_positive.bin --reference_database data_model_files/chemembed_reference_mol2vec_300d_520k.parquet --top_n_candidates 5 --prediction_results results.csv
```

## Evaluation signals

- One epoch with individual spectra takes over 95 minutes while merged spectra take about 5.5 minutes, giving a ratio close to 17x
- Both timing runs use identical hardware, batch size, and optimizer settings so the only differing factor is spectral representation
- Timing is measured over a full epoch (not a batch) to amortize per-epoch startup overhead
- The timing comparison is paired with an embedding-quality check (e.g. mean Euclidean distance from 40 to 24.5 to 23.1 in positive ionization) to confirm the speed-up is not offset by accuracy loss

## Limitations

- Timing was reported on the paper's specific hardware and test split; absolute minutes will differ on other machines, so only the ratio is comparable
- The ~17x speed-up applies to CNN training on MSP-derived spectra and does not automatically transfer to inference-time candidate matching
- Merged spectra reduce epoch time but individual spectra may still be preferred if the added embedding quality (e.g. 23.1 vs 24.5 Euclidean distance) outweighs the cost for a given application

## Evidence

- [results] Timing comparison is anchored to the test-split ablation whose embedding quality improved from individual (40) to merged (24.5) to merged+neutral-loss (23.1) spectra: [evidence span withheld — anchor status: not_found]
