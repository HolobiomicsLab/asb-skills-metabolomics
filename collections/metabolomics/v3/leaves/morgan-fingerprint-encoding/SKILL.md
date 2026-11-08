---
name: morgan-fingerprint-encoding
description: Use when a deep-learning RT-prediction model requires a fixed-length
  substructure descriptor for each molecule — e.g. to complement a molecular-graph
  encoder with the 2048D Morgan fingerprint stream described in the RT-Transformer
  pipeline.
when_to_use_negative:
- The downstream model expects only graph-structured input (no fingerprint stream)
  — then the 2048D Morgan vector is irrelevant.
- Molecules cannot be parsed by RDKit (invalid or missing structures), so fingerprint
  generation will fail.
- The task requires per-atom or per-bond attributes rather than whole-molecule substructure
  hashing — a graph-encoder-only approach is the correct route.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: RDKit
  role: Used to parse molecular structures and compute the 2048-bit Morgan fingerprints
    that form the RT-Transformer's fingerprint input stream.
  repo: https://github.com/01dadada/RT-Transformer
- name: PyTorch
  role: Implements the 12 stacked 1D-Transformer blocks that embed the 2048D fingerprint
    into a 512D representation.
  repo: https://github.com/01dadada/RT-Transformer
- name: RT-Transformer
  role: The overall model whose fingerprint encoding pathway (Morgan fingerprint →
    1D-Transformer → 512D vector) this skill describes.
  repo: https://github.com/01dadada/RT-Transformer
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:ef3099e0af0ab9572b66a40afcf2af20ae26edda117291af39b8fba8791a20d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - PyTorch
  - PyTorch Geometric
  - RDKit
  - RT-Transformer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# morgan-fingerprint-encoding

## Summary

Compute 2048-bit Morgan fingerprints with RDKit and feed them as an input stream to the RT-Transformer's 1D-Transformer module, which encodes the fingerprint into a 512D representation that is fused with graph-level features for retention-time prediction.

## When to use

Use when a deep-learning RT-prediction model requires a fixed-length substructure descriptor for each molecule — e.g. to complement a molecular-graph encoder with the 2048D Morgan fingerprint stream described in the RT-Transformer pipeline.

## When NOT to use

- The downstream model expects only graph-structured input (no fingerprint stream) — then the 2048D Morgan vector is irrelevant.
- Molecules cannot be parsed by RDKit (invalid or missing structures), so fingerprint generation will fail.
- The task requires per-atom or per-bond attributes rather than whole-molecule substructure hashing — a graph-encoder-only approach is the correct route.

## Inputs

- Molecule structures (SMILES / InChI strings, or RDKit Mol objects parsed from the SMRT or PredRet CSV datasets)
- 2048-bit Morgan fingerprint vector per molecule (RDKit output, e.g. via AllChem.GetMorganFingerprintAsBitVect)

## Outputs

- 512D fingerprint representation per molecule (output of the 12 stacked 1D-Transformer blocks)
- Concatenated fused feature vector entering the Output-Block for the scalar RT prediction

## How to apply

Use RDKit to generate a 2048-bit Morgan fingerprint from each molecule's structure; the article treats this 2048D vector as one of the model's two input streams. Feed the fingerprint into a 1D-Transformer that stacks 12 blocks and projects it to a 512D fingerprint representation. Rationale: the fingerprint encodes local substructure motifs that the graph-attention module may not capture, and the RT-Transformer concatenates the 512D graph feature and 512D fingerprint feature in the Output-Block to make the final RT prediction. Keep fingerprint bit count at 2048 so the pre-trained model's input dimensionality matches; do not reduce the vector before the Transformer blocks.

## Related tools

- **RDKit** (Used to parse molecular structures and compute the 2048-bit Morgan fingerprints that form the RT-Transformer's fingerprint input stream.) — https://github.com/01dadada/RT-Transformer
- **PyTorch** (Implements the 12 stacked 1D-Transformer blocks that embed the 2048D fingerprint into a 512D representation.) — https://github.com/01dadada/RT-Transformer
- **RT-Transformer** (The overall model whose fingerprint encoding pathway (Morgan fingerprint → 1D-Transformer → 512D vector) this skill describes.) — https://github.com/01dadada/RT-Transformer

## Examples

```
from rdkit import Chem; from rdkit.Chem import AllChem; mol = Chem.MolFromInChI(inchi_str); fp = AllChem.GetMorganFingerprintAsBitVect(mol, 2, nBits=2048)
```

## Evaluation signals

- Fingerprint dimensionality must be 2048 bits per molecule — verify the vector length before feeding to the 1D-Transformer.
- The 1D-Transformer output must be a 512D vector; assert `fingerprint_features.shape == (batch, 512)`.
- Fingerprint bits should be reproducible for the same input structure — re-running RDKit fingerprint generation must yield identical bit vectors.
- The fused Output-Block must receive a concatenation of the 512D graph feature and 512D fingerprint feature (1024D total) before the scalar RT head.

## Limitations

- The README specifies Python 3.9 and a specific PyTorch-Geometric stack (torch-scatter, torch-sparse, torch-cluster); fingerprint encoding is coupled to that environment and may fail elsewhere.
- Fingerprint bit length (2048) is fixed by the pre-trained model's architecture — changing it requires retraining, not just re-encoding.
- Molecules that RDKit cannot parse (e.g. malformed InChI in the 'InChI' column of data.csv) will abort fingerprint generation.

## Evidence

- [abstract] Fingerprint stream input: "we present RT-Transformer, a novel deep neural network model coupled with graph attention network and 1D-Transformer"
- [abstract] Dataset scale: "by training RT-Transformer on the large small molecule retention time dataset containing 80 038 molecules"
- [readme] Model architecture description: [evidence span withheld — anchor status: external_doc]
- [readme] Dataset source: [evidence span withheld — anchor status: not_found]
- [readme] Input format for custom datasets: [evidence span withheld — anchor status: external_doc]
