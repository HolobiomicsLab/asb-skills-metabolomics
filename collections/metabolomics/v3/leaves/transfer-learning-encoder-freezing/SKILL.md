---
name: transfer-learning-encoder-freezing
description: Use when predicting retention times on a new liquid-chromatography dataset
  (e.g., a PredRet dataset with 'InChI' and 'RT' columns) under chromatographic conditions
  different from the pre-training data, and a pre-trained RT-Transformer checkpoint
  is available.
when_to_use_negative:
- When no suitable pre-trained checkpoint exists for the molecular domain — retrain
  from scratch with train.py instead of freezing a mismatched encoder
- When the target dataset's retention behavior is expected to require re-learning
  molecular representations, not just re-calibrating the prediction head
- When the input is not molecule-RT pairs (e.g., raw LC-MS mzML data or already-predicted
  retention times) — this skill operates on InChI/RT CSV tables
edam_operation: http://edamontology.org/operation_3937
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3172
tools:
- name: RT-Transformer
  role: Provides the ResGAT + 1D-Transformer architecture, the pre-trained SMRT checkpoints,
    and the transfer.py / train.py entry points used to freeze the encoder and fine-tune
    the Output-Block
  repo: https://github.com/01dadada/RT-Transformer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:e56926f4bd44bc6a9c4c0db54ff3309412b17aba230843523d48c5b496d51b80
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - RT-Transformer
  edam_operation: http://edamontology.org/operation_3937
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# transfer-learning-encoder-freezing

## Summary

Transfer the pre-trained RT-Transformer (ResGAT + 1D-Transformer encoder, pre-trained on the 80,038-molecule SMRT dataset) to a new chromatographic-method retention-time dataset by freezing the encoder weights and re-initializing/fine-tuning only the Output-Block. This preserves the general retention-time representation learned from SMRT while adapting the prediction head to the target chromatographic conditions.

## When to use

Use when predicting retention times on a new liquid-chromatography dataset (e.g., a PredRet dataset with 'InChI' and 'RT' columns) under chromatographic conditions different from the pre-training data, and a pre-trained RT-Transformer checkpoint is available.

## When NOT to use

- When no suitable pre-trained checkpoint exists for the molecular domain — retrain from scratch with train.py instead of freezing a mismatched encoder
- When the target dataset's retention behavior is expected to require re-learning molecular representations, not just re-calibrating the prediction head
- When the input is not molecule-RT pairs (e.g., raw LC-MS mzML data or already-predicted retention times) — this skill operates on InChI/RT CSV tables

## Inputs

- Target dataset CSV with 'InChI' and 'RT' columns (e.g., a PredRet chromatographic-method dataset)
- Pre-trained RT-Transformer checkpoint (best_state_dict.pth for full-SMRT or best_state_download_dict.pth for retained-molecules pre-training)

## Outputs

- Fine-tuned Output-Block weights adapted to the target chromatographic method
- Per-fold R2 scores over the cross-validation folds; aggregated per-dataset R2 distribution table (e.g., r2_distribution.csv)

## How to apply

Load a pre-trained RT-Transformer checkpoint — for the most stable transfer, use best_state_dict.pth (trained on the full SMRT dataset) rather than best_state_download_dict.pth (retained molecules only). Prepare the target dataset as a CSV with 'InChI' and 'RT' columns renamed to data.csv at the repository root. Freeze the pre-trained ResGAT and 1D-Transformer encoder weights and re-initialize only the Output-Block, then fine-tune that head on the target dataset using 10-fold cross-validation repeated over five random seeds (50 runs per dataset). Judge success by the stability criterion that no R2 score on held-out folds falls below -0.5, and by outperforming the retained-molecules pre-trained model when R2 exceeds 0.75. The full-SMRT checkpoint was empirically more stable across the 41 external PredRet datasets, so prefer it for transfer.

## Related tools

- **RT-Transformer** (Provides the ResGAT + 1D-Transformer architecture, the pre-trained SMRT checkpoints, and the transfer.py / train.py entry points used to freeze the encoder and fine-tune the Output-Block) — https://github.com/01dadada/RT-Transformer

## Examples

```
cp my_dataset.csv data.csv && python transfer.py
```

## Evaluation signals

- No R2 score on any held-out fold (per dataset/seed/fold combination) falls below -0.5 when using the full-SMRT pre-trained checkpoint
- The transferred model outperforms the retained-molecules pre-trained model on datasets where R2 exceeds 0.75
- Encoder weights are bitwise unchanged from the checkpoint after fine-tuning; only the Output-Block parameters differ from initialization
- 10-fold CV × 5 seeds (50 runs per dataset) complete and the aggregated R2 distribution table (r2_distribution.csv) has one distribution per target dataset

## Limitations

- Transfer stability depends on checkpoint choice: the full-SMRT model (best_state_dict.pth) is more stable; the retained-molecules model can produce R2 scores below -0.5 on some PredRet datasets
- The encoder weights are frozen, so performance is bounded by the representation learned on SMRT; datasets requiring new molecular representations are not addressed by head-only fine-tuning
- Transfer results are per-chromatographic-method: performance varies across the 41 PredRet datasets, and low-R2 datasets are not rescued by this procedure

## Evidence

- [methods] freeze encoder, fine-tune head: [evidence span withheld — anchor status: not_found]
- [results] stability criterion: [evidence span withheld — anchor status: not_found]
- [readme] input format: [evidence span withheld — anchor status: not_found]
- [abstract] transfer rationale: "First, we obtain a pre-trained model by training RT-Transformer on the large small molecule retention time dataset containing 80 038 molecules, and then transfer the resulting model to different"
