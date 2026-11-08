---
name: molecular-graph-representation-learning
description: Use when predicting liquid chromatography retention times for metabolites
  from SMILES/InChI structures, especially when a pretrained model must be transferred
  to a new chromatographic condition or evaluated on the SMRT benchmark.
when_to_use_negative:
- Input is a numeric molecular descriptor table or fingerprint matrix rather than
  a structure from which a molecular graph and SMILES sequence can be built.
- Retention times come from gas chromatography or a technique not represented in the
  SMRT/PredRet liquid chromatography training data.
- The task is de novo metabolite identification rather than retention time prediction/scoring
  of candidate annotations.
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_3334
- http://edamontology.org/topic_0091
tools:
- name: RT-Transformer
  role: 'Core model: graph attention network + 1D-Transformer architecture that consumes
    molecular graphs and tokenized SMILES to output retention times'
  repo: https://github.com/01dadada/RT-Transformer
- name: RDKit (rdkit-pypi)
  role: Converts input InChI/SMILES strings into molecular graph representations used
    by the graph attention network
- name: PyTorch Geometric (torch_geometric)
  role: Handles graph data structures, scatter/sparse operations, and clustering needed
    to build and batch molecular graphs
- name: Hugging Face Spaces
  role: Hosts a web server for RT-Transformer predictions and the downloadable pretrained
    model files
  repo: https://huggingface.co/spaces/Xue-Jun/RT-Transformer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:67d5f707b3089516067d70ee1e16b6b8b48445b1db1b0d8784d22d37631d097e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - Hugging Face Spaces
  - PyTorch Geometric (torch_geometric)
  - RDKit (rdkit-pypi)
  - RT-Transformer
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_3334
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# molecular-graph-representation-learning

## Summary

A dual-representation deep learning approach that couples a graph attention network over molecular graphs with a 1D-Transformer over tokenized SMILES to predict chromatographic retention times, pretrained on the 80,038-molecule SMRT dataset and transferable across chromatographic methods.

## When to use

Use when predicting liquid chromatography retention times for metabolites from SMILES/InChI structures, especially when a pretrained model must be transferred to a new chromatographic condition or evaluated on the SMRT benchmark.

## When NOT to use

- Input is a numeric molecular descriptor table or fingerprint matrix rather than a structure from which a molecular graph and SMILES sequence can be built.
- Retention times come from gas chromatography or a technique not represented in the SMRT/PredRet liquid chromatography training data.
- The task is de novo metabolite identification rather than retention time prediction/scoring of candidate annotations.

## Inputs

- CSV with "InChI" and "RT" columns (molecular structures and experimental retention times)
- SMRT dataset: small molecule retention time dataset with 80,038 molecules
- PredRet datasets for transfer learning across chromatographic conditions
- Molecular graphs (via RDKit/torch_geometric) and tokenized SMILES sequences derived from the input structures

## Outputs

- Predicted retention times (in seconds) for each input molecule
- Fine-tuned or pretrained model checkpoints (best_state_dict.pth, best_state_download_dict.pth)
- Validation metrics: MAE and MedAE on the test split

## How to apply

Load the small molecule retention time dataset (80,038 molecules) as a CSV with InChI and RT columns; for transfer learning rename it to data.csv at the repository root. Convert each molecule into the two representations RT-Transformer requires: a molecular graph (atoms as nodes, bonds as edges, processed via torch_geometric) for the graph attention network, and a tokenized SMILES sequence for the 1D-Transformer. Train with train.py, or load the pretrained checkpoint best_state_dict.pth (full data, MAE 33.41 s / MedAE 13.05 s) or best_state_download_dict.pth (retained-only, MAE 27.30 s) and fine-tune with transfer.py on a new chromatographic-condition dataset from PredRet. Run test.py for validation/inference, or follow QuickStart.ipynb to predict retention times on your own molecules. Evaluate by computing MAE and MedAE on the test split; correct application reproduces the reported 33.41 s MAE (no samples removed) or 27.30 s (not-retained molecules removed).

## Related tools

- **RT-Transformer** (Core model: graph attention network + 1D-Transformer architecture that consumes molecular graphs and tokenized SMILES to output retention times) — https://github.com/01dadada/RT-Transformer
- **RDKit (rdkit-pypi)** (Converts input InChI/SMILES strings into molecular graph representations used by the graph attention network)
- **PyTorch Geometric (torch_geometric)** (Handles graph data structures, scatter/sparse operations, and clustering needed to build and batch molecular graphs)
- **Hugging Face Spaces** (Hosts a web server for RT-Transformer predictions and the downloadable pretrained model files) — https://huggingface.co/spaces/Xue-Jun/RT-Transformer

## Examples

```
python test.py  # validate pretrained RT-Transformer on SMRT; or: python transfer.py  # fine-tune best_state_dict.pth on data.csv (InChI, RT columns) from a new chromatographic condition
```

## Evaluation signals

- Test-set MAE on the full SMRT dataset (no samples removed) should reproduce 33.41 s (MedAE 13.05 s).
- Test-set MAE with not-retained molecules removed should reproduce 27.30 s.
- Fine-tuned model should show competitive performance on the 5 PredRet datasets versus state-of-the-art retention-time prediction methods.
- Predictions must fall within chemically plausible retention-time ranges for the given chromatographic method.
- Per-molecule predictions should align with experimental RT values when compared molecule-by-molecule (e.g. via QuickStart.ipynb output).

## Limitations

- Pre-training was on the 80,038-molecule SMRT dataset; performance on chromatographic conditions absent from SMRT/PredRet is unvalidated.
- Removing not-retained molecules from training improves MAE from 33.41 to 27.30, so the full-data checkpoint carries the penalty of not-retained molecules.
- No changelog is available for the 01dadada/RT-Transformer repository, making version-dependent behavior hard to track.
- Different chromatographic conditions yield different retention times for the same metabolite; the model relies on transfer learning from a pre-trained model rather than condition-specific training data.
- Evaluation on 41 external datasets was reported for liquid chromatography; no evidence is given for other separation techniques.

## Evidence

- [abstract] Dual-representation architecture claim: "we present RT-Transformer, a novel deep neural network model coupled with graph attention network and 1D-Transformer, which can predict retention times under any chromatographic methods"
- [abstract] Pre-training dataset size: "by training RT-Transformer on the large small molecule retention time dataset containing 80 038 molecules"
- [abstract] Benchmark MAE results: "the average absolute error reached 27.30 after removing not retained molecules. Still, it reached 33.41 when no samples were removed"
- [abstract] Transfer learning workflow: "First, we obtain a pre-trained model by training RT-Transformer on the large small molecule retention time dataset containing 80 038 molecules, and then transfer the resulting model to different"
- [readme] README usage instructions: [evidence span withheld — anchor status: external_doc]
- [readme] README training prerequisites: [evidence span withheld — anchor status: external_doc]
- [readme] README pretrained model files: [evidence span withheld — anchor status: external_doc]
- [readme] SMRT dataset source: [evidence span withheld — anchor status: external_doc]
