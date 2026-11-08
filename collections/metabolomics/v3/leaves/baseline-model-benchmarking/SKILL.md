---
name: baseline-model-benchmarking
description: Use when a new predictive model (e.g., SigmaCCS for CCS prediction) must
  be validated against a published baseline (e.g., DeepCCS) on the same curated test
  set to confirm the reported performance gap.
when_to_use_negative:
- The baseline model cannot run on any of the test molecules (e.g., SMILES strings
  contain unknown chemical symbols or exceed 250 characters), making the comparison
  empty
- The goal is training or hyperparameter tuning rather than independent evaluation
  on a held-out test set
- Predictions target adduct ion types outside the three encoded types ([M+H]+, [M-H]-,
  [M+Na]+), so neither model can be meaningfully compared
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: SigmaCCS
  role: Proposed GNN-based CCS prediction model evaluated in Isevaluate mode on the
    full 559-molecule test set
  repo: https://github.com/zmzhang/SigmaCCS
- name: DeepCCS
  role: CNN baseline that predicts CCS from one-hot encoded SMILES, run on the 514
    test molecules it supports
- name: RDKit
  role: Generates one 3D conformer per molecule via ETKDGv3 (numConfs=1) and optimizes
    it with MMFF before SigmaCCS inference
- name: TensorFlow
  role: Runtime for loading and executing the trained SigmaCCS model (model.h5)
  repo: https://www.tensorflow.org
- name: Spektral
  role: Graph neural network library used by SigmaCCS for graph-structured model inputs
  repo: https://graphneural.network/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42004-023-00939-w
    title: Highly accurate and large-scale collision cross sections prediction with
      graph neural networks
  source_content_hash: sha256:4f05d76f79cc94cc230db2308b5e46680499d9fac6d0a729d4956ccf2453ec16
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmzhang/SigmaCCS
  tools:
  - DeepCCS
  - RDKit
  - SigmaCCS
  - Spektral
  - TensorFlow
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# baseline-model-benchmarking

## Summary

Head-to-head evaluation of a proposed model against a published baseline on identical held-out data, computing the same metrics (R2 and median relative error) for both and comparing them to reported reference values.

## When to use

Use when a new predictive model (e.g., SigmaCCS for CCS prediction) must be validated against a published baseline (e.g., DeepCCS) on the same curated test set to confirm the reported performance gap.

## When NOT to use

- The baseline model cannot run on any of the test molecules (e.g., SMILES strings contain unknown chemical symbols or exceed 250 characters), making the comparison empty
- The goal is training or hyperparameter tuning rather than independent evaluation on a held-out test set
- Predictions target adduct ion types outside the three encoded types ([M+H]+, [M-H]-, [M+Na]+), so neither model can be meaningfully compared

## Inputs

- TestData.csv — held-out test set of 559 molecules with SMILES strings, adduct types ([M+H]+, [M-H]-, [M+Na]+), and experimental CCS values
- Trained model artifacts: parameter.pkl and model.h5 (SigmaCCS)
- DeepCCS baseline model supporting one-hot encoded SMILES (length <=250 characters)

## Outputs

- TestData-pred.csv — 559 SigmaCCS predicted CCS values
- evaluation_metrics.csv — comparison table of R2 and median RE for SigmaCCS vs DeepCCS

## How to apply

Load the held-out test set (TestData.csv, 559 molecules with SMILES, adduct types [M+H]+, [M-H]-, [M+Na]+, and experimental CCS values) and run the proposed model in evaluation mode (SigmaCCS Model_prediction with Isevaluate, loading stored parameter.pkl and model.h5), generating one 3D conformer per molecule with RDKit ETKDGv3 (numConfs=1) and MMFF optimization. Then run the baseline (DeepCCS) only on the subset it supports — 514 of 559 molecules with one-hot encoded SMILES of length <=250 characters and no unknown chemical symbols. Compute R2 and median relative error (RE) for each model against the experimental CCS values, and compare the computed metrics with the reported references (SigmaCCS R2=0.9945, median RE=1.1751%; DeepCCS R2=0.9794, median RE=2.403%, Supplementary Table 6), saving the comparison as evaluation_metrics.csv. Note that models are compared on different effective sample sizes when the baseline cannot predict all molecules, which is itself a coverage finding.

## Related tools

- **SigmaCCS** (Proposed GNN-based CCS prediction model evaluated in Isevaluate mode on the full 559-molecule test set) — https://github.com/zmzhang/SigmaCCS
- **DeepCCS** (CNN baseline that predicts CCS from one-hot encoded SMILES, run on the 514 test molecules it supports)
- **RDKit** (Generates one 3D conformer per molecule via ETKDGv3 (numConfs=1) and optimizes it with MMFF before SigmaCCS inference)
- **TensorFlow** (Runtime for loading and executing the trained SigmaCCS model (model.h5)) — https://www.tensorflow.org
- **Spektral** (Graph neural network library used by SigmaCCS for graph-structured model inputs) — https://graphneural.network/

## Examples

```
conda activate sigma; python Model_prediction.py --mode Isevaluate --model model/model.h5 --param parameter/parameter.pkl --input data/TestData.csv --output data/TestData-pred.csv
```

## Evaluation signals

- TestData-pred.csv exists and contains exactly 559 SigmaCCS predictions
- Computed SigmaCCS R2 reproduces the reported 0.9945 and median relative error reproduces 1.1751%
- DeepCCS is evaluated only on its 514 predictable molecules and reproduces R2=0.9794 and median RE=2.403% (Supplementary Table 6)
- The comparison table evaluation_metrics.csv contains both models' metrics alongside the reported reference values
- Conformer generation uses numConfs=1 with ETKDGv3 and MMFF optimization, matching the reported inference procedure

## Limitations

- DeepCCS can only predict 514 of the 559 test-set molecules (SMILES length limited to 250 characters; fails on unknown chemical symbols), so metrics are not computed on identical subsets
- SigmaCCS cannot predict molecules containing elements other than C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se
- ETKDG conformer generation can fail for some molecules (33 of 94,161,896 PubChem molecules and 5 of 5,645 CCSbase entries), and 662 molecules' conformers could not be minimized by MMFF94

## Evidence

- [abstract] It achieved a coefficient of determination of 0.9945 and a median relative error of 1.1751% on the test set: "It achieved a coefficient of determination of 0.9945 and a median relative error of 1.1751% on the test set"
- [discussion] Only 514 of 559 test-set molecules predictable by DeepCCS: "There are 559 molecules in the test set. Only 514 molecules can be predicted by DeepCCS (Supplementary Table 6)."
- [discussion] DeepCCS one-hot encoding limitations: "DeepCCS can not perform a prediction when a SMILES string contains unknown chemical symbols."
- [methods] Graph construction via RDKit conformer generation: "re = AllChem.EmbedMultipleConfs(mol, numConfs = 1, params = ps)"
- [readme] SigmaCCS repo data files: "TestData.csv
    - TestData-pred.csv (Predicted results)"
