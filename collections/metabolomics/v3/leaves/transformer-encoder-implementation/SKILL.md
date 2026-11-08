---
name: transformer-encoder-implementation
description: Use when building a retention-time predictor from molecular structure
  where one input stream is a fixed-length fingerprint (e.g. 2048D Morgan) and a transformer
  encoder must be implemented in PyTorch to turn that fingerprint into a fixed-size
  embedding.
when_to_use_negative:
- The input is already a feature/embedding table and no molecular graph or fingerprint
  must be encoded
- The task is GC retention-time prediction or a chromatographic method with no transferable
  LC data; the paper's pre-training and transfer evidence covers liquid-chromatography
  datasets (SMRT, PredRet)
- A non-PyTorch stack is mandated and no equivalent of torch_geometric/torch-scatter
  is available
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3172
tools:
- name: RT-Transformer
  role: The full model whose 1D-Transformer fingerprint branch this skill implements
  repo: https://github.com/01dadada/RT-Transformer
- name: PyTorch
  role: Framework for implementing the 12 stacked 1D-Transformer blocks and Output-Block
- name: PyTorch Geometric
  role: Framework for the parallel ResGAT graph encoder fused with the fingerprint
    encoder
- name: RDKit
  role: Computes the 2048D Morgan fingerprints and parses InChI-derived molecular
    graphs
- name: torch-scatter
  role: PyG dependency needed to run the graph branch alongside the transformer encoder
- name: torch-sparse
  role: PyG dependency needed to run the graph branch alongside the transformer encoder
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:5a24cd26a4c00db556de777f8a4d5761f445f4631dd86aab39834872e87bfa26
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - PyTorch
  - PyTorch Geometric
  - RDKit
  - RT-Transformer
  - torch-scatter
  - torch-sparse
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# transformer-encoder-implementation

## Summary

Implements the 1D-Transformer fingerprint encoder of RT-Transformer: 12 stacked 1D-Transformer blocks that embed a 2048-dimensional Morgan fingerprint and produce a 512-dimensional molecular representation, which is fused with a graph-attention branch to predict liquid-chromatography retention time.

## When to use

Use when building a retention-time predictor from molecular structure where one input stream is a fixed-length fingerprint (e.g. 2048D Morgan) and a transformer encoder must be implemented in PyTorch to turn that fingerprint into a fixed-size embedding.

## When NOT to use

- The input is already a feature/embedding table and no molecular graph or fingerprint must be encoded
- The task is GC retention-time prediction or a chromatographic method with no transferable LC data; the paper's pre-training and transfer evidence covers liquid-chromatography datasets (SMRT, PredRet)
- A non-PyTorch stack is mandated and no equivalent of torch_geometric/torch-scatter is available

## Inputs

- Molecular structures parsed as graphs (atoms and bonds) via RDKit from InChI strings
- 2048-dimensional Morgan fingerprints per molecule
- CSV dataset with 'InChI' and 'RT' columns (data.csv)
- Pretrained weight files best_state_dict.pth / best_state_download_dict.pth (from the HuggingFace space) for transfer or fine-tuning

## Outputs

- 512-dimensional fingerprint representation per molecule from the 1D-Transformer encoder
- 512-dimensional graph-level representation per molecule from the 9 ResGAT blocks
- Fused scalar retention-time prediction per molecule (e.g. saved as rt_prediction.npy)
- Trained/fine-tuned model checkpoints

## How to apply

Parse each molecule with RDKit into its InChI-derived structure and compute a 2048-dimensional Morgan fingerprint; embed this fingerprint as the input sequence to a 1D-Transformer implemented in PyTorch. Stack 12 1D-Transformer blocks so the fingerprint stream is encoded in parallel with the graph stream, which uses 9 ResGAT blocks in PyTorch Geometric; both encoders target 512-dimensional outputs. Combine the 512D graph feature and 512D fingerprint feature in an Output-Block that concatenates/projects the two vectors into fused layers ending in a scalar retention-time prediction. Decide between training from scratch (train.py on a CSV with InChI and RT columns) and fine-tuning the released pretrained weights (best_state_dict.pth for full data, best_state_download_dict.pth for retained-only data); the pre-trained model is first trained on the 80,038-molecule SMRT dataset and then transferred to chromatographic conditions from PredRet via transfer.py. Verify a forward pass by asserting the 512D graph-feature shape, 512D fingerprint-feature shape, and scalar RT output shape.

## Related tools

- **RT-Transformer** (The full model whose 1D-Transformer fingerprint branch this skill implements) — https://github.com/01dadada/RT-Transformer
- **PyTorch** (Framework for implementing the 12 stacked 1D-Transformer blocks and Output-Block)
- **PyTorch Geometric** (Framework for the parallel ResGAT graph encoder fused with the fingerprint encoder)
- **RDKit** (Computes the 2048D Morgan fingerprints and parses InChI-derived molecular graphs)
- **torch-scatter** (PyG dependency needed to run the graph branch alongside the transformer encoder)
- **torch-sparse** (PyG dependency needed to run the graph branch alongside the transformer encoder)

## Examples

```
python ./test.py  # or: python transfer.py with data.csv (InChI,RT columns) and best_state_dict.pth downloaded from the HuggingFace space
```

## Evaluation signals

- Forward pass produces a 512D graph feature vector and a 512D fingerprint feature vector before fusion
- Final output is a scalar RT prediction (assert output shape; save as rt_prediction.npy)
- test.py runs without error on the provided checkpoints, reproducing reported average absolute errors of 27.30 (retained molecules removed) and 33.41 (no samples removed) on SMRT
- Transfer learning on a PredRet-derived data.csv (InChI + RT columns) yields competitive performance against state-of-the-art RT predictors
- Pretrained weights load cleanly from best_state_dict.pth (full data) or best_state_download_dict.pth (retained data) into the instantiated model

## Limitations

- Different chromatographic conditions yield different retention times for the same metabolite; the encoder alone does not generalize and must be fine-tuned (transfer.py) per method
- Performance degrades when 'not retained' molecules are kept in evaluation (33.41 vs 27.30 average absolute error on SMRT)
- Pretrained checkpoints must be downloaded separately from the HuggingFace space; they are not bundled in the repo
- The paper's citations list no changelog for 01dadada/RT-Transformer, so API drift in the code is possible
- SMRT dataset comes from a separate publication and transfer datasets from PredRet; license/availability of those datasets is outside this repo

## Evidence

- [abstract] Architecture: 12 transformer blocks on fingerprint: "we present RT-Transformer, a novel deep neural network model coupled with graph attention network and 1D-Transformer, which can predict retention times under any chromatographic methods"
- [abstract] Pretraining scale for transfer: "First, we obtain a pre-trained model by training RT-Transformer on the large small molecule retention time dataset containing 80 038 molecules"
- [abstract] Expected error on SMRT: "the average absolute error reached 27.30 after removing not retained molecules. Still, it reached 33.41 when no samples were removed"
- [readme] README data-format contract: [evidence span withheld — anchor status: external_doc]
- [readme] Pretrained checkpoint names: [evidence span withheld — anchor status: external_doc]
- [readme] Source-code location: "The source code for the model is available at https://github.com/01dadada/RT-Transformer"
