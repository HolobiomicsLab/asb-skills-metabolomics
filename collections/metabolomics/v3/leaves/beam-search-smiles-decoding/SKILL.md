---
name: beam-search-smiles-decoding
description: Use when you have a threshold-filtered, intensity-binned EI-MS spectrum
  (m/z ≤ 500, ≤ 300 peaks, SMILES ≤ 100 chars) and need de novo 2D structure candidates
  without a reference database.
when_to_use_negative:
- Input spectra are MS/MS (tandem) spectra — all compared de novo approaches were
  designed and trained for MS/MS, not GC-EI-MS, and SpecTUS likewise expects EI-MS
  input
- Spectra exceed the model constraints (m/z > 500, more than 300 peaks, or target
  SMILES longer than 100 characters) — such inputs were filtered out (~3% of NIST)
- The analysis requires explainable predictions, peak annotations, fragmentation trees,
  or reference spectra for candidates — SpecTUS provides none of these intermediate
  representations
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_2258
tools:
- name: SpecTUS
  role: Encoder-decoder transformer whose decoder performs the autoregressive beam-search
    SMILES generation and certainty-scored candidate ranking
  repo: https://github.com/hejjack/SpecTUS
- name: 'SpecTUS: Spectral Translator for Unknown Structures'
  role: The published model/method whose inference routine implements the spectrum-to-SMILES
    translation
  repo: https://github.com/hejjack/SpecTUS
- name: Jupyter Notebook
  role: Demonstration and inference interface; the demo notebook runs beam-search
    prediction as a client of the model served via REST API, and notebooks/5_inference_on_open_data.ipynb
    reconstructs structures from an msp file
  repo: https://github.com/ljocha/spectus-demo
- name: Hugging Face Hub
  role: Hosts the pretrained model checkpoint used to initialize the decoder before
    beam-search inference
  repo: https://huggingface.co/MS-ML
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:a099ac05f6d056f5187fb3b6f02cea0538f9e312358749ff9712cd199320b69d
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
  - 'SpecTUS: Spectral Translator for Unknown Structures'
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# beam-search-smiles-decoding

## Summary

Runs autoregressive beam-search inference with the SpecTUS encoder-decoder transformer (354M parameters) to translate a preprocessed low-resolution GC-EI-MS spectrum into up to 10 ranked SMILES candidate structures, scored by the model's inherent certainty.

## When to use

Use when you have a threshold-filtered, intensity-binned EI-MS spectrum (m/z ≤ 500, ≤ 300 peaks, SMILES ≤ 100 chars) and need de novo 2D structure candidates without a reference database.

## When NOT to use

- Input spectra are MS/MS (tandem) spectra — all compared de novo approaches were designed and trained for MS/MS, not GC-EI-MS, and SpecTUS likewise expects EI-MS input
- Spectra exceed the model constraints (m/z > 500, more than 300 peaks, or target SMILES longer than 100 characters) — such inputs were filtered out (~3% of NIST)
- The analysis requires explainable predictions, peak annotations, fragmentation trees, or reference spectra for candidates — SpecTUS provides none of these intermediate representations

## Inputs

- Intensity-binned, threshold-filtered GC-EI-MS spectrum (m/z ≤ 500, ≤ 300 peaks), e.g. entries from an msp file
- Pretrained or finetuned SpecTUS encoder-decoder transformer checkpoint (354M parameters) from the MS-ML Hugging Face release

## Outputs

- Up to 10 ranked, parseable SMILES candidate strings per input spectrum, each with a probability/certainty score
- Prediction table (e.g. spectus_predictions.csv) with spectrum identifier, candidate rank, SMILES string, and probability score per row

## How to apply

