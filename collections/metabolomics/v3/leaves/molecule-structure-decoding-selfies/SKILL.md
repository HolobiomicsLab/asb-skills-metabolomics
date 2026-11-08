---
name: molecule-structure-decoding-selfies
description: Use when a generative model outputs molecule representations as SELFIES
  token sequences (possibly beam-sampled) and you must convert them to valid SMILES,
  then score Top-1/Top-10 accuracy by InChIKey, minimum MCES distance, and maximum
  Tanimoto similarity.
when_to_use_negative:
- The model output is already a valid SMILES string and no SELFIES decoding is needed
  — decoding SELFIES is specific to models that emit SELFIES tokens (MS-BART, FRIGID
  use SAFE/SELFIES-style tokenization).
- Only spectrum-level metadata (e.g., formula, adduct) needs to be predicted without
  generating any molecular structure candidates — this skill presupposes a generative
  molecule decoder.
- Evaluating retrieval-based methods that return database compounds directly rather
  than de novo generated token sequences.
edam_operation: http://edamontology.org/operation_4009
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_0091
tools:
- name: MS-BART
  role: Generative model producing SELFIES token sequences for each spectrum; its
    chemical feedback alignment (Tanimoto rank loss) shapes the token distribution
    that is decoded.
  repo: github.com/OpenDFM/MS-BART
- name: RDKit
  role: Cheminformatics toolkit used to compute InChIKey exact matches, MCES distance,
    and Morgan fingerprints (radius 2, 2048 bits) for Tanimoto similarity after SELFIES
    decoding.
- name: SELFIES
  role: String-based molecular representation tokenized by MS-BART; decoding SELFIES
    back to SMILES is the core of this skill.
- name: MIST
  role: Upstream model predicting the 4096-bit fingerprint that conditions generation;
    not involved in decoding itself but supplies the conditioning input.
  repo: github.com/coleygroup/FRIGID
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:ec76e7aec8699f4dca291c2f28d878f97122bc9b331ce9d2af9f252291fd11d3
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BART-Base
  - MIST
  - MS-BART
  - MassSpecGym
  - NPLIB1
  - RDKit
  - SELFIES
  edam_operation: http://edamontology.org/operation_4009
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# molecule-structure-decoding-selfies

## Summary

Decode generated molecular SELFIES token sequences into SMILES molecules and compute identity and structural similarity metrics (exact InChIKey match, MCES distance, Tanimoto similarity) using RDKit, as done in MS-BART for evaluating structure elucidation from mass spectra.

## When to use

Use when a generative model outputs molecule representations as SELFIES token sequences (possibly beam-sampled) and you must convert them to valid SMILES, then score Top-1/Top-10 accuracy by InChIKey, minimum MCES distance, and maximum Tanimoto similarity.

## When NOT to use

- The model output is already a valid SMILES string and no SELFIES decoding is needed — decoding SELFIES is specific to models that emit SELFIES tokens (MS-BART, FRIGID use SAFE/SELFIES-style tokenization).
- Only spectrum-level metadata (e.g., formula, adduct) needs to be predicted without generating any molecular structure candidates — this skill presupposes a generative molecule decoder.
- Evaluating retrieval-based methods that return database compounds directly rather than de novo generated token sequences.

## Inputs

- Generated SELFIES token sequences from the model (beam-search multinomial sampling, beam widths up to 100)
- Ground-truth molecule structures for the test fold (SMILES, from MassSpecGym or NPLIB1)
- Predicted chemical formula for re-ranking (MS1 precursor formula)

## Outputs

- Decoded SMILES / molecular structures for each generated candidate
- Top-1 and Top-10 accuracy by exact full InChIKey match
- Minimum MCES distance and maximum Tanimoto similarity (RDKit Morgan radius 2, 2048 bits)

## How to apply

