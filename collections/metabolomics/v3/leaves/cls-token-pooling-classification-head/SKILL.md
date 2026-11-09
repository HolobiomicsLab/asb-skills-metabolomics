---
name: cls-token-pooling-classification-head
description: 'Use when adapting a pre-trained transformer language model for metabolomics-based
  disease prediction: you need a pooled sequence representation from a learnable [CLS]
  token feeding a classification head over metabolite-derived token embeddings.'
when_to_use_negative:
- Input is a raw NMR spectra file rather than a quantified metabolite feature table
  — this skill operates on 249 named metabolite columns, not raw spectral data
- The task requires multi-class or survival modeling outputs rather than binary classification
  with a pooled [CLS] logit head
- Metabolite-metabolite interaction information is irrelevant and a plain transformer
  without the correlation-matrix attention bias suffices
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Framework for implementing the [CLS] token, BERT encoder layers, and classification
    head
- name: metabolome-transformer
  role: CLI tool (train-binary subcommand) that fine-tunes the model with the [CLS]-pooled
    classification head for binary disease prediction
  repo: https://github.com/Qiu-Shizheng/Metabolome-Transformer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-66163-3
    title: 'MetaboLM: a metabolomic language model for multi-disease early prediction
      and risk stratification'
  source_content_hash: sha256:4308dc65665c72f8383f520d525cac21ff7be221f9058385431fe721ce283588
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Qiu-Shizheng/MetaboLM
  repo_url: https://github.com/Qiu-Shizheng/Metabolome-Transformer
  tools:
  - PyTorch
  - metabolome-transformer
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cls-token-pooling-classification-head

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a fine-tuning classification head that prepends a learnable [CLS] token (without positional encoding) to the metabolite embeddings and pools its final-layer representation to output class logits, as used in MetaboLM's BERT-encoder metabolomic language model for multi-disease prediction.

## When to use

Use when adapting a pre-trained transformer language model for metabolomics-based disease prediction: you need a pooled sequence representation from a learnable [CLS] token feeding a classification head over metabolite-derived token embeddings.

## When NOT to use

- Input is a raw NMR spectra file rather than a quantified metabolite feature table — this skill operates on 249 named metabolite columns, not raw spectral data
- The task requires multi-class or survival modeling outputs rather than binary classification with a pooled [CLS] logit head
- Metabolite-metabolite interaction information is irrelevant and a plain transformer without the correlation-matrix attention bias suffices

## Inputs

- Metabolite value table with one sample ID column and the 249 metabolite columns listed in assets/pretrained/metabolite_columns.txt (CSV format)
- Training table with sample ID column, metabolite columns, and a binary label column for fine-tuning (e.g., train.csv)

## Outputs

- Class logits from the classification head on the pooled [CLS] representation
- Per-sample prediction probabilities and binary predictions (e.g., predictions.csv with sample_id, probability, prediction columns)

## How to apply

Build the metabolite embedding layer that projects the 249 NMR metabolite features to a 768-dimensional space and combines them via a Hadamard (element-wise) product, then prepend a learnable [CLS] token to the token sequence without positional encoding. Pass the sequence through 12 stacked Transformer encoder layers with 8 attention heads, computing attention as softmax(QK^T/sqrt(d) + alpha*C-tilde), where alpha is a learnable scalar and C-tilde is the metabolite-metabolite association matrix used as an additive attention bias. Attach the fine-tuning classification head that takes the pooled [CLS] representation and outputs class logits (binary labels for current or future disease prediction tasks). Run a smoke test that instantiates the full model and performs a forward pass on a dummy batch to verify output shapes and that gradients flow to alpha and the [CLS] token.

## Related tools

- **PyTorch** (Framework for implementing the [CLS] token, BERT encoder layers, and classification head)
- **metabolome-transformer** (CLI tool (train-binary subcommand) that fine-tunes the model with the [CLS]-pooled classification head for binary disease prediction) — https://github.com/Qiu-Shizheng/Metabolome-Transformer

## Examples

```
metabolome-transformer train-binary --input train.csv --output-dir results/current_task --id-column sample_id --label-column label --task current --epochs 10 --batch-size 12
```

## Evaluation signals

- Smoke-test forward pass on a dummy batch returns the expected logit/shape output without errors
- Gradients flow to both the learnable [CLS] token embedding and the attention scalar alpha after a backward pass
- Fine-tuning on a labeled binary task (current or future disease) converges and produces a classifier artifact (e.g., results/current_task/classifier.pt)
- Prediction output CSV contains exactly the ID column plus 'probability' and 'prediction' columns, with probabilities in [0, 1]

## Limitations

- Positional encoding is intentionally omitted for the [CLS] token; the model relies on the metabolite-metabolite association matrix (C-tilde) added as an attention bias rather than sequence order
- Pre-training cohort was predominantly of European descent, healthier, and of higher socioeconomic status, which may limit generalizability of downstream fine-tuned classifiers
- NMR-based metabolomics may miss metabolites detectable by other analytical platforms, bounding what the [CLS]-pooled classifier can learn

## Evidence

- [methods] Prepend a learnable [CLS] token without positional encoding to the token sequence: [evidence span withheld — anchor status: not_found]
- [methods] Add the fine-tuning classification head that takes the pooled [CLS] representation and outputs class logits: [evidence span withheld — anchor status: not_found]
- [methods] Run a smoke test that instantiates the full model and performs a forward pass on a dummy batch to verify shapes and that gradients flow to alpha and the [CLS] token: [evidence span withheld — anchor status: not_found]
- [intro] we fine-tuned the pre-trained model using samples from individuals diagnosed with 16 common chronic diseases during the follow-up period: "Subsequently, we fine-tuned the pre-trained model using samples from individuals diagnosed with 16 common chronic diseases during the follow-up period"
- [readme] The training file should contain the sample ID column, metabolite columns, and a binary label column: [evidence span withheld — anchor status: external_doc]
- [readme] Prediction output contains the ID column, probability, and prediction: [evidence span withheld — anchor status: external_doc]
