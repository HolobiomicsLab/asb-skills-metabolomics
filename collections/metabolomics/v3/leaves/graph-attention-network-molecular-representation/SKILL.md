---
name: graph-attention-network-molecular-representation
description: Use when predicting liquid chromatography retention times from small-molecule
  structures (e.g. InChI) and you need a hybrid representation that fuses graph-attention
  features with fingerprint/Transformer features, especially when transferring across
  chromatographic methods.
when_to_use_negative:
- The task is not retention time prediction from small-molecule structures — e.g.
  no InChI/graph or fingerprint representation of the molecules is available or relevant
- Data correspond to chromatographic conditions outside the SMRT/PredRet liquid chromatography
  setting and no transfer/fine-tuning step is planned, since the article stresses
  transfer across chromatographic methods rather than general molecular property regression
- Only raw chromatographic peaks or spectra are available without molecular structures,
  since the representation is built from molecular graphs and fingerprints, not from
  MS data
edam_operation: http://edamontology.org/operation_0432
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3399
- http://edamontology.org/topic_3520
tools:
- name: RT-Transformer
  role: Deep neural network combining graph attention network and 1D-Transformer that
    is trained/fine-tuned to predict retention times under any chromatographic method
  repo: https://github.com/01dadada/RT-Transformer
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:c2ff3b4a1233b8450f526605766a1c6aae941d7fb11e2854d070d6956b0e63eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - RT-Transformer
  edam_operation: http://edamontology.org/operation_0432
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3399
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# graph-attention-network-molecular-representation

## Summary

Builds molecular representations for retention time prediction by coupling a graph attention network (GAT) over the molecular graph with a 1D-Transformer over fingerprint features, as implemented in RT-Transformer.

## When to use

Use when predicting liquid chromatography retention times from small-molecule structures (e.g. InChI) and you need a hybrid representation that fuses graph-attention features with fingerprint/Transformer features, especially when transferring across chromatographic methods.

## When NOT to use

- The task is not retention time prediction from small-molecule structures — e.g. no InChI/graph or fingerprint representation of the molecules is available or relevant
- Data correspond to chromatographic conditions outside the SMRT/PredRet liquid chromatography setting and no transfer/fine-tuning step is planned, since the article stresses transfer across chromatographic methods rather than general molecular property regression
- Only raw chromatographic peaks or spectra are available without molecular structures, since the representation is built from molecular graphs and fingerprints, not from MS data

## Inputs

- Small-molecule structures as InChI strings (CSV with 'InChI' and 'RT' columns per the RT-Transformer README)
- Molecular graphs derived from structures (atoms as nodes, bonds as edges) via RDKit/torch_geometric
- Molecular fingerprints paired with the graphs
- Measured retention time values (in seconds) for supervised training/fine-tuning
- Pre-trained RT-Transformer checkpoint (best_state_download_dict.pth or best_state_dict.pth)

## Outputs

- Predicted retention time (seconds) per molecule
- Evaluation metrics on a test split, e.g. MAE and MedAE (RT-Transformer reports MAE 27.30 s / MedAE 12.46 s on retained SMRT test molecules; 33.41 average absolute error with no samples removed)
- Fine-tuned or retrained model checkpoint usable for new chromatographic conditions

## How to apply

Convert each input structure (e.g. an InChI string from a CSV with 'InChI' and 'RT' columns, as in the RT-Transformer repository) into a molecular graph (atoms as nodes, bonds as edges) plus a fingerprint vector using RDKit. Feed the molecular graph through a graph attention network with torch_geometric (torch-scatter, torch-sparse, torch-cluster dependencies) and the fingerprint through a 1D-Transformer, then fuse both branches to output a predicted retention time. Pre-train the combined model on the large SMRT dataset of 80 038 molecules, or start from the released checkpoint best_state_download_dict.pth (trained on retained data) or best_state_dict.pth (trained on full data). Fine-tune on target datasets by placing 'data.csv' at the repository root and running transfer.py for new chromatographic conditions, or train.py to retrain from scratch. Correct application is judged by retention-time error metrics such as MAE/MedAE on a test split (e.g. MAE of 27.30 s and MedAE of 12.46 s on the SMRT test set when training only on retained molecules with RT > 300 s).

## Related tools

- **RT-Transformer** (Deep neural network combining graph attention network and 1D-Transformer that is trained/fine-tuned to predict retention times under any chromatographic method) — https://github.com/01dadada/RT-Transformer

## Examples

```
python ./test.py
```

## Evaluation signals

- Test-set retention time MAE/MedAE within a plausible range relative to the reported benchmark (MAE 27.30 s, MedAE 12.46 s on retained SMRT molecules; 33.41 average absolute error without removing samples)
- Pipeline converts every InChI in data.csv into both a graph (torch_geometric) and a fingerprint (RDKit) without failures, using the README's dependency set (torch, rdkit-pypi, torch-scatter, torch-sparse, torch-cluster, torch_geometric, scikit-learn, pandas, networkx)
- Correct checkpoint is loaded depending on evaluation condition: best_state_download_dict.pth for retained-data evaluation, best_state_dict.pth for full-data evaluation
- transfer.py / train.py runs complete on data.csv with 'InChI' and 'RT' columns and writes a usable model checkpoint, and test.py produces error metrics comparable to paper values
- Filtering condition matches the evaluation: retained molecules only (RT > 300 s) when comparing against the 27.30 s MAE result, full dataset when comparing against 33.41

## Limitations

- The SMRT-derived evaluation depends on the definition of retained molecules — removing not-retained molecules lowers the error from 33.41 to 27.30, so filtering changes comparability
- Different chromatographic conditions yield different retention times for the same metabolite, so models must be transferred/fine-tuned per condition rather than applied blindly
- The GAT/Transformer architecture is built for retention time prediction in liquid chromatography for metabolite annotation; no changelog was found for the RT-Transformer repository, and training requires specific dependencies (Python 3.9, torch_geometric stack)
- Performance on 41 external datasets was reported for liquid chromatography retention times; behavior outside that modality is not demonstrated in the article

## Evidence

- [readme] Architecture: [evidence span withheld — anchor status: external_doc]
- [abstract] Abstract: "we present RT-Transformer, a novel deep neural network model coupled with graph attention network and 1D-Transformer"
- [abstract] Dataset size: "by training RT-Transformer on the large small molecule retention time dataset containing 80 038 molecules"
- [abstract] Benchmark metrics: "the average absolute error reached 27.30 after removing not retained molecules. Still, it reached 33.41 when no samples were removed"
- [readme] Input format: [evidence span withheld — anchor status: not_found]
- [readme] Pretrained checkpoints: [evidence span withheld — anchor status: external_doc]
