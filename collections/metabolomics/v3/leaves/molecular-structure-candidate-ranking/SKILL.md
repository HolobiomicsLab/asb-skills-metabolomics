---
name: molecular-structure-candidate-ranking
description: Use when a preprocessed low-resolution GC-EI-MS spectrum (intensity-binned,
  threshold-filtered, m/z ≤ 500, ≤ 300 peaks) must be translated de novo into a ranked
  list of candidate SMILES structures without a reference database.
when_to_use_negative:
- Input is an MS/MS (tandem) spectrum rather than a GC-EI-MS spectrum — the model
  was trained specifically on EI-MS data and comparisons in the paper note other de
  novo models were built for MS/MS
- Spectra violate model constraints (m/z > 500, more than 300 peaks, deuterium-containing
  compounds, expected SMILES longer than 100 characters)
- A rigorously curated reference library (e.g. NIST or SWGDRUG) is available and a
  database search would suffice — SpecTUS is aimed at novel compounds absent from
  spectral libraries
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SpecTUS
  role: Encoder-decoder transformer (354M parameters) that autoregressively decodes
    ranked SMILES candidates from the binned EI-MS spectrum via beam search
  repo: https://github.com/hejjack/SpecTUS
- name: Jupyter Notebook
  role: Demo inference notebook (spectus-demo) that sends spectra to the SpecTUS service
    to obtain ranked SMILES predictions
  repo: https://github.com/ljocha/spectus-demo
- name: Hugging Face Hub
  role: Hosts the 17.2M synthetic pretraining spectra and the pretrained SpecTUS checkpoint
    used for inference
  repo: https://huggingface.co/MS-ML
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:f6d223e5648d56fc85d20b8cffeeef4cf19f75322e10097afedfbc7034e36b3b
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# molecular-structure-candidate-ranking

## Summary

Rank candidate molecular structures for an EI-MS spectrum using SpecTUS, a transformer encoder-decoder that de novo translates GC-EI-MS spectra into SMILES and orders candidates by the model's inherent certainty, enabling annotation of compounds absent from spectral libraries.

## When to use

Use when a preprocessed low-resolution GC-EI-MS spectrum (intensity-binned, threshold-filtered, m/z ≤ 500, ≤ 300 peaks) must be translated de novo into a ranked list of candidate SMILES structures without a reference database.

## When NOT to use

- Input is an MS/MS (tandem) spectrum rather than a GC-EI-MS spectrum — the model was trained specifically on EI-MS data and comparisons in the paper note other de novo models were built for MS/MS
- Spectra violate model constraints (m/z > 500, more than 300 peaks, deuterium-containing compounds, expected SMILES longer than 100 characters)
- A rigorously curated reference library (e.g. NIST or SWGDRUG) is available and a database search would suffice — SpecTUS is aimed at novel compounds absent from spectral libraries

## Inputs

- Preprocessed low-resolution GC-EI-MS spectrum (intensity-binned, threshold-filtered), e.g. from an .msp file
- Pretrained SpecTUS checkpoint from https://huggingface.co/MS-ML
- Constraint-compliant spectra: max m/z 500, ≤ 300 peaks, SMILES ≤ 100 characters

## Outputs

- spectus_predictions.csv with one row per candidate: spectrum identifier, candidate rank, SMILES string, probability score
- Up to 10 ranked, parseable SMILES candidate strings per spectrum

## How to apply

Load the pretrained/fine-tuned 354M-parameter SpecTUS encoder-decoder checkpoint from the MS-ML Hugging Face release, along with intensity-binned, threshold-filtered spectra from upstream preprocessing. Verify each spectrum satisfies the model constraints (max m/z 500, max 300 peaks, target SMILES ≤ 100 characters); these thresholds excluded ~3% of the NIST dataset. Run autoregressive beam-search decoding in which the encoder compresses each peak into one aggregated embedding (summed m/z and intensity embeddings) and the decoder emits character-level SMILES tokens. Keep up to 10 candidates per spectrum, ranked by the model's inherent certainty score, and write spectus_predictions.csv with one row per candidate containing spectrum identifier, candidate rank, SMILES string, and probability score. The logarithmic binning with 30 bins used during training (0.9% better validation Acc1 than 4-decimal linear binning) means input spectra must match that representation.

## Related tools

- **SpecTUS** (Encoder-decoder transformer (354M parameters) that autoregressively decodes ranked SMILES candidates from the binned EI-MS spectrum via beam search) — https://github.com/hejjack/SpecTUS
- **Jupyter Notebook** (Demo inference notebook (spectus-demo) that sends spectra to the SpecTUS service to obtain ranked SMILES predictions) — https://github.com/ljocha/spectus-demo
- **Hugging Face Hub** (Hosts the 17.2M synthetic pretraining spectra and the pretrained SpecTUS checkpoint used for inference) — https://huggingface.co/MS-ML

## Examples

```
jupyter notebook notebooks/5_inference_on_open_data.ipynb
```

## Evaluation signals

- Output CSV contains at most 10 parseable (valid) SMILES candidates per spectrum with non-decreasing probability scores by rank
- Reference performance on the NIST 20 held-out test set (28,267 spectra): Acc1 of 43% and Acc10 of 65% perfect reconstructions
- Spot-check Tanimoto similarity of Morgan fingerprints of top candidates against known structures; the paper reports 65% perfect Tanimoto similarity among ten sampled suggestions
- Every input spectrum passed the constraint check (m/z ≤ 500, ≤ 300 peaks) before translation; rejected spectra are flagged rather than silently truncated
- Predictions of true compounds absent from NIST-style libraries should still yield chemically plausible top-10 candidates, reflecting de novo rather than lookup behavior

## Limitations

- SpecTUS cannot explicitly justify its outputs for a given input (no explainability of predictions)
- Lacks an intermediate representation such as peak annotations or fragmentation trees, and does not provide reference spectra for candidates, limiting manual correction
- Sensitive to input spectrum quality — performance is poorer on the Cayman and MONA test sets than on the more rigorously curated NIST or SWGDRUG libraries
- The final fine-tuned model is not redistributable because NIST20 is proprietary; a NIST20 license is required to train it yourself (the pretrained checkpoint is available on Hugging Face)

## Evidence
