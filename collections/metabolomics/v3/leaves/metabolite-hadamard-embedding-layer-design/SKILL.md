---
name: metabolite-hadamard-embedding-layer-design
description: Use when building or reimplementing a transformer model whose inputs
  are tabular metabolite (or similar numeric feature) vectors and you need feature-to-token
  embeddings plus interaction-aware attention biasing for pre-training or fine-tuning.
when_to_use_negative:
- Input is raw spectral or mass-spec data (e.g. mzML peaks) rather than a table of
  already quantified NMR metabolite values per sample
- The task is purely missing-value imputation or plain tabular prediction where a
  correlation-biased transformer's inductive bias is unnecessary (e.g. hundreds of
  weakly interacting features with simple linear relationships)
- Only a handful of metabolites are available, making a 768-dimensional 12-layer BERT-scale
  architecture over-parameterized relative to the data
tools:
- name: PyTorch
  role: Framework for implementing the Hadamard-product metabolite embedding layer,
    the learnable [CLS] token, the 12-layer/8-head BERT encoder with correlation-matrix
    attention bias, and the classification head; used for smoke-test forward passes
- name: Metabolome-Transformer
  role: Companion command-line implementation (metabolome-transformer impute / train-binary
    / predict-binary) that consumes the 249-metabolite column table, imputes missing
    values, and fine-tunes binary disease classifiers on the MetaboLM embeddings
  repo: github.com/Qiu-Shizheng/Metabolome-Transformer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-66163-3
    title: 'MetaboLM: a metabolomic language model for multi-disease early prediction
      and risk stratification'
  source_content_hash: sha256:f53c00685527f5c03aa64bab8c82b5fcb53c20989b41149999d16829a949bdfc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Qiu-Shizheng/MetaboLM
  tools:
  - Metabolome-Transformer
  - PyTorch
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# metabolite-hadamard-embedding-layer-design

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Designs the MetaboLM input pipeline that converts a table of 249 quantified NMR plasma metabolite values into 768-dimensional token embeddings via a Hadamard (element-wise) product, prepends a learnable [CLS] token, and feeds the sequence to a 12-layer BERT encoder whose attention incorporates a metabolite-metabolite correlation matrix as an additive bias.

## When to use

Use when building or reimplementing a transformer model whose inputs are tabular metabolite (or similar numeric feature) vectors and you need feature-to-token embeddings plus interaction-aware attention biasing for pre-training or fine-tuning.

## When NOT to use

- Input is raw spectral or mass-spec data (e.g. mzML peaks) rather than a table of already quantified NMR metabolite values per sample
- The task is purely missing-value imputation or plain tabular prediction where a correlation-biased transformer's inductive bias is unnecessary (e.g. hundreds of weakly interacting features with simple linear relationships)
- Only a handful of metabolites are available, making a 768-dimensional 12-layer BERT-scale architecture over-parameterized relative to the data

## Inputs

- Tabular CSV with one sample_id column and the 249 metabolite columns listed in assets/pretrained/metabolite_columns.txt (values may contain missing entries)
- Optional binary label column (e.g. 'label') for fine-tuning disease-prediction heads
- Metabolite-metabolite association/correlation matrix C-tilde used as the additive attention bias

## Outputs

- 768-dimensional token embedding sequence per sample with a learnable [CLS] token prepended
- Trained MetaboLM encoder (12 layers, 8 heads, learnable alpha) and classification-head logits
- Binary prediction outputs (probability and 0/1 prediction per sample_id) for downstream MetaboRS risk scoring

## How to apply

Define an embedding layer that projects each of the input metabolite features (the 249 NMR metabolite columns, e.g. Total Cholesterol, Total Cholesterol Minus HDL-C) to a 768-dimensional space and combines them via a Hadamard (element-wise) product. Prepend a learnable [CLS] token to the token sequence without positional encoding, since metabolites are an unordered set rather than a sequence. Stack 12 Transformer encoder layers with 8 attention heads, computing attention as softmax(QK^T/sqrt(d) + alpha*C-tilde), where alpha is a learnable scalar and C-tilde is the metabolite-metabolite association (correlation) matrix used as an additive attention bias so attention focuses on biologically relevant metabolic networks. Instantiate the full model in PyTorch and run a smoke-test forward pass on a dummy batch. For reuse, the companion Metabolome-Transformer repo shows the practical input contract: one sample_id column plus the 249 metabolite columns, with missing values imputable before fine-tuning.

## Related tools

- **PyTorch** (Framework for implementing the Hadamard-product metabolite embedding layer, the learnable [CLS] token, the 12-layer/8-head BERT encoder with correlation-matrix attention bias, and the classification head; used for smoke-test forward passes)
- **Metabolome-Transformer** (Companion command-line implementation (metabolome-transformer impute / train-binary / predict-binary) that consumes the 249-metabolite column table, imputes missing values, and fine-tunes binary disease classifiers on the MetaboLM embeddings) — github.com/Qiu-Shizheng/Metabolome-Transformer

## Examples

```
metabolome-transformer impute --input examples/example_input.csv --output results/imputed.csv --id-column sample_id --batch-size 12
```

## Evaluation signals

- Smoke-test forward pass on a dummy batch produces logits of the expected shape and gradients flow to both the learnable alpha scalar and the [CLS] token
- Input CSV conforms to the repo contract: one sample_id column plus exactly the 249 metabolite columns enumerated in assets/pretrained/metabolite_columns.txt
- Embedding output dimensionality per token is 768 and the token sequence length equals 249 metabolites + 1 [CLS] token with no positional encodings applied
- Attention maps change when alpha or C-tilde are perturbed, confirming the correlation bias is actually applied inside softmax(QK^T/sqrt(d) + alpha*C-tilde)
- Fine-tuned classifier outputs contain per-sample probability and 0/1 prediction columns keyed by sample_id, as shown in the README example

## Limitations

- NMR-based metabolomics may miss metabolites detectable by other analytical platforms
- Metabolite columns may contain missing values that must be imputed (the impute subcommand fills empty cells before training/prediction)
- The pre-training cohort (83,744 UK Biobank participants) is predominantly of European descent, healthier, and of higher socioeconomic status, limiting generalizability of the learned metabolite interaction patterns

## Evidence

- [abstract] pre-training data scale and population: "a transformer-based language model pre-trained on plasma metabolomics data from 83,744 relatively healthy UK Biobank participants"
- [readme] input table contract from README: [evidence span withheld — anchor status: external_doc]
- [full_text] findings on architecture per source card: [evidence span withheld — anchor status: not_found]