The model (MS-BART, a BART-Base backbone) generates molecules as SELFIES token sequences conditioned on fingerprint tokens; decode each SELFIES sequence to a canonical SMILES molecule via the SELFIES library and RDKit. For each spectrum in the MassSpecGym or NPLIB1 test fold, generate candidate molecules with beam-search multinomial sampling (beam widths up to 100), optionally re-ranking the 100 DiffMS-comparison samples by formula distance with log-probability tie-breaking. Compute Top-1 and Top-10 accuracy by exact full InChIKey match, minimum maximum common edge substructure (MCES) distance, and maximum Tanimoto similarity using RDKit Morgan fingerprints (radius 2, 2048 bits). A valid decode requires that the SELFIES string converts to a chemically sensible molecule — the chemical feedback alignment step (Tanimoto-similarity rank loss, margin gamma scaled by rank difference) is applied upstream precisely to push generated tokens toward molecules structurally closer to the reference, mitigating molecular hallucinations.

## Related tools

- **MS-BART** (Generative model producing SELFIES token sequences for each spectrum; its chemical feedback alignment (Tanimoto rank loss) shapes the token distribution that is decoded.) — github.com/OpenDFM/MS-BART
- **RDKit** (Cheminformatics toolkit used to compute InChIKey exact matches, MCES distance, and Morgan fingerprints (radius 2, 2048 bits) for Tanimoto similarity after SELFIES decoding.)
- **SELFIES** (String-based molecular representation tokenized by MS-BART; decoding SELFIES back to SMILES is the core of this skill.)
- **MIST** (Upstream model predicting the 4096-bit fingerprint that conditions generation; not involved in decoding itself but supplies the conditioning input.) — github.com/coleygroup/FRIGID

## Examples

```
python evaluate.py --model-path data/MassSpecGym/model-weights --beams 100 --metrics top1,top10,mces,tanimoto --output results_table.csv
```

## Evaluation signals

- Every decoded SELFIES sequence must yield a molecule whose SMILES passes RDKit sanitization without error.
- Reported Top-1 accuracy must land in the range consistent with the paper: NPLIB1 Top-1 accuracy 7.45%, MassSpecGym Top-1 accuracy 1.07% for MS-BART; NPLIB1 MCES 9.66 and Tanimoto 0.44; MassSpecGym MCES 16.47 and Tanimoto 0.28.
- Gold-fingerprint upper bounds must exceed achieved values: 73.50% and 47.56% Top-1 accuracy on NPLIB1 and MassSpecGym respectively.
- Reproduced metrics should match the paper's Table-1-style comparison across baselines (Spec2Mol, MIST+Neuraldecipher, MIST+MSNovelist, MADGEN, DiffMS) within reasonable tolerance.
- Tanimoto similarity must be computed with Morgan fingerprints of radius 2 and 2048 bits — verify the RDKit fingerprint parameters match before comparing scores.

## Limitations

- The accuracy of upstream fingerprint predictions (MIST) significantly impacts the overall performance of the decoded structures.
- MS-BART hallucinates molecular structures; the chemical feedback alignment step is designed to reduce but not eliminate molecular hallucinations.
- Formula information is used only for re-ranking (formula distance, Eq. 6) and not for training the model, deferring tighter integration to future work.
- For the DiffMS comparison, sampling 100 molecules per spectrum and re-ranking by formula distance with log-probability tie-breaking is required for a fair head-to-head evaluation.

## Evidence

- [methods] SELFIES token decode to SMILES: [evidence span withheld — anchor status: not_found]
- [methods] SELFIES denoising task: [evidence span withheld — anchor status: not_found]
- [methods] Tanimoto fingerprint parameters: [evidence span withheld — anchor status: not_found]
- [results] InChIKey accuracy evaluation: [evidence span withheld — anchor status: not_found]
- [readme] Beam-search sampling: [evidence span withheld — anchor status: not_found]
- [methods] DiffMS re-ranking protocol: [evidence span withheld — anchor status: not_found]