Load the pretrained checkpoint from the MS-ML Hugging Face release (or a locally finetuned NIST 20 model), then verify each input spectrum satisfies the model constraints: maximum m/z of 500, at most 300 peaks per spectrum, and maximum SMILES length of 100 characters — these thresholds excluded ~3% of NIST. Internally, the encoder compresses each spectrum peak into one aggregated embedding by summing m/z and intensity embeddings, and the decoder autoregressively generates SMILES character-by-character (character-level encoding, not BPE). Run beam-search decoding to emit up to 10 candidate SMILES strings per spectrum, ranked by the model's probability/certainty score. Write one row per candidate (spectrum identifier, candidate rank, SMILES, probability score), e.g. to spectus_predictions.csv. Because the decoder uses character-level SMILES with no intermediate representation, candidates must be chemically validated post hoc (e.g. parseability and Morgan fingerprint Tanimoto similarity against any known structure).

## Related tools

- **SpecTUS** (Encoder-decoder transformer whose decoder performs the autoregressive beam-search SMILES generation and certainty-scored candidate ranking) — https://github.com/hejjack/SpecTUS
- **SpecTUS: Spectral Translator for Unknown Structures** (The published model/method whose inference routine implements the spectrum-to-SMILES translation) — https://github.com/hejjack/SpecTUS
- **Jupyter Notebook** (Demonstration and inference interface; the demo notebook runs beam-search prediction as a client of the model served via REST API, and notebooks/5_inference_on_open_data.ipynb reconstructs structures from an msp file) — https://github.com/ljocha/spectus-demo
- **Hugging Face Hub** (Hosts the pretrained model checkpoint used to initialize the decoder before beam-search inference) — https://huggingface.co/MS-ML

## Examples

```
git clone https://github.com/hejjack/SpecTUS.git && cd SpecTUS && conda env create -f trainSpectus_build.yaml && jupyter notebook notebooks/5_inference_on_open_data.ipynb
```

## Evaluation signals

- Output contains at most 10 parseable (chemically valid) SMILES candidates per spectrum, each ranked with a probability score
- On the held-out NIST 20 test split (28,267 spectra), expected reference performance is Acc1 of 43% (single suggestion perfectly reconstructs 43% of compounds) and Acc10 of 65% (65% perfect reconstructions with ten suggestions)
- Tanimoto similarity of Morgan fingerprints: with ten suggestions the top candidate set should surpass hybrid database search on ~84% of the test set
- Every input spectrum passes the constraint check (m/z ≤ 500, ≤ 300 peaks, SMILES ≤ 100 characters) before decoding
- Each spectrum identifier maps to its own candidate list with no cross-spectrum leakage of ranks or scores

## Limitations

- SpecTUS cannot explicitly justify its specific outputs for a given input (no explainability of predictions)
- It lacks an intermediate representation, such as peak annotations or fragmentation trees, and does not provide reference spectra for the candidates, limiting manual correction
- Performance is sensitive to input spectra quality — poorer results on the less rigorously curated Cayman and MONA test sets compared with NIST or SWGDRUG
- The finetuned NIST 20 model cannot be shared because NIST20 is proprietary; only the pretrained checkpoint and 17.2M synthetic spectra are publicly released, so full reproduction requires a NIST20 license

## Evidence

- [methods] Inference translates spectra into ranked SMILES candidates: "at inference, translates an analysed EI-MS spectrum directly into ranked SMILES candidates."
- [methods] Model input constraints: "a maximum m/z value of 500, a maximum of 300 peaks per spectrum, and a maximum SMILES length of 100 characters"
- [abstract] Ten-suggestion reconstruction performance: "With ten suggestions, SpecTUS achieved 65% perfect reconstructions, surpassing hybrid search on 84% of the test set."
- [abstract] Single-suggestion reconstruction performance: "the model's single suggestion perfectly reconstructed 43% of the subset's compounds."
- [readme] Inference notebook from msp file: "An example notebook for [inference](notebooks/5_inference_on_open_data.ipynb) is available to help you get started reconstructing spectra from an `msp` file"
- [readme] Model releases: pretrained checkpoint and synthetic data: "We make freely available the 17.2M synthetic spectra used for pretraining the model and the checkpoint of the pretrained model"
