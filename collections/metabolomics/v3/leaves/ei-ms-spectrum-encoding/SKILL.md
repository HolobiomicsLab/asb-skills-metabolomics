---
name: ei-ms-spectrum-encoding
description: Use when preprocessing intensity-binned EI-MS peaks (m/z <= 500, <= 300
  peaks) for transformer-based de novo SMILES translation, or when choosing a peak/intensity
  encoding scheme for a spectral translation model.
when_to_use_negative:
- Input is an MS/MS spectrum or requires precursor ion mass or molecular formula cues
  — SpecTUS was trained for GC-EI-MS and does not use those inputs
- Spectrum violates model constraints (m/z above 500, more than 300 peaks, or expected
  SMILES longer than 100 characters) — such data was excluded by the model's filters
- A explainable intermediate representation (peak annotations, fragmentation trees)
  or reference spectra for candidates is required — SpecTUS provides none
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SpecTUS
  role: Encoder-decoder transformer whose encoder sums m/z and intensity embeddings
    per peak and whose decoder generates character-level SMILES
  repo: https://github.com/hejjack/SpecTUS
- name: Hugging Face Hub
  role: Hosts the pretrained SpecTUS checkpoint and 17.2M synthetic pretraining spectra
    (MS-ML organization)
  repo: https://huggingface.co/MS-ML
- name: Jupyter Notebook
  role: Demo inference client (spectus-demo) and the 5_inference_on_open_data.ipynb
    example for translating msp spectra
  repo: https://github.com/ljocha/spectus-demo
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:2a613419b7745cb9e47a6519ba2b3b4bd60978c0b76718d1b4d44eb9bcda2396
  source_papers:
  - doi: 10.48550/arxiv.2502.05114
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hejjack/SpecTUS
  tools:
  - Hugging Face Hub
  - Jupyter Notebook
  - SpecTUS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# ei-ms-spectrum-encoding

## Summary

Encodes a low-resolution GC-EI-MS spectrum for SpecTUS by representing each peak as the sum of its m/z and intensity embeddings, with intensity binning tuned via ablation, and decodes the structure autoregressively as character-level SMILES.

## When to use

Use when preprocessing intensity-binned EI-MS peaks (m/z <= 500, <= 300 peaks) for transformer-based de novo SMILES translation, or when choosing a peak/intensity encoding scheme for a spectral translation model.

## When NOT to use

- Input is an MS/MS spectrum or requires precursor ion mass or molecular formula cues — SpecTUS was trained for GC-EI-MS and does not use those inputs
- Spectrum violates model constraints (m/z above 500, more than 300 peaks, or expected SMILES longer than 100 characters) — such data was excluded by the model's filters
- A explainable intermediate representation (peak annotations, fragmentation trees) or reference spectra for candidates is required — SpecTUS provides none

## Inputs

- Low-resolution GC-EI-MS spectrum (intensity-binned, threshold-filtered), with m/z up to 500 and at most 300 peaks per spectrum
- Spectra in msp format as used by the SpecTUS inference notebook

## Outputs

- Per-peak aggregated encoder embeddings (sum of m/z and intensity embeddings)
- Autoregressively generated, character-level SMILES candidate strings ranked by model certainty

## How to apply

Filter each input spectrum to the model constraints — maximum m/z of 500, at most 300 peaks per spectrum, and target SMILES up to 100 characters; these thresholds excluded ~3% of the NIST dataset. Internally, the encoder compresses each spectrum peak into one aggregated embedding by summing the m/z and intensity embeddings rather than treating peaks as separate tokens. Choose intensity binning carefully: over-simplified schemes (linear binning at 2 decimal places, logarithmic binning with 10 bins) cost 3–4% validation Acc1, whereas logarithmic binning with 30 bins gave a 0.9% Acc1 improvement over 4-decimal-place linear binning while reducing trainable parameters by 10 million. On the output side, the decoder autoregressively generates SMILES from character-level embeddings, which outperformed all BPE tokenizations by 3.5–4% and SELFIES by 1.7–5.8% on validation Acc1, so keep character-level tokenization when reproducing the design.

## Related tools

- **SpecTUS** (Encoder-decoder transformer whose encoder sums m/z and intensity embeddings per peak and whose decoder generates character-level SMILES) — https://github.com/hejjack/SpecTUS
- **Hugging Face Hub** (Hosts the pretrained SpecTUS checkpoint and 17.2M synthetic pretraining spectra (MS-ML organization)) — https://huggingface.co/MS-ML
- **Jupyter Notebook** (Demo inference client (spectus-demo) and the 5_inference_on_open_data.ipynb example for translating msp spectra) — https://github.com/ljocha/spectus-demo

## Examples

```
conda env create -f trainSpectus_build.yaml && jupyter notebook notebooks/5_inference_on_open_data.ipynb
```

## Evaluation signals

- Each input spectrum satisfies the constraints: max m/z 500, at most 300 peaks, SMILES length up to 100 characters
- Validation Acc1 within ~1% of the 4-decimal-place linear binning baseline when using logarithmic binning with 30 bins
- Character-level SMILES decoding yields 3.5–4% higher validation Acc1 than any BPE tokenization variant
- Reference performance on the NIST 20 test split: Acc1 of 43% and Acc10 of 65%

## Limitations

- SpecTUS cannot explicitly justify its outputs for a given input (no explainability of predictions)
- It lacks an intermediate representation such as peak annotations or fragmentation trees, and does not provide reference spectra for candidates
- Performance is sensitive to input spectra quality, shown by poorer results on Cayman and MONA test sets vs. NIST or SWGDRUG

## Evidence

- [supplementary] Encoder aggregation design: [evidence span withheld — anchor status: not_found]
