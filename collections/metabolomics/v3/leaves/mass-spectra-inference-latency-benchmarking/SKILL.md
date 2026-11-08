---
name: mass-spectra-inference-latency-benchmarking
description: Use when comparing end-to-end inference cost of a mass-spectra-to-structure
  generative model against diffusion/iterative baselines, or when an acceptance criterion
  specifies a per-spectrum latency and speed-up factor to reproduce.
when_to_use_negative:
- You only need accuracy metrics (Top-1/Top-10) on MassSpecGym or NPLIB1 and have
  no requirement to measure wall-clock inference cost or compare against diffusion
  baselines.
- You are evaluating a retrieval-based or rule-based structure-identification pipeline
  rather than a generative model with beam-search or diffusion sampling, where beam-width
  sweeps are inapplicable.
- You only have a small CPU-only environment; latency numbers would not be comparable
  to the GPU-timed A800/RTX-4090 measurements.
edam_operation: http://edamontology.org/operation_3452
edam_topics:
- http://edamontology.org/topic_3418
tools:
- name: MS-BART
  role: Model under test; final checkpoint is run with beam-search multinomial sampling
    over the test fold while latency and Top-1/Top-10 accuracy are recorded per beam
    width
  repo: https://github.com/OpenDFM/MS-BART
- name: FRIGID
  role: Diffusion-based competitor framework (with MIST fingerprint conditioning)
    whose inference-time scaling results contextualize the diffusion-class latency
    being compared against MS-BART
  repo: https://github.com/coleygroup/FRIGID
- name: MIST
  role: Pretrained spectral inference model used to tokenize spectra into fingerprints
    for the fine-tuned MS-BART inference pipeline under benchmarking
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:1e8e55c18f09951c5cae07b16844c053eb3e9ffd81711b9aed3a61dad03c4a2a
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenDFM/MS-BART
  tools:
  - FRIGID
  - MIST
  - MS-BART
  edam_operation: http://edamontology.org/operation_3452
  edam_topics:
  - http://edamontology.org/topic_3418
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# mass-spectra-inference-latency-benchmarking

## Summary

Benchmarks the wall-clock inference latency of MS-BART structure elucidation against diffusion-based competitors (DiffMS, FRIGID), sweeping beam-search widths 10–100 to show MS-BART is roughly one order of magnitude faster (~3 s vs ~160 s per spectrum).

## When to use

Use when comparing end-to-end inference cost of a mass-spectra-to-structure generative model against diffusion/iterative baselines, or when an acceptance criterion specifies a per-spectrum latency and speed-up factor to reproduce.

## When NOT to use

- You only need accuracy metrics (Top-1/Top-10) on MassSpecGym or NPLIB1 and have no requirement to measure wall-clock inference cost or compare against diffusion baselines.
- You are evaluating a retrieval-based or rule-based structure-identification pipeline rather than a generative model with beam-search or diffusion sampling, where beam-width sweeps are inapplicable.
- You only have a small CPU-only environment; latency numbers would not be comparable to the GPU-timed A800/RTX-4090 measurements.

## Inputs

- Final MS-BART model checkpoint (e.g. data/MassSpecGym/model-weights from the Figshare release)
- Preprocessed MassSpecGym test fold spectra tokenized as 4096-bit Morgan fingerprints at threshold 0.2
- Chemical formula/metadata conditioning used by the inference pipeline
- GPU reference timing environment (e.g. NVIDIA A800-SXM4-80GB)

## Outputs

- Per-beam-width table of Top-1 accuracy, Top-10 accuracy, and wall-clock latency
- Average per-spectrum inference time at beam width 100 (~3 s)
- Speed-up ratio versus DiffMS (~160 s per spectrum; reported as 53x, one order of magnitude)
- Linear-scaling confirmation of latency vs beam width

## How to apply

Load the final MS-BART checkpoint (MassSpecGym or CANOPUS model-weights from the Figshare release) and the preprocessed test fold, whose spectra are fingerprint-tokenized at threshold ϵ=0.2. Run multinomial beam-search inference over the full MassSpecGym test fold on an NVIDIA A800-SXM4-80GB GPU, sweeping beam widths from 10 to 100 and recording Top-1 accuracy, Top-10 accuracy, and wall-clock latency at each width. Verify latency scales linearly with beam width, then record average per-spectrum time at beam width 100 (~3 s) and compare against DiffMS's ~160 s per spectrum to compute the ~53x speed-up (one order of magnitude). Use known-formula setting metrics consistent with the FRIGID benchmark tables.

## Related tools

- **MS-BART** (Model under test; final checkpoint is run with beam-search multinomial sampling over the test fold while latency and Top-1/Top-10 accuracy are recorded per beam width) — https://github.com/OpenDFM/MS-BART
- **FRIGID** (Diffusion-based competitor framework (with MIST fingerprint conditioning) whose inference-time scaling results contextualize the diffusion-class latency being compared against MS-BART) — https://github.com/coleygroup/FRIGID
- **MIST** (Pretrained spectral inference model used to tokenize spectra into fingerprints for the fine-tuned MS-BART inference pipeline under benchmarking)

## Examples

```
python scripts/infer.py --checkpoint data/MassSpecGym/model-weights --test data/MassSpecGym/test --beam-width 100 --sweep 10,20,50,100 --time
```

## Evaluation signals

- Latency at each beam width (10–100) scales approximately linearly with beam width
- Average per-spectrum inference time at beam width 100 is about 3 seconds on the benchmark GPU
- Computed speed-up vs DiffMS (~160 s per spectrum) is roughly one order of magnitude (reported 53x)
- Top-1 and Top-10 accuracies improve monotonically as beam width increases from 10 to 100
- Accuracy values at each beam width match the reported MS-BART baseline numbers on the known-formula benchmark (e.g. 7.45% Top-1 on NPLIB1 in the FRIGID comparison table)

## Limitations

- Measured speed-up is hardware- and dataset-specific (A800-SXM4-80GB for the sweep; RTX 4090-class confirmation); absolute times do not transfer to other environments
- FRIGID's revised results supersede earlier values and its MIST encoders were retrained (MIST 2M-aug / 4M-aug), so diffusion-baseline latency and accuracy must be taken from the same revision for a fair comparison
- Latency depends on fingerprint-prediction quality upstream; MIST prediction accuracy significantly impacts overall performance

## Evidence

- [methods] Beam-search latency sweep protocol: [evidence span withheld — anchor status: not_found]
- [methods] Acceptance criteria for the latency comparison: [evidence span withheld — anchor status: not_found]
- [methods] Linear scaling requirement: [evidence span withheld — anchor status: not_found]
- [readme] FRIGID revised-benchmark caveat: [evidence span withheld — anchor status: not_found]
