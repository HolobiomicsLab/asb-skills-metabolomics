---
name: ei-ms-spectral-transformer-inference
description: Use when you have GC-EI mass spectra (e.g. from an msp file or synthetic
  NEIMS/RASSP test splits) and need de novo molecular structure reconstruction via
  the pretrained SpecTUS model without reference spectral libraries.
when_to_use_negative:
- You need reference-library matching or spectral similarity search against known
  EI-MS databases (e.g. hybrid search) rather than de novo structure prediction.
- The task is finetuning or training SpecTUS itself on NIST20 — this skill covers
  inference with the released pretrained checkpoint only (the final NIST20-finetuned
  model is not shareable).
- 'Spectra exceed model thresholds: max m/z 500, max 300 peaks per spectrum, SMILES
  > 100 characters.'
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: SpecTUS
  role: Transformer model whose pretrained checkpoint performs autoregressive inference,
    translating EI-MS spectra into ranked SMILES candidates
  repo: https://github.com/hejjack/SpecTUS
- name: conda
  role: Environment setup via the trainSpectus_build.yaml specification
  repo: https://github.com/hejjack/SpecTUS
- name: MS-ML (Hugging Face Hub)
  role: Hosts the pretrained SpecTUS checkpoint and the 17.2M synthetic pretraining
    spectra used as inference inputs
  repo: https://huggingface.co/MS-ML
- name: ljocha/spectus-demo
  role: Binder-ready Jupyter notebook client for demo inference against the SpecTUS
    model served via REST API
  repo: https://github.com/ljocha/spectus-demo
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:cd8492e27a956b91858d9de5761de8ce2fecd3538343fc8a17252369543669c7
  source_papers:
  - doi: 10.48550/arxiv.2502.05114
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hejjack/SpecTUS
  tools:
  - MS-ML (Hugging Face Hub)
  - NEIMS
  - SpecTUS
  - conda
  - ljocha/spectus-demo
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# ei-ms-spectral-transformer-inference

## Summary

Runs autoregressive inference with the pretrained SpecTUS transformer checkpoint to translate GC-EI-MS spectra directly into ranked SMILES candidate structures, producing top-1/top-10 predictions evaluated by exact reconstruction (Acc1/Acc10) and Morgan-fingerprint Tanimoto similarity (Sim1/Sim10).

## When to use

Use when you have GC-EI mass spectra (e.g. from an msp file or synthetic NEIMS/RASSP test splits) and need de novo molecular structure reconstruction via the pretrained SpecTUS model without reference spectral libraries.

## When NOT to use

- You need reference-library matching or spectral similarity search against known EI-MS databases (e.g. hybrid search) rather than de novo structure prediction.
- The task is finetuning or training SpecTUS itself on NIST20 — this skill covers inference with the released pretrained checkpoint only (the final NIST20-finetuned model is not shareable).
- Spectra exceed model thresholds: max m/z 500, max 300 peaks per spectrum, SMILES > 100 characters.

## Inputs

- GC-EI mass spectra (msp file or synthetic spectra from the NEIMS- and RASSP-generated halves of synth1/synth2, held-out 5% test split)
- Pretrained SpecTUS checkpoint from Hugging Face Hub (MS-ML)
- Ground-truth SMILES structures for the queried spectra

## Outputs

- Ranked SMILES candidate structures (top-1 and top-10) per input spectrum
- synthetic_test_metrics.csv with rows per generator (NEIMS, RASSP) and columns Acc1, Acc10, Sim1, Sim10

## How to apply

Create the environment with `conda env create -f trainSpectus_build.yaml`, then download the pretrained checkpoint and the synthetic datasets (synth1, synth2, NEIMS- and RASSP-generated halves) from the Hugging Face Hub at MS-ML. Select the held-out 5% test partition (per the 0.9:0.05:0.05 train/val/test split) and preprocess spectra with SpecTUS's intensity-binning scheme — logarithmic binning with 30 bins is the paper's optimal choice — subject to model thresholds (max 300 peaks, max m/z 500, SMILES length ≤ 100). Run autoregressive decoding to produce 1 and 10 ranked SMILES candidates per spectrum. Score Acc1/Acc10 as the proportion of spectra whose ground-truth structure appears exactly in the top-k predictions, and Sim1/Sim10 as the per-query maximum Morgan-fingerprint Tanimoto similarity averaged over all queries, evaluated separately for the NEIMS-generated and RASSP-generated halves. Aggregate rows per generator (NEIMS, RASSP) into synthetic_test_metrics.csv with columns Acc1, Acc10, Sim1, Sim10.

## Related tools

- **SpecTUS** (Transformer model whose pretrained checkpoint performs autoregressive inference, translating EI-MS spectra into ranked SMILES candidates) — https://github.com/hejjack/SpecTUS
- **conda** (Environment setup via the trainSpectus_build.yaml specification) — https://github.com/hejjack/SpecTUS
- **MS-ML (Hugging Face Hub)** (Hosts the pretrained SpecTUS checkpoint and the 17.2M synthetic pretraining spectra used as inference inputs) — https://huggingface.co/MS-ML
- **ljocha/spectus-demo** (Binder-ready Jupyter notebook client for demo inference against the SpecTUS model served via REST API) — https://github.com/ljocha/spectus-demo

## Examples

```
git clone https://github.com/hejjack/SpecTUS.git; cd SpecTUS; conda env create -f trainSpectus_build.yaml
```

## Evaluation signals

- Reproduced Acc1 on the synthetic held-out test split should match the paper's end-of-pretraining exact-reconstruction rates: 38% for RASSP-generated and 29% for NEIMS-generated spectra (3% for NIST spectra).
- Metrics must follow the paper's definitions: Acc_k = ground truth appearing exactly among the top-k predictions; Sim_k = highest Morgan-fingerprint Tanimoto similarity per query, averaged over queries.
- synthetic_test_metrics.csv must contain one row per generator (NEIMS, RASSP) with columns Acc1, Acc10, Sim1, Sim10.
- Input spectra must conform to SpecTUS preprocessing: logarithmic intensity binning with 30 bins, max 300 peaks, max m/z 500, SMILES ≤ 100 characters.

## Limitations

- SpecTUS cannot explicitly justify its specific outputs for a given input (no explainability of predictions).
- It lacks intermediate representations such as peak annotations or fragmentation trees, and does not provide reference spectra for candidates, limiting manual correction.
- Performance is sensitive to input spectra quality — the model performed worse on Cayman and MONA test sets than on the rigorously curated NIST or SWGDRUG libraries.
- The final NIST20-finetuned model cannot be shared (NIST20 is proprietary); only the pretrained checkpoint is freely available, and its exact-structure reconstruction on synthetic spectra is relatively low (38% RASSP / 29% NEIMS Acc1).

## Evidence
