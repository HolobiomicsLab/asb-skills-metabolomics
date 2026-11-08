---
name: contrastive-rank-loss-alignment
description: 'Use when reducing molecular hallucinations after fingerprint-conditioned
  generation: candidates are sampled from MS-BART and their probability mass must
  be shifted toward structures most Tanimoto-similar to the reference.'
when_to_use_negative:
- No ground-truth structure is available for scoring — the preference P(S_i) is defined
  as Tanimoto similarity to the ground-truth structure, so unsupervised elucidation
  without references cannot use this rank signal
- You need formula as a training-time conditioning signal — the paper explicitly defers
  this, using formula information only for re-ranking, not in training
- Working on data where the encoder representations should keep adapting — this stage
  freezes the encoder and updates only the decoder
tools:
- name: MS-BART
  role: Unified pretraining–finetuning–alignment framework whose decoder is aligned
    with the rank loss; checkpoints saved under logs/<run-name>
  repo: github.com/OpenDFM/MS-BART
- name: RDKit
  role: Computes Morgan fingerprints from candidate and ground-truth molecules for
    Tanimoto preference scoring
- name: MolGen
  role: Origin of the chemical-feedback alignment method adapted here to mass-spectra
    elucidation
  repo: github.com/zjunlp/MolGen
- name: BART-Base
  role: Backbone of MS-BART whose decoder-only parameters are updated during alignment
- name: MIST
  role: Supplies the predicted fingerprints that condition candidate SELFIES generation
    before ranking
provenance:
  source_task_ids:
  - task_008
  source_content_hash: sha256:9f8ad031bcb5d4c7d8661925aab88d00954608e359e823008bf1ccdeecb53c47
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BART-Base
  - MIST
  - MS-BART
  - MolGen
  - RDKit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# Contrastive rank loss alignment

## Summary

A chemical-feedback alignment stage (following MolGen) that fine-tunes MS-BART after Pretrain-FT so generated SELFIES molecules closer to the ground-truth structure receive higher probability, via a rank-scaled contrastive loss added to token-level cross-entropy.

## When to use

Use when reducing molecular hallucinations after fingerprint-conditioned generation: candidates are sampled from MS-BART and their probability mass must be shifted toward structures most Tanimoto-similar to the reference.

## When NOT to use

- No ground-truth structure is available for scoring — the preference P(S_i) is defined as Tanimoto similarity to the ground-truth structure, so unsupervised elucidation without references cannot use this rank signal
- You need formula as a training-time conditioning signal — the paper explicitly defers this, using formula information only for re-ranking, not in training
- Working on data where the encoder representations should keep adapting — this stage freezes the encoder and updates only the decoder

## Inputs

- Fine-tuned MS-BART checkpoint (Pretrain-FT, BART-Base backbone)
- Thresholded fingerprint–SELFIES finetuning pairs (MassSpecGym, threshold 0.2, [M+H]+ only)
- Ground-truth molecular structures (SELFIES/SMILES) for Tanimoto scoring
- Predicted fingerprints from MIST conditioned on molecular formula

## Outputs

- Aligned MS-BART decoder checkpoint saved under logs/<run-name>
- Ranked candidate SELFIES sequences per spectrum with Tanimoto/formula-distance scores
- Evaluation metrics: Top-1/Top-10 Tanimoto, accuracy, and MCES on the validation split

## How to apply

Load the Pretrain-FT MS-BART checkpoint (BART-Base backbone) and the thresholded fingerprint–SELFIES fine-tuning data (threshold 0.2 for MassSpecGym, retaining only [M+H]+ adducts). Generate n candidate SELFIES sequences per fingerprint input with beam-search multinomial sampling. Score each candidate by Tanimoto similarity to the ground-truth structure using RDKit Morgan fingerprints (radius 2, 2048 bits), then rank candidates by molecular-formula distance (sum of per-atom count differences) with log-probability as tie-breaker. Optimize L = L_ce + alpha*L_rank, where the rank contrastive loss uses rank-scaled margins gamma_ij = (j-i)*gamma to push probability toward higher-Tanimoto candidates. Freeze the encoder (following prior-work practice) and update only the decoder; save the aligned checkpoint under logs/<run-name> and evaluate Top-1/Top-10 Tanimoto, accuracy, and MCES, confirming improvement over the Pretrain-FT stage (Fig. 3 Stage 3).

## Related tools

- **MS-BART** (Unified pretraining–finetuning–alignment framework whose decoder is aligned with the rank loss; checkpoints saved under logs/<run-name>) — github.com/OpenDFM/MS-BART
- **RDKit** (Computes Morgan fingerprints from candidate and ground-truth molecules for Tanimoto preference scoring)
- **MolGen** (Origin of the chemical-feedback alignment method adapted here to mass-spectra elucidation) — github.com/zjunlp/MolGen
- **BART-Base** (Backbone of MS-BART whose decoder-only parameters are updated during alignment)
- **MIST** (Supplies the predicted fingerprints that condition candidate SELFIES generation before ranking)

## Examples

```
from rdkit import Chem, DataStructs; from rdkit.Chem import AllChem; sim = DataStructs.TanimotoSimilarity(AllChem.GetMorganFingerprintAsBitVect(Chem.MolFromSmiles(ref), 2, nBits=2048), AllChem.GetMorganFingerprintAsBitVect(Chem.MolFromSmiles(cand), 2, nBits=2048))
```

## Evaluation signals

- Validation Top-1/Top-10 Tanimoto, accuracy, and MCES must improve relative to the Pretrain-FT stage (Fig. 3 Stage 3 pattern)
- Candidate preference scores must be Tanimoto similarities computed with RDKit Morgan fingerprints of radius 2 and 2048 bits
- Ranking must sort sampled candidates by molecular-formula distance (sum of per-atom count differences), breaking ties by log-probability
- Encoder parameters must remain frozen after alignment — only decoder weights differ from the Pretrain-FT checkpoint
- Aligned model should assign strictly higher probability to candidates with smaller rank index i under the gamma_ij = (j-i)*gamma margin schedule

## Limitations

- Formula information is used only for re-ranking and not for training the model, which the authors defer to future work
- Overall quality depends on MIST fingerprint-prediction accuracy, since candidates are generated from MIST-predicted fingerprints
- Alignment ground truth ties preferences to a single reference structure, matching the paper's scope of reducing hallucinations toward known references

## Evidence

- [intro] we introduce an alignment step that incorporates chemical feedback by assigning higher probabilities to structures more similar to the ground truth: "we introduce an alignment step that incorporates chemical feedback by assigning higher probabilities to structures more similar to the ground truth"
- [abstract] chemical feedback mechanism guiding toward reference: "We therefore introduce a chemical feedback mechanism that guides the model toward generating molecules closer to the reference structure."
- [methods] candidate generation conditioned on fingerprint tokens: [evidence span withheld — anchor status: not_found]
- [readme] MolGen is the chemical-feedback origin: [evidence span withheld — anchor status: external_doc]
