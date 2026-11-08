---
name: pretrained-gnn-inference
description: Use when building the 4D derivatized sterol lipid database where predicted
  retention times must complement fragment ion and CCS predictions, and molecular
  structures (RDKit-parsable) are available for inference with the pretrained GNN.
when_to_use_negative:
- The goal is CCS prediction for N-Me derived sterols — that uses the QCC-assisted
  LASSO/SVR (scikit-learn) workflow, not the GNN RT predictor
- Input structures cannot be parsed into molecular graphs by RDKit (e.g., non-molecular
  records or corrupted structures)
- You need MS/MS fragment ion prediction — that is the separate RDkit-based N-Me fragmentation
  script in QCCAssisted4DSterol
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: seokhokang/retention_time_gnn
  role: Pretrained GNN implementation run in inference mode to compute predicted chromatographic
    retention times for derivatized sterol structures
  repo: https://github.com/seokhokang/retention_time_gnn/
- name: RDkit
  role: Parses derivatized sterol lipid structures into molecular graph representations
    for GNN input
- name: Chen-micslab/QCCAssisted4DSterol
  role: Parent pipeline whose 'GNN RT prediction' step references the pretrained GNN;
    supplies the 4D sterol database structures being scored
  repo: https://github.com/Chen-micslab/QCCAssisted4DSterol
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/anie.202507483
    title: Quantum Chemistry Calculation‐Assisted Large‐Scale Collision Cross Section
      Prediction Empowers Derivatization‐Enhanced Multidimensional Metabolomics
  source_content_hash: sha256:1b55fb263679909f2b131854c8ca8bc7cf5100fb29ad9a8e43bff5ca012dcdea
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/seokhokang/retention_time_gnn
  tools:
  - Chen-micslab/QCCAssisted4DSterol
  - Jupyter Notebook
  - Python
  - RDkit
  - seokhokang/retention_time_gnn
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# pretrained-gnn-inference

## Summary

Applies the pretrained graph neural network retention time predictor from 'Retention Time Prediction through Learning from a Small Training Data Set with a Pretrained GNN' (seokhokang/retention_time_gnn) to derivatized sterol lipid structures, producing predicted chromatographic retention times that serve as one dimension of the 4D sterol lipid database (n = 4891).

## When to use

Use when building the 4D derivatized sterol lipid database where predicted retention times must complement fragment ion and CCS predictions, and molecular structures (RDKit-parsable) are available for inference with the pretrained GNN.

## When NOT to use

- The goal is CCS prediction for N-Me derived sterols — that uses the QCC-assisted LASSO/SVR (scikit-learn) workflow, not the GNN RT predictor
- Input structures cannot be parsed into molecular graphs by RDKit (e.g., non-molecular records or corrupted structures)
- You need MS/MS fragment ion prediction — that is the separate RDkit-based N-Me fragmentation script in QCCAssisted4DSterol

## Inputs

- Derivatized sterol lipid structures from the 4D sterol lipid database (n = 4891), parsed into RDKit molecular graphs
- Pretrained GNN retention time model (seokhokang/retention_time_gnn, PyTorch/DGL)

## Outputs

- rt_predictions.csv: structured table of per-molecule predicted retention times keyed by sterol lipid identity

## How to apply

Obtain the pretrained GNN RT prediction implementation from github.com/seokhokang/retention_time_gnn/ and set up the Python environment per the referenced Analytical Chemistry method (dependencies: Python, PyTorch, DGL, RDKit). Load the derivatized sterol lipid structures from the 4D sterol database (n = 4891) and parse them into molecular graph representations using RDKit, consistent with the QCCAssisted4DSterol pipeline's RDkit-based structure handling. Run the pretrained GNN in inference mode on each derivatized sterol structure to compute its predicted chromatographic retention time, then assemble per-molecule predictions into a structured table keyed by sterol lipid identity (rt_predictions.csv). No quantitative RT accuracy threshold is reported in this section, so completeness and format serve as the acceptance check.

## Related tools

- **seokhokang/retention_time_gnn** (Pretrained GNN implementation run in inference mode to compute predicted chromatographic retention times for derivatized sterol structures) — https://github.com/seokhokang/retention_time_gnn/
- **RDkit** (Parses derivatized sterol lipid structures into molecular graph representations for GNN input)
- **Chen-micslab/QCCAssisted4DSterol** (Parent pipeline whose 'GNN RT prediction' step references the pretrained GNN; supplies the 4D sterol database structures being scored) — https://github.com/Chen-micslab/QCCAssisted4DSterol

## Examples

```
python run_transfer.py -t FEM_long
```

## Evaluation signals

- rt_predictions.csv exists and is a valid CSV file
- rt_predictions.csv contains a retention time field
- rt_predictions.csv covers the derivatized sterol lipid entries from the 4D database (n = 4891) with no missing identities
- Each prediction is traceable to a RDKit-parsable derivatized sterol structure used as GNN input
- No quantitative RT accuracy threshold is reported for this section, so completeness and format serve as the acceptance check

## Limitations

- No reported quantitative RT accuracy threshold applies to this section; validation relies on output completeness and format rather than an accuracy metric
- The GNN method is designed for learning from a small training dataset via pretrained transfer, so predictions depend on transfer to the derivatized sterol domain (the METLIN-SMRT pretraining and PredRet/MoNA target datasets in the original repo)

## Evidence

- [readme] Pytorch implementation of the model described in the paper [Retention Time Prediction by Learning from Small Training Dataset with Pre-Trained Graph Neural Network]: [evidence span withheld — anchor status: not_found]
