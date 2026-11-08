---
name: beam-width-sweep-decoding-evaluation
description: Use when benchmarking inference speed and accuracy trade-offs of an autoregressive
  structure-elucidation model against diffusion-based baselines on the same spectra
  test fold.
when_to_use_negative:
- Evaluating diffusion-based competitors themselves (e.g. DiffMS, FRIGID), which use
  diffusion sampling rather than beam-search multinomial decoding and cannot be swept
  over beam widths
- Comparing only accuracy without latency measurement — the speed-up claim requires
  recording wall-clock time per spectrum at a fixed beam width
- Datasets with restricted access such as NIST23, for which no evaluation data can
  be obtained
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MS-Bart
  role: Autoregressive BART-Base model whose beam-search multinomial sampling decoding
    is swept over beam widths 10–100 during inference on the MassSpecGym test fold
  repo: github.com/OpenDFM/MS-BART
- name: MIST
  role: Pretrained spectral inference model used to convert each test spectrum into
    a 4096-bit Morgan fingerprint representation (threshold 0.2) that serves as the
    model input
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:984e0c77b96b81db77b25d9bd41906d2aaba2c240d184c74da6e151cc58675ea
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MIST
  - MS-Bart
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# beam-width-sweep-decoding-evaluation

## Summary

Sweeps beam-search multinomial sampling widths (10–100) during MS-Bart inference on MassSpecGym spectra to measure Top-1/Top-10 accuracy gains and per-spectrum wall-clock latency, demonstrating that latency scales linearly and MS-Bart (~3 s/spectrum at beam width 100) is roughly one order of magnitude (reported 53x) faster than diffusion-based competitors like DiffMS (~160 s).

## When to use

Use when benchmarking inference speed and accuracy trade-offs of an autoregressive structure-elucidation model against diffusion-based baselines on the same spectra test fold.

## When NOT to use

- Evaluating diffusion-based competitors themselves (e.g. DiffMS, FRIGID), which use diffusion sampling rather than beam-search multinomial decoding and cannot be swept over beam widths
- Comparing only accuracy without latency measurement — the speed-up claim requires recording wall-clock time per spectrum at a fixed beam width
- Datasets with restricted access such as NIST23, for which no evaluation data can be obtained

## Inputs

- Final MS-Bart checkpoint trained on MassSpecGym (from data/MassSpecGym/model-weights in the Figshare release)
- Preprocessed MassSpecGym test fold spectra, fingerprint-tokenized by MIST at threshold 0.2
- NVIDIA A800-SXM4-80GB GPU for the main sweep; RTX 4090-class GPU for consumer-hardware latency measurement
- Reported DiffMS latency (~160 s per spectrum) as the diffusion-based comparison baseline

## Outputs

- Table of Top-1 accuracy, Top-10 accuracy, and wall-clock inference latency at each beam width (10–100)
- Demonstration that latency scales linearly with beam width
- Average per-spectrum inference time at beam width 100 (~3 s on RTX 4090-class hardware)
- Computed speed-up ratio vs DiffMS (~53x, one order of magnitude)

## How to apply

Load the final MS-Bart checkpoint trained on MassSpecGym together with the preprocessed test fold spectra, which are fingerprint-tokenized via MIST at a bit threshold of 0.2. Run beam-search multinomial sampling inference over the full test fold on a GPU (NVIDIA A800-SXM4-80GB), sweeping beam widths from 10 to 100 and recording Top-1 accuracy, Top-10 accuracy, and wall-clock latency at each width. Confirm that latency grows linearly with beam width, then record the average per-spectrum inference time at beam width 100 on a consumer-grade GPU (RTX 4090 class), expected to be about 3 seconds. Compare this measured time against the reported DiffMS latency of roughly 160 seconds per spectrum to compute the speed-up ratio (reported as 53x, i.e. about one order of magnitude). Acceptance requires Top-1/Top-10 accuracies to improve monotonically with beam width while the speed-up remains roughly an order of magnitude.

## Related tools

- **MS-Bart** (Autoregressive BART-Base model whose beam-search multinomial sampling decoding is swept over beam widths 10–100 during inference on the MassSpecGym test fold) — github.com/OpenDFM/MS-BART
- **MIST** (Pretrained spectral inference model used to convert each test spectrum into a 4096-bit Morgan fingerprint representation (threshold 0.2) that serves as the model input)

## Examples

```
python run_inference.py --checkpoint data/MassSpecGym/model-weights --test data/MassSpecGym/test --beam_width 100 --device cuda
```

## Evaluation signals

- Top-1 and Top-10 accuracy improve monotonically as beam width increases from 10 to 100
- Inference latency scales linearly with beam width across the sweep
- Average per-spectrum inference time at beam width 100 is about 3 seconds on an RTX 4090-class GPU
- Computed speed-up vs DiffMS (~160 s/spectrum) is roughly one order of magnitude (paper reports 53x)
- Measured accuracy figures are reproducible against the released model weights from the Figshare MS-BART data release

## Limitations

- NIST23 is excluded from the comparison because access is restricted due to commercial licensing requirements
- MS-Bart depends on the external MIST model for fingerprint prediction; the accuracy of these fingerprint predictions significantly impacts the overall performance
- GPU hardware (A800-SXM4-80GB vs RTX 4090) affects absolute latency measurements, so speed-up ratios are hardware-dependent

## Evidence

- [discussion] MS-Bart is reported to be about one order of magnitude faster than diffusion-based competing methods: "is faster by one order of magnitude than competing diffusion-based method"
- [other] Task workflow records per-spectrum latency at beam width 100 (~3 s) versus DiffMS (~160 s), reported as 53x speed-up: [evidence span withheld — anchor status: not_found]
- [readme] Model weights and preprocessed data are released for reproducibility of the inference sweep: "Download the preprocessed datasets and original model weights from [Figshare](https://figshare.com/articles/dataset/MS-BART-Model-Weights-Data/30393544)"
- [methods] Spectra are represented as Morgan fingerprints tokenized by MIST, defining the model input for the sweep: [evidence span withheld — anchor status: not_found]
