---
name: transformer-encoder-decoder-inference
description: Use when you have an intensity-binned, threshold-filtered low-resolution
  EI-MS spectrum (m/z ≤ 500, ≤ 300 peaks) from an unknown compound and need de novo
  SMILES candidates without a spectral library.
when_to_use_negative:
- Input is an MS/MS spectrum with precursor ion mass or molecular formula cues — existing
  de novo methods were designed for MS/MS, and SpecTUS is trained and applied to GC-EI-MS
  spectra
- Spectra exceed model constraints (m/z above 500, more than 300 peaks) or compounds
  contain deuterium — such data were excluded by the model-specific filters (~3% of
  the NIST dataset was removed by these thresholds)
- High-quality input spectra from poorly curated sources (Cayman, MONA) when rigorous
  performance claims are needed — model performance is sensitive to input spectra
  quality
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: SpecTUS
  role: 354M-parameter encoder-decoder transformer that aggregates m/z and intensity
    embeddings per peak and autoregressively decodes ranked SMILES candidates via
    beam search
  repo: https://github.com/hejjack/SpecTUS
- name: Jupyter Notebook
  role: Client for demo inference against the SpecTUS model wrapped as a service on
    the cluster (ljocha/spectus-demo), or local inference via notebooks/5_inference_on_open_data.ipynb
  repo: https://github.com/ljocha/spectus-demo
- name: Hugging Face Hub
  role: Hosts the pretrained model checkpoint and the 17.2M synthetic pretraining
    spectra under the MS-ML organization
  repo: https://huggingface.co/MS-ML
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:6cb8579f7df56cb276a03df14d0ca4ceb81f678a11d025ae1f98e81d0681aab1
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
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# transformer-encoder-decoder-inference

## Summary

Run de novo structural annotation by translating a preprocessed GC-EI-MS spectrum into ranked SMILES candidates with the pretrained SpecTUS encoder-decoder transformer (354M parameters), scoring candidates by the model's inherent certainty.

## When to use

Use when you have an intensity-binned, threshold-filtered low-resolution EI-MS spectrum (m/z ≤ 500, ≤ 300 peaks) from an unknown compound and need de novo SMILES candidates without a spectral library.

## When NOT to use

- Input is an MS/MS spectrum with precursor ion mass or molecular formula cues — existing de novo methods were designed for MS/MS, and SpecTUS is trained and applied to GC-EI-MS spectra
- Spectra exceed model constraints (m/z above 500, more than 300 peaks) or compounds contain deuterium — such data were excluded by the model-specific filters (~3% of the NIST dataset was removed by these thresholds)
- High-quality input spectra from poorly curated sources (Cayman, MONA) when rigorous performance claims are needed — model performance is sensitive to input spectra quality

## Inputs

- Intensity-binned, threshold-filtered low-resolution GC-EI-MS spectrum (m/z ≤ 500, at most 300 peaks; e.g. from an msp file)
- Pretrained 354M-parameter SpecTUS encoder-decoder transformer checkpoint from Hugging Face Hub (MS-ML)
- Optional: user spectra outside the training data (demo testing data are guaranteed unused in training)

## Outputs

- spectus_predictions.csv with one row per candidate: spectrum identifier, candidate rank, SMILES string, and probability score
- Up to 10 parseable, ranked SMILES strings per input spectrum

## How to apply

Load the pretrained 354M-parameter SpecTUS encoder-decoder transformer checkpoint from the MS-ML Hugging Face release (or run the demo client notebook in ljocha/spectus-demo against the hosted REST service). Verify each input spectrum satisfies the model constraints: m/z up to 500, at most 300 peaks per spectrum, and maximum SMILES length of 100 characters for predicted structures. Encode each spectrum so that the encoder compresses every peak into one aggregated embedding by summing its m/z and intensity embeddings, then run autoregressive beam-search decoding to generate SMILES character by character. Collect up to 10 candidate SMILES strings per spectrum, ranked by the model's inherent certainty score, and write them to spectus_predictions.csv with spectrum identifier, candidate rank, SMILES string, and probability score per row. Reference performance for correctness checking is Acc1 of 43% and Acc10 of 65% on the NIST 20 held-out test set of 28,267 spectra, where the top candidate is strictly better than hybrid database search in Tanimoto similarity of Morgan fingerprints on 76% of the set.

## Related tools

- **SpecTUS** (354M-parameter encoder-decoder transformer that aggregates m/z and intensity embeddings per peak and autoregressively decodes ranked SMILES candidates via beam search) — https://github.com/hejjack/SpecTUS
- **Jupyter Notebook** (Client for demo inference against the SpecTUS model wrapped as a service on the cluster (ljocha/spectus-demo), or local inference via notebooks/5_inference_on_open_data.ipynb) — https://github.com/ljocha/spectus-demo
- **Hugging Face Hub** (Hosts the pretrained model checkpoint and the 17.2M synthetic pretraining spectra under the MS-ML organization) — https://huggingface.co/MS-ML

## Examples

```
git clone https://github.com/hejjack/SpecTUS.git && conda env create -f trainSpectus_build.yaml && jupyter notebook notebooks/5_inference_on_open_data.ipynb
```

## Evaluation signals

- Output contains at most 10 parseable (valid SMILES) ranked candidates per input spectrum
- Reference NIST 20 test-split performance: Acc1 = 43% and Acc10 = 65% perfect reconstructions
- Perfect Tanimoto similarity (Morgan fingerprint) achieved for ~65% of sampled queries
- Single suggestion strictly better than hybrid database search in Tanimoto similarity on ~76% of the NIST 20 test set
- Each input spectrum satisfies the hard constraints: m/z ≤ 500, ≤ 300 peaks, predicted SMILES ≤ 100 characters

## Limitations

- SpecTUS cannot explicitly justify its outputs for a given input — no explainability of predictions
- It lacks an intermediate representation such as peak annotations or fragmentation trees, and does not provide reference spectra for candidates, limiting manual correction
- Performance is sensitive to input spectra quality, shown by poorer results on the Cayman and MONA test sets compared to the more rigorously curated NIST or SWGDRUG libraries
- The final model finetuned on NIST20 is not redistributable because NIST20 is proprietary; the pretrained checkpoint and code are available, but finetuning yourself requires a NIST20 license

## Evidence
