---
name: molecular-structure-generation-from-spectra
description: Use when you need de novo small-molecule structure candidates from unannotated
  tandem mass spectra, conditioned on chemical formula metadata, without a reference
  spectral library — e.g. MassSpecGym or NPLIB1-style test folds.
when_to_use_negative:
- When a reference spectral library exists and library-search retrieval is sufficient
  (retrieval-based pipelines are cheaper and more reliable for known compounds)
- When the adduct is not [M+H]+ (e.g. [M+Na]+) — the model was fine-tuned and aligned
  only on the dominant [M+H]+ data
- When exact structure is required without candidate re-ranking — molecular hallucination
  remains possible and Top-1 accuracy on MassSpecGym is still below diffusion baselines
  for some metrics
edam_operation: http://edamontology.org/operation_3937
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MS-BART
  role: BART-Base language model finetuned for fingerprint-to-SELFIES translation
    from MS/MS; used to generate beam candidates
  repo: https://github.com/OpenDFM/MS-BART
- name: MIST
  role: Spectral fingerprint-prediction model that converts the mass spectrum into
    4096-bit Morgan fingerprint tokens for MS-BART input and re-ranking
  repo: https://github.com/coleygroup/FRIGID
- name: RDKit
  role: Computes ground-truth Morgan fingerprints from unlabeled molecules for pretraining
    corpus construction and Tanimoto re-ranking
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:fdc350063c34c6d9f7196b657426fe38eea06926fe9871e9e9c06483cf1cdcbe
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenDFM/MS-BART
  tools:
  - MIST
  - MS-BART
  - RDKit
  edam_operation: http://edamontology.org/operation_3937
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# molecular-structure-generation-from-spectra

## Summary

Generate candidate molecular structures from MS/MS spectra by representing each spectrum as a tokenized Morgan fingerprint intermediate, using a finetuned BART language model (MS-BART) for fingerprint-to-SELFIES translation, and re-ranking beam candidates against predicted fingerprints — the paper's fastest de novo elucidation approach (~3 s/spectrum vs ~160 s for diffusion).

## When to use

Use when you need de novo small-molecule structure candidates from unannotated tandem mass spectra, conditioned on chemical formula metadata, without a reference spectral library — e.g. MassSpecGym or NPLIB1-style test folds.

## When NOT to use

- When a reference spectral library exists and library-search retrieval is sufficient (retrieval-based pipelines are cheaper and more reliable for known compounds)
- When the adduct is not [M+H]+ (e.g. [M+Na]+) — the model was fine-tuned and aligned only on the dominant [M+H]+ data
- When exact structure is required without candidate re-ranking — molecular hallucination remains possible and Top-1 accuracy on MassSpecGym is still below diffusion baselines for some metrics

## Inputs

- Tandem mass spectrum (MS/MS) with precursor m/z, collision energy, and adduct metadata
- Chemical formula of the precursor ion (used for MIST conditioning and re-ranking)
- Pretrained MS-BART checkpoint (BART-Base backbone, tokenizer with unified fingerprint/SELFIES vocabulary)
- Pretrained MIST fingerprint-prediction checkpoint (e.g. MIST 4M-aug for MassSpecGym)

## Outputs

- Ranked list of candidate molecular structures as SELFIES/SMILES strings (Top-1 … Top-k per spectrum)
- Per-spectrum wall-clock inference latency and beam-search statistics
- Evaluation metrics: Top-1/Top-10 accuracy, MCES distance, Tanimoto similarity

## How to apply

Tokenize each spectrum as a 4096-bit radius-2 circular Morgan fingerprint: use the pretrained MIST model conditioned on the chemical formula to predict fingerprint bits from the spectrum, then apply a bit threshold ε to binarize them into fingerprint tokens. Feed these tokens, together with the precursor metadata, into MS-Bart — a BART-Base backbone whose encoder was pretrained on fingerprint–SELFIES molecule pairs computed via RDKit for ~4M unlabeled molecules — and run beam-search multinomial sampling over the fingerprint-to-molecule translation task. Sweep beam width (10–100) as the accuracy/latency trade-off; Top-1 and Top-10 accuracy rise monotonically with width while wall-clock time scales linearly. Re-rank the beam candidates using the chemical feedback / Tanimoto similarity against the predicted fingerprint; on spectra with non-dominant ion species such as [M+Na]+, filter them out first and retain only [M+H]+ data before fine-tuning/inference. Validate candidates via MCES distance and Tanimoto similarity against ground truth and check that average inference latency at beam width 100 is ~3 s on an A800-class GPU (≈53× faster than DiffMS).

## Related tools

- **MS-BART** (BART-Base language model finetuned for fingerprint-to-SELFIES translation from MS/MS; used to generate beam candidates) — https://github.com/OpenDFM/MS-BART
- **MIST** (Spectral fingerprint-prediction model that converts the mass spectrum into 4096-bit Morgan fingerprint tokens for MS-BART input and re-ranking) — https://github.com/coleygroup/FRIGID
- **RDKit** (Computes ground-truth Morgan fingerprints from unlabeled molecules for pretraining corpus construction and Tanimoto re-ranking)

## Examples

```
python scripts/predict.py --task fingerprint_to_molecule --data MassSpecGym/test --model data/MassSpecGym/model-weights --beam 100
```

## Evaluation signals

- Top-1 and Top-10 accuracy (e.g. 8.21% Top-1 on MassSpecGym with MIST 4M-aug FRIGID baseline; MS-BART reports SOTA on 5/12 metrics) improve monotonically as beam width increases from 10 to 100
- Average per-spectrum inference time at beam width 100 is ~3 s on an A800-SXM4-80GB (reported 53× speed-up vs DiffMS ~160 s)
- Top-1 MCES distance drops (≈9.66 on NPLIB1 for MS-BART) and Top-1 Tanimoto similarity (≈0.44 on NPLIB1) improve when beam width is increased
- No candidates are generated for spectra whose adduct is not [M+H]+ unless the spectrum is first filtered/converted accordingly
- Re-ranked candidates score higher Tanimoto similarity to the MIST-predicted fingerprint than unranked beam outputs

## Limitations

- MS-Bart is heavily dependent on the accuracy of the external MIST fingerprint predictions; poor fingerprints propagate directly to poor structures
- Formula information is used only for re-ranking and not for training, which limits how much the formula constraint shapes generation
- The model hallucinates molecules occasionally; the chemical-feedback alignment only reduces, not eliminates, hallucinations
- Evaluation on NIST23 was skipped in the paper because that dataset is commercially restricted, so benchmark generalization beyond MassSpecGym/NPLIB1 is partially unverified

## Evidence

- [methods] tokenize spectra as Morgan fingerprints via MIST: [evidence span withheld — anchor status: not_found]
- [readme] beam search multinomial sampling for inference: [evidence span withheld — anchor status: not_found]
- [methods] adduct filtering before fine-tuning: [evidence span withheld — anchor status: not_found]
