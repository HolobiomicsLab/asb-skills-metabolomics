---
name: retention-time-prediction-inference
description: Use when you have a molecular formula (e.g. from PubChem PUG REST) with
  multiple isomeric candidate SMILES and one observed retention time under an SMRT-type
  chromatographic method, and need to prune inconsistent candidates.
when_to_use_negative:
- The observed retention time is already known for every candidate structure, so RT-based
  filtering would remove nothing
- The chromatographic method differs from the SMRT conditions the pre-trained model
  was trained on and no InChI/RT dataset is available to fine-tune via transfer.py
- Only a bare molecular formula is available with no candidate SMILES/InChI structures,
  since the model requires molecular graph and fingerprint input
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_0154
tools:
- name: RT-Transformer
  role: Predicts retention time for each candidate structure from combined fingerprint
    and molecular graph features; predictions are used to eliminate candidates inconsistent
    with the observed RT
  repo: https://github.com/01dadada/RT-Transformer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:f7803ccf4f67432adad59fc1b9cea7b8848ddc3b4a8395dd19927c53d1705770
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - RT-Transformer
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# retention-time-prediction-inference

## Summary

Uses RT-Transformer, a graph attention network + 1D-Transformer model that combines molecular fingerprint and molecular graph features, to predict LC retention times under a given chromatographic method and thereby eliminate candidate metabolite structures whose predicted RT is inconsistent with the observed RT.

## When to use

Use when you have a molecular formula (e.g. from PubChem PUG REST) with multiple isomeric candidate SMILES and one observed retention time under an SMRT-type chromatographic method, and need to prune inconsistent candidates.

## When NOT to use

- The observed retention time is already known for every candidate structure, so RT-based filtering would remove nothing
- The chromatographic method differs from the SMRT conditions the pre-trained model was trained on and no InChI/RT dataset is available to fine-tune via transfer.py
- Only a bare molecular formula is available with no candidate SMILES/InChI structures, since the model requires molecular graph and fingerprint input

## Inputs

- Molecular formula of the unknown metabolite and its observed LC retention time (e.g. from a SMRT test molecule)
- Isomeric candidate SMILES/structures retrieved from PubChem PUG REST by exact formula
- Pre-trained RT-Transformer weight file (best_state_download_dict.pth or best_state_dict.pth)
- Optional fine-tuning set as a CSV with 'InChI' and 'RT' columns (data.csv) for a different chromatographic method

## Outputs

- Predicted retention time for each candidate isomeric structure
- Per-molecule candidate counts and filter rate (fraction of candidates eliminated by the RT filter)
- Aggregate results table, e.g. candidate_filter_results.csv with mean filter rate across molecules

## How to apply

Retrieve all isomeric candidate structures from PubChem by exact molecular formula, then convert each candidate's isomeric SMILES into the molecular-graph and fingerprint features that RT-Transformer's graph attention network + 1D-Transformer consumes. Load the pre-trained weights appropriate to your data ('best_state_download_dict.pth' for retained-only data, 'best_state_dict.pth' for full data) and follow the repo's QuickStart.ipynb to predict retention time for each candidate. Discard candidates whose predicted RT is inconsistent with the observed RT of the unknown compound and record the fraction removed per molecule. The pre-trained model was trained on the SMRT-derived small molecule retention time dataset (80 038 molecules) and reaches an average absolute error of 27.30 after removing not retained molecules (33.41 with none removed), so treat predictions with errors at roughly that scale when setting the consistency window.

## Related tools

- **RT-Transformer** (Predicts retention time for each candidate structure from combined fingerprint and molecular graph features; predictions are used to eliminate candidates inconsistent with the observed RT) — https://github.com/01dadada/RT-Transformer

## Examples

```
python ./test.py
```

## Evaluation signals

- Per-molecule candidate count before filtering exceeds the count after filtering, and filter rates lie in [0, 1]
- Mean filter rate across the 100 randomly selected SMRT test molecules should be reproducible near the reported 59.35%
- Predicted RTs should fall within the SMRT retention-time range, with average absolute error near the reported 27.30 (retained data) or 33.41 (full data)
- Every candidate SMILES parses in RDKit and yields a valid molecular graph before the forward pass
- Output CSV contains per-molecule candidate counts, filter rates, and an overall mean without missing rows

## Limitations

- The model is chromatographic-method-specific: different chromatographic conditions give different RTs for the same metabolite, so a new method requires transfer learning (transfer.py) on a data.csv with InChI and RT columns
- Best retained-data model ('best_state_download_dict.pth') was trained after removing not-retained molecules, so it may mispredict compounds outside the retained regime; using full-data weights raises AAE to 33.41
- Pre-training covered 80 038 molecules from the small molecule retention time dataset, so candidates far outside that chemical space may be unreliably scored
- No changelog exists for the 01dadada/RT-Transformer repository, so code/weight versions are hard to track

## Evidence

- [abstract] graph attention network and 1D-Transformer: "we present RT-Transformer, a novel deep neural network model coupled with graph attention network and 1D-Transformer, which can predict retention times under any chromatographic methods"
- [abstract] pre-trained on SMRT dataset: "First, we obtain a pre-trained model by training RT-Transformer on the large small molecule retention time dataset containing 80 038 molecules"
- [abstract] AAE 27.30: "the average absolute error reached 27.30 after removing not retained molecules. Still, it reached 33.41 when no samples were removed"
- [readme] architecture combines fingerprint and graph: [evidence span withheld — anchor status: external_doc]
- [readme] QuickStart for prediction on own data: [evidence span withheld — anchor status: external_doc]
- [readme] transfer learning input format: [evidence span withheld — anchor status: external_doc]
