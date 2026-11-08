---
name: mass-spectra-to-smiles-translation
description: Use when you have a preprocessed low-resolution GC-EI-MS spectrum (intensity-binned,
  threshold-filtered) and need de novo molecular structure candidates as SMILES strings
  without a reference spectral library match.
when_to_use_negative:
- Input is an MS/MS (tandem) spectrum — SpecTUS was designed and trained for GC-EI-MS
  spectra, and comparison with MS/MS de novo methods is not meaningful
- Spectrum violates model constraints (m/z above 500, more than 300 peaks, or expected
  SMILES longer than 100 characters) — apply the upstream filter step first instead
- Input spectra are low-quality or from poorly curated sources (e.g. Cayman, MONA-like),
  where model performance degrades significantly versus curated NIST/SWGDRUG-quality
  data
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SpecTUS
  role: Encoder-decoder transformer (354M params) that translates an EI-MS spectrum
    directly into ranked SMILES candidates via autoregressive beam-search inference
  repo: https://github.com/hejjack/SpecTUS
- name: Hugging Face Hub
  role: Hosts the pretrained 354M-parameter checkpoint and the 17.2M synthetic pretraining
    spectra under the MS-ML organization
  repo: https://huggingface.co/MS-ML
- name: Jupyter Notebook
  role: Runs the demo inference notebook (demo.ipynb in spectus-demo) as a Binder
    client to the SpecTUS model served via REST API
  repo: https://github.com/ljocha/spectus-demo
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:40654065927dd320a5f69ec7da78e933a26149a18fbd00aefff159b9df7941d3
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
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# mass-spectra-to-smiles-translation

## Summary

De novo translation of GC-EI-MS spectra into ranked SMILES candidate structures using the SpecTUS encoder-decoder transformer (354M parameters), pretrained on 17.2M synthetic NEIMS/RASSP spectra and finetuned on NIST 20 experimental spectra, bypassing spectral library database search.

## When to use

Use when you have a preprocessed low-resolution GC-EI-MS spectrum (intensity-binned, threshold-filtered) and need de novo molecular structure candidates as SMILES strings without a reference spectral library match.

## When NOT to use

- Input is an MS/MS (tandem) spectrum — SpecTUS was designed and trained for GC-EI-MS spectra, and comparison with MS/MS de novo methods is not meaningful
- Spectrum violates model constraints (m/z above 500, more than 300 peaks, or expected SMILES longer than 100 characters) — apply the upstream filter step first instead
- Input spectra are low-quality or from poorly curated sources (e.g. Cayman, MONA-like), where model performance degrades significantly versus curated NIST/SWGDRUG-quality data

## Inputs

- Low-resolution GC-EI-MS spectrum, intensity-binned and threshold-filtered (max m/z 500, max 300 peaks)
- Pretrained SpecTUS checkpoint from Hugging Face Hub (MS-ML organization)
- Optional: msp-format spectra file for inference on open data

## Outputs

- Up to 10 ranked candidate SMILES strings per spectrum with certainty/probability scores
- Predictions table (e.g. spectus_predictions.csv) with columns: spectrum identifier, candidate rank, SMILES string, probability score

## How to apply

Load the pretrained SpecTUS encoder-decoder transformer checkpoint (354M parameters) from the MS-ML Hugging Face release, together with the upstream intensity-binned, threshold-filtered spectra. Validate each input spectrum against model constraints — m/z up to 500, at most 300 peaks, and target SMILES length up to 100 characters — which excluded roughly 3% of the NIST dataset. Internally, the encoder compresses each spectrum peak into one aggregated embedding by summing the m/z and intensity embeddings; the decoder then autoregressively generates SMILES characters (character-level encoding, which outperformed BPE and SELFIES tokenizations by 3.5–5.8% in validation Acc1). Run autoregressive beam-search inference to produce up to 10 candidate SMILES strings per spectrum, ranked by the model's inherent certainty score. Write results as one row per candidate with spectrum identifier, candidate rank, SMILES string, and probability score.

## Related tools

- **SpecTUS** (Encoder-decoder transformer (354M params) that translates an EI-MS spectrum directly into ranked SMILES candidates via autoregressive beam-search inference) — https://github.com/hejjack/SpecTUS
- **Hugging Face Hub** (Hosts the pretrained 354M-parameter checkpoint and the 17.2M synthetic pretraining spectra under the MS-ML organization) — https://huggingface.co/MS-ML
- **Jupyter Notebook** (Runs the demo inference notebook (demo.ipynb in spectus-demo) as a Binder client to the SpecTUS model served via REST API) — https://github.com/ljocha/spectus-demo

## Examples

```
git clone https://github.com/hejjack/SpecTUS.git && conda env create -f trainSpectus_build.yaml  # then run notebooks/5_inference_on_open_data.ipynb on an msp file
```

## Evaluation signals

- Output contains at most 10 parseable SMILES candidates per spectrum, each with rank and probability score
- Reference performance on the NIST 20 held-out test set (28,267 spectra): Acc1 ≈ 43% and Acc10 ≈ 65% perfect reconstructions
- Tanimoto similarity of Morgan fingerprints should reach 1.0 for ~65% of sampled queries at 10 suggestions
- All input spectra passed the pre-inference check: m/z ≤ 500 and ≤ 300 peaks
- Generated SMILES strings are chemically parseable and at most 100 characters

## Limitations

- SpecTUS cannot explicitly justify its outputs for a given input (no explainability of predictions)
- Lacks an intermediate representation such as peak annotations or fragmentation trees, and does not provide reference spectra for candidates, limiting manual correction
- Model performance is sensitive to input spectra quality — notably poorer on Cayman and MONA test sets than on curated NIST or SWGDRUG libraries
- Finetuning the model yourself requires a license for the NIST20 dataset; the final finetuned model cannot be freely shared
- All existing de novo comparison baselines were designed for MS/MS spectra, so direct comparison with SpecTUS on GC-EI-MS is challenging

## Evidence

- [methods] encoder/decoder architecture: [evidence span withheld — anchor status: not_found]
