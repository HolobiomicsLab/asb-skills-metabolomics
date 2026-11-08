---
name: de-novo-generation-metric-computation
description: Use when scoring de novo molecule generation outputs from mass spectra
  (NPLIB1 or MassSpecGym spectra-to-molecule predictions) to produce accuracy, MCES-distance,
  and Tanimoto-similarity tables.
when_to_use_negative:
- Input is a single retrieved structure per spectrum rather than a sample of 100 candidates
  — the frequency-based top-k ranking is undefined without multiple samples
- Benchmarking tasks that historically use NIST20 or NIST23 reference libraries rather
  than de novo generation splits like NPLIB1
- Spectrum-to-fingerprint or formula-prediction evaluation (MIST encoder performance),
  which uses fingerprint prediction metrics rather than structure-level accuracy/MCES/Tanimoto
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3474
tools:
- name: RDKit
  role: Parse sampled structures, drop invalid or disconnected molecules, and compute
    similarity-related quantities during metric computation
- name: DiffMS
  role: Generative model whose per-spectrum samples (100 per spectrum) are the input
    to the metric computation
  repo: github.com/coleygroup/DiffMS
- name: PyTorch
  role: Runtime for the diffusion decoder whose generated candidates are scored
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:f96480fec7967c41470856e66ad3dc660e87d9115ecc9f21a4d5c4c82a7977ed
  source_papers:
  - doi: 10.48550/arxiv.2502.09571
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - COCONUT
  - DSSTox
  - DiffMS
  - HMDB
  - MIST
  - MOSES
  - NPLIB1
  - PyTorch
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.84
n_claims: 25
---

# De novo generation metric computation

## Summary

Evaluation protocol for de novo molecular structure generation from mass spectra: sample 100 molecules per spectrum from the generative decoder, remove invalid or disconnected molecules with RDKit, rank top-k candidates by frequency, and report top-1/top-10 accuracy, minimum MCES distance, and maximum Tanimoto similarity against ground-truth structures.

## When to use

Use when scoring de novo molecule generation outputs from mass spectra (NPLIB1 or MassSpecGym spectra-to-molecule predictions) to produce accuracy, MCES-distance, and Tanimoto-similarity tables.

## When NOT to use

- Input is a single retrieved structure per spectrum rather than a sample of 100 candidates — the frequency-based top-k ranking is undefined without multiple samples
- Benchmarking tasks that historically use NIST20 or NIST23 reference libraries rather than de novo generation splits like NPLIB1
- Spectrum-to-fingerprint or formula-prediction evaluation (MIST encoder performance), which uses fingerprint prediction metrics rather than structure-level accuracy/MCES/Tanimoto

## Inputs

- Sampled molecule set per spectrum (100 molecules per spectrum from the diffusion decoder, e.g. SMILES or molecular graphs)
- Ground-truth reference structures for the benchmark test set (NPLIB1 or MassSpecGym)
- Mass spectra with associated chemical formulas from the finetuned end-to-end model

## Outputs

- Top-1 accuracy (%) on NPLIB1 / MassSpecGym
- Top-10 accuracy (%) on NPLIB1 / MassSpecGym
- Minimum MCES distance per benchmark
- Maximum Tanimoto similarity per benchmark
- Top-k molecule rankings by per-spectrum sampling frequency

## How to apply

For each test spectrum, sample 100 molecules from the diffusion decoder (DiffMS) conditioned on the spectrum and its inferred formula. Parse each sampled structure with RDKit (rdkit=2024.09.4 in the diffms conda env) and discard molecules that are invalid or disconnected (multiple GetMolFrags). Rank the remaining molecules by sampling frequency to identify the top-k molecules per spectrum. Compute top-1 and top-10 accuracy against the ground-truth molecule, the minimum MCES distance between ranked candidates and the ground truth, and the maximum Tanimoto similarity of the candidates to the ground truth. Run this protocol on the established de novo benchmarks NPLIB1 and MassSpecGym so results are comparable to DiffMS's reported tables.

## Related tools

- **RDKit** (Parse sampled structures, drop invalid or disconnected molecules, and compute similarity-related quantities during metric computation)
- **DiffMS** (Generative model whose per-spectrum samples (100 per spectrum) are the input to the metric computation) — github.com/coleygroup/DiffMS
- **PyTorch** (Runtime for the diffusion decoder whose generated candidates are scored)

## Examples

```
from rdkit import Chem; valid=[Chem.MolToSmiles(m) for s in sampled_smis for m in [Chem.MolFromSmiles(s)] if m and len(Chem.GetMolFrags(m))==1]
```

## Evaluation signals

- Every retained molecule per spectrum passes RDKit MolFromSmiles/validity check and consists of exactly one connected fragment (GetMolFrags length 1)
- On NPLIB1 with the full 2.8M-pair pretrained decoder, the protocol should reproduce approximately 8.34% top-1 accuracy and 15.44% top-10 accuracy as reported for DiffMS
- Metrics are computed after the 100-sample-per-spectrum protocol; top-k rankings must be based on sampling frequency, not per-sample likelihood
- Results are reported on both common open-source de novo benchmarks, NPLIB1 and MassSpecGym, enabling direct comparison to DiffMS tables
- MCES distance should decrease and Tanimoto similarity should increase as pretraining dataset size grows, consistent with the reported scaling trend

## Limitations

- Exact molecular structure recovery from de novo generation remains challenging; structurally close matches (measured via MCES distance and Tanimoto similarity) are the realistic outcome
- Metric values depend on sampling 100 molecules per spectrum and the frequency-based top-k ranking; changing the sample count or ranking rule changes reported accuracies
- Results are tied to the specific benchmark splits (NPLIB1, MassSpecGym); other NIST20/NIST23-style splits are not directly comparable

## Evidence

- [methods] Sampling 100 molecules per spectrum and ranking by frequency: [evidence span withheld — anchor status: not_found]
- [methods] Benchmarks used for de novo generation evaluation: [evidence span withheld — anchor status: not_found]
- [abstract] Benchmark-level performance claim: "Extensive experiments on established benchmarks show that DiffMS outperforms existing models on de novo molecule generation."
- [readme] RDKit version used for molecule validity and metric computation: "conda create -y -c conda-forge -n diffms rdkit=2024.09.4 python=3.9"
- [abstract] Scaling trend reproduced by the metric protocol: "show consistent performance scaling with increasing pretraining dataset size"
