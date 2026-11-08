---
name: forward-pass-shape-validation
description: Decision support for verifying a newly implemented dual-branch model
  (graph encoder + sequence encoder) produces correctly dimensioned representations
  before committing to a training run.
when_to_use_negative:
- When the model architecture has already been validated and training is under way
  — repeated shape assertions add overhead without new information
- When the input is an already-computed feature table or embedding rather than a raw
  molecular structure requiring graph/fingerprint construction
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0154
tools:
- name: RT-Transformer
  role: The model whose forward pass is being validated; its three modules (ResGAT
    graph encoder, 1D-Transformer fingerprint encoder, Output-Block) define the expected
    tensor shapes.
  repo: https://github.com/01dadada/RT-Transformer
- name: PyTorch Geometric
  role: Implements the ResGAT graph encoder whose node/edge tensor shapes must match
    the 512D expectation.
  repo: https://github.com/01dadada/RT-Transformer
- name: RDKit
  role: Parses molecule structures into graphs and 2048D Morgan fingerprints that
    feed the validated forward pass.
  repo: https://github.com/01dadada/RT-Transformer
- name: PyTorch
  role: Holds the 1D-Transformer fingerprint encoder and Output-Block whose output
    dimensions are asserted.
  repo: https://github.com/01dadada/RT-Transformer
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:49a743c58fa47853b72901cdeff985addc3e1e3d383b5f683312b8bbb455bb90
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - PyTorch
  - PyTorch Geometric
  - RDKit
  - RT-Transformer
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# forward-pass-shape-validation

## Summary

Sanity-check a deep learning model's forward pass by asserting intermediate and final tensor shapes before training, using the RT-Transformer's 512D graph/fingerprint features and scalar RT prediction as invariants.

## When to use

Decision support for verifying a newly implemented dual-branch model (graph encoder + sequence encoder) produces correctly dimensioned representations before committing to a training run.

## When NOT to use

- When the model architecture has already been validated and training is under way — repeated shape assertions add overhead without new information
- When the input is an already-computed feature table or embedding rather than a raw molecular structure requiring graph/fingerprint construction

## Inputs

- Sample molecule structure parsed into a molecular graph (RDKit)
- 2048D Morgan fingerprint for the same molecule
- Instantiated RT-Transformer model (9 ResGAT blocks + 12 1D-Transformer blocks + Output-Block)

## Outputs

- rt_prediction.npy — NumPy file holding the forward-pass retention-time prediction vector
- Assertion log confirming 512D graph features, 512D fingerprint features, and final scalar RT output shape

## How to apply

Feed a sample molecule processed with RDKit (molecular graph plus 2048D Morgan fingerprint) through the freshly instantiated model. Assert that the graph branch — nine stacked ResGAT blocks with three-head graph attention, residual connections and layer normalization — outputs a 512D graph-level vector, and that the fingerprint branch — twelve stacked 1D-Transformer blocks embedding the fingerprint — likewise emits a 512D vector. Then confirm the Output-Block, which concatenates/projects the two 512D vectors, yields a scalar retention-time prediction (shape [1] or [batch, 1]). Save the prediction vector with NumPy (rt_prediction.npy) so the shape and values are inspectable. Rationale: catching dimension mismatches between the PyTorch Geometric graph stream and the PyTorch fingerprint stream is far cheaper at instantiation than at training time.

## Related tools

- **RT-Transformer** (The model whose forward pass is being validated; its three modules (ResGAT graph encoder, 1D-Transformer fingerprint encoder, Output-Block) define the expected tensor shapes.) — https://github.com/01dadada/RT-Transformer
- **PyTorch Geometric** (Implements the ResGAT graph encoder whose node/edge tensor shapes must match the 512D expectation.) — https://github.com/01dadada/RT-Transformer
- **RDKit** (Parses molecule structures into graphs and 2048D Morgan fingerprints that feed the validated forward pass.) — https://github.com/01dadada/RT-Transformer
- **PyTorch** (Holds the 1D-Transformer fingerprint encoder and Output-Block whose output dimensions are asserted.) — https://github.com/01dadada/RT-Transformer

## Examples

```
from RTTransformer import RTTransformer; model = RTTransformer(); out = model(graph, fp2048); assert out.shape == (1, 1); np.save('rt_prediction.npy', out.detach().numpy())
```

## Evaluation signals

- Graph-branch output has shape [num_nodes, 512] (or pooled to a 512D graph-level vector) after 9 ResGAT blocks
- Fingerprint-branch output has shape [1, 512] for a single 2048D Morgan fingerprint input
- Final output is a scalar per molecule (shape [1] or [batch, 1]), matching the paper's average-absolute-error evaluation (27.30 after removing not-retained molecules; 33.41 on full data)
- rt_prediction.npy is written and loads via np.load without shape errors
- All three branches run without a dimensionality assertion failure in a single forward call on a sample SMILES/InChI from the SMRT dataset

## Limitations

- Shape validation catches structural wiring errors but not correctness of learned parameters — the model can pass all assertions and still predict RT poorly
- Firmly tied to the 512D/2048D/scalar contract of RT-Transformer; transferring the check to a different dual-branch architecture requires re-deriving expected dimensions
- Pretrained weights (best_state_dict.pth, best_state_download_dict.pth) must be loaded separately from the shape check, or a randomly initialised forward pass will pass validation but produce meaningless RT values

## Evidence

- [methods] stacks 9 ResGAT blocks so atoms aggregate information from other atoms and bonds, and stacks 12 1D-Transformer blocks to process the 2048D molecular fingerprint, sending both to the Output-Block for the final RT prediction.: [evidence span withheld — anchor status: not_found]
- [methods] assert the intermediate shapes (512D graph features, 512D fingerprint features) and final output shape are correct: [evidence span withheld — anchor status: not_found]
- [methods] Save the forward-pass RT prediction vector to rt_prediction.npy using NumPy.: [evidence span withheld — anchor status: not_found]
- [readme] The RT-Tranformer combine the fingerprint and the molecular graph data and predict retention time as the output.: [evidence span withheld — anchor status: external_doc]
- [abstract] coupled with graph attention network and 1D-Transformer, which can predict retention times: "we present RT-Transformer, a novel deep neural network model coupled with graph attention network and 1D-Transformer, which can predict retention times"
