---
name: dual-encoder-feature-fusion
description: Use when you need to predict LC retention times from SMILES/InChI structures
  under a new chromatographic method and want to jointly exploit both topological
  graph structure and molecular fingerprints rather than a single descriptor stream.
when_to_use_negative:
- Input is already a per-molecule feature/descriptor table with no structure files
  (InChI/SMILES), since the pipeline requires RDKit graph and fingerprint generation
  from raw structures.
- Target property is not liquid-chromatography retention time under a comparable chromatographic
  method — the model is trained/transferred for RT prediction, not general molecular
  property regression.
- Only one data modality is available (e.g. fingerprints alone or graphs alone); the
  architecture requires both streams for the Output-Block fusion.
edam_operation: http://edamontology.org/operation_0327
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3391
tools:
- name: RT-Transformer
  role: 'The dual-encoder model being constructed: graph encoder + fingerprint encoder
    + Output-Block fusion for RT prediction'
  repo: https://github.com/01dadada/RT-Transformer
- name: PyTorch
  role: Implements the 12 stacked 1D-Transformer blocks for fingerprint encoding and
    the Output-Block fusion layers
- name: PyTorch Geometric
  role: Implements the 9 stacked ResGAT blocks (graph attention with residual connections)
    for the molecular-graph encoder
- name: RDKit
  role: Parses SMILES/InChI into molecular graphs and computes the 2048-bit Morgan
    fingerprints that form the two input streams
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:0b0a1c88572abafbf3de5e75005a99dc6a90c6ea294abf50a8a655693c68128d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - PyTorch
  - PyTorch Geometric
  - RDKit
  - RT-Transformer
  edam_operation: http://edamontology.org/operation_0327
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# dual-encoder-feature-fusion

## Summary

Builds RT-Transformer, a dual-encoder deep neural network that encodes a molecule's 2D graph with 9 ResGAT graph-attention blocks and its 2048-bit Morgan fingerprint with 12 1D-Transformer blocks, then fuses the two 512D representations in an Output-Block to predict liquid-chromatography retention time.

## When to use

Use when you need to predict LC retention times from SMILES/InChI structures under a new chromatographic method and want to jointly exploit both topological graph structure and molecular fingerprints rather than a single descriptor stream.

## When NOT to use

- Input is already a per-molecule feature/descriptor table with no structure files (InChI/SMILES), since the pipeline requires RDKit graph and fingerprint generation from raw structures.
- Target property is not liquid-chromatography retention time under a comparable chromatographic method — the model is trained/transferred for RT prediction, not general molecular property regression.
- Only one data modality is available (e.g. fingerprints alone or graphs alone); the architecture requires both streams for the Output-Block fusion.

## Inputs

- SMILES or InChI strings parsed with RDKit into molecular graphs (node/atom features and bond/edge features)
- 2048-bit Morgan fingerprints computed from the same structures
- Training/transfer-learning CSV with 'InChI' and 'RT' columns (data.csv at the repo root)

## Outputs

- Scalar retention-time prediction per molecule
- 512D graph-level feature vector and 512D fingerprint feature vector (intermediate representations)
- Trained model state dictionaries (best_state_dict.pth for full data, best_state_download_dict.pth for retained-data training)

## How to apply

Parse molecules with RDKit into two streams: molecular graphs (atoms as nodes, bonds as edges) and 2048D Morgan fingerprints. Feed the graph through 9 stacked ResGAT blocks — each block uses three-head graph attention with residual connections and layer normalization, mapping node features to a 512D graph-level representation — and feed the fingerprint through 12 stacked 1D-Transformer blocks that embed the 2048-bit vector into a 512D representation. In the Output-Block, concatenate/project the two 512D vectors into fused layers ending in a scalar RT prediction. For new chromatographic conditions, start from the pre-trained model (trained on the 80,038-molecule SMRT dataset) and fine-tune via transfer learning; for retraining, prepare a CSV with 'InChI' and 'RT' columns as data.csv and run train.py. The residual connections and layer normalization in each ResGAT block stabilize the deep 9-block stack, while the Transformer's self-attention lets the model weigh informative fingerprint bits non-linearly.

## Related tools

- **RT-Transformer** (The dual-encoder model being constructed: graph encoder + fingerprint encoder + Output-Block fusion for RT prediction) — https://github.com/01dadada/RT-Transformer
- **PyTorch** (Implements the 12 stacked 1D-Transformer blocks for fingerprint encoding and the Output-Block fusion layers)
- **PyTorch Geometric** (Implements the 9 stacked ResGAT blocks (graph attention with residual connections) for the molecular-graph encoder)
- **RDKit** (Parses SMILES/InChI into molecular graphs and computes the 2048-bit Morgan fingerprints that form the two input streams)

## Examples

```
python ./test.py
```

## Evaluation signals

- Forward-pass shape check: graph encoder outputs a 512D vector, fingerprint encoder outputs a 512D vector, and the Output-Block yields a scalar RT prediction
- On the SMRT dataset, average absolute error should reach ~27.30 after removing not-retained molecules and ~33.41 on full data
- Fine-tuned model on 5 chromatographic-condition datasets from PredRet should be competitive with state-of-the-art RT predictors
- Model files best_state_dict.pth / best_state_download_dict.pth load without missing-key errors when instantiated with the 9+12+Output-Block architecture
- Web/HuggingFace deployment reproduces predictions matching the QuickStart.ipynb output for the same input structures

## Limitations

- Performance degrades to 33.41 average absolute error on full data (not-retained molecules included) versus 27.30 after removal, so 'not retained' molecules are a known error source.
- Different chromatographic conditions yield different RTs for the same metabolite; direct cross-method prediction without transfer learning/fine-tuning underperforms — the paper pre-trains on SMRT and transfers to 5 condition-specific datasets.
- Scalability was demonstrated on 41 external LC datasets — evidence is liquid-chromatography-specific; gas-chromatography RT is not addressed in the provided context.
- Repository has no changelog; version provenance of pretrained weights is not documented.

## Evidence

- [abstract] architecture composition: "we present RT-Transformer, a novel deep neural network model coupled with graph attention network and 1D-Transformer, which can predict retention times under any chromatographic methods"
- [abstract] pretraining + transfer strategy: "First, we obtain a pre-trained model by training RT-Transformer on the large small molecule retention time dataset containing 80 038 molecules, and then transfer the resulting model to different"
- [readme] dual input streams and encoders: [evidence span withheld — anchor status: external_doc]
- [readme] retraining data format: [evidence span withheld — anchor status: external_doc]
- [readme] pretrained weight files: [evidence span withheld — anchor status: external_doc]
