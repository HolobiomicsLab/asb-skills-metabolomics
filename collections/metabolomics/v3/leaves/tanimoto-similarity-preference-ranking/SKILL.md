---
name: tanimoto-similarity-preference-ranking
description: 'Use when reducing molecular hallucinations in a fingerprint-to-SELFIES
  generative model: you have a fine-tuned checkpoint, per-input ground-truth structures,
  and need preference-based ranking of sampled candidates for alignment training.'
when_to_use_negative:
- Inference/deployment on novel spectra where no ground-truth structure exists — the
  preference scores Ps(Si) are computed against ground-truth references, so ranking
  cannot be supervised
- Baseline generation or fingerprint-prediction stages — this skill applies only to
  the alignment (Stage 3) step after pretraining and MIST-fingerprint finetuning are
  complete
tools:
- name: MS-BART
  role: Unified spectra–molecule language model being aligned; candidate SELFIES sequences
    are sampled from its fine-tuned decoder and probability mass is shifted by the
    rank loss
  repo: github.com/OpenDFM/MS-BART
- name: RDKit
  role: Computes Morgan fingerprints (radius 2) of candidates and ground-truth structures
    used to score Tanimoto-similarity preference
- name: SELFIES
  role: String-based molecular representation in which candidates are generated and
    ranked
- name: zjunlp/MolGen
  role: Prior-work reference for chemical-feedback alignment with rank-based preference;
    source of the freeze-encoder/update-decoder practice
  repo: github.com/zjunlp/MolGen
provenance:
  source_task_ids:
  - task_008
  source_content_hash: sha256:44ce49f5f302c795d71a5ccbb75ecd167334a2abae3935fdb36f447fb0d2684f
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BART-Base
  - MS-BART
  - RDKit
  - SELFIES
  - zjunlp/MolGen
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# tanimoto-similarity-preference-ranking

## Summary

A chemical-feedback alignment procedure for molecule-from-spectrum generators (MS-BART): sample candidate SELFIES sequences, score each candidate by Tanimoto similarity to the ground-truth structure (RDKit Morgan fingerprints), rank by molecular-formula distance, and train with a rank-scaled contrastive loss so probability mass shifts toward candidates closer to the reference.

## When to use

Use when reducing molecular hallucinations in a fingerprint-to-SELFIES generative model: you have a fine-tuned checkpoint, per-input ground-truth structures, and need preference-based ranking of sampled candidates for alignment training.

## When NOT to use

- Inference/deployment on novel spectra where no ground-truth structure exists — the preference scores Ps(Si) are computed against ground-truth references, so ranking cannot be supervised
- Baseline generation or fingerprint-prediction stages — this skill applies only to the alignment (Stage 3) step after pretraining and MIST-fingerprint finetuning are complete

## Inputs

- Fine-tuned MS-BART checkpoint (Pretrain-FT stage, BART-Base backbone)
- Thresholded fingerprint–SELFIES finetuning data (e.g., logs/datasets/MassSpecGym/, threshold 0.2, [M+H]+ only)
- Ground-truth molecular structures per spectrum (SELFIES/SMILES for reference Morgan fingerprint computation)
- Morgan fingerprint parameters: radius 2, bit length 2048

## Outputs

- Chemical-feedback-aligned MS-BART checkpoint saved under logs/<run-name>
- Ranked candidate SELFIES sequences per input with Tanimoto preference scores Ps(Si) and formula distances
- Validation-split metrics: Top-1/Top-10 Tanimoto, accuracy, and MCES (compare against Pretrain-FT, Fig. 3 Stage 3)

## How to apply

Load the fine-tuned MS-BART checkpoint (BART-Base backbone, Pretrain-FT stage) and the thresholded fingerprint–SELFIES finetuning data (fingerprint bit threshold ε = 0.2 on MassSpecGym; [M+Na]+ adducts filtered out, retaining only [M+H]+). Generate n candidate SELFIES sequences per fingerprint input using beam-search multinomial sampling. Compute a 2048-bit radius-2 Morgan fingerprint for each candidate and the ground-truth structure with RDKit, score preference Ps(Si) as Tanimoto similarity, then rank candidates by molecular-formula distance (sum of per-atom count differences) using log-probability as tie-breaker. Train with L = L_ce + alpha*L_rank, where the contrastive rank loss uses rank-scaled margins gamma_ij = (j-i)*gamma to push probability toward higher-Tanimoto candidates; freeze the encoder and update only the decoder, following prior-work practice (MolGen). Save the aligned checkpoint and evaluate on the validation split. Note that formula information is only used for re-ranking, not injected into model inputs.

## Related tools

- **MS-BART** (Unified spectra–molecule language model being aligned; candidate SELFIES sequences are sampled from its fine-tuned decoder and probability mass is shifted by the rank loss) — github.com/OpenDFM/MS-BART
- **RDKit** (Computes Morgan fingerprints (radius 2) of candidates and ground-truth structures used to score Tanimoto-similarity preference)
- **SELFIES** (String-based molecular representation in which candidates are generated and ranked)
- **zjunlp/MolGen** (Prior-work reference for chemical-feedback alignment with rank-based preference; source of the freeze-encoder/update-decoder practice) — github.com/zjunlp/MolGen

## Examples

```
from rdkit import DataStructs; sim = DataStructs.TanimotoSimilarity(Morgan(fp_gen_mol, 2, nBits=2048), Morgan(gt_mol, 2, nBits=2048))
```

## Evaluation signals

- Aligned checkpoint shows improved Top-1/Top-10 Tanimoto, accuracy, and MCES on the validation split relative to the Pretrain-FT stage (Fig. 3 Stage 3)
- Ranked probability mass shifts toward candidates with higher Tanimoto similarity: expected sequence probability should be monotone in preference rank under gamma_ij = (j-i)*gamma margins
- Candidates ranked by molecular-formula distance (sum of per-atom count differences) agree with Tanimoto ordering; ties resolved by log-probability reproduce a consistent, deterministic ranking
- Combined objective retains token-level cross-entropy: L_ce on the reference SELFIES should not diverge while alpha*L_rank decreases across alignment epochs
- Encoder parameters remain bit-identical to the Pretrain-FT checkpoint (frozen); only decoder weights change after alignment

## Limitations

- Ranking supervises against ground-truth structures, so the mechanism cannot apply where reference structures are unavailable (e.g., true novel-compound elucidation)
- Chemical formula information is used only for candidate re-ranking and not for training the model; incorporating it is deferred to future work
- Overall quality depends on upstream MIST fingerprint-prediction accuracy; incorrect predicted fingerprints propagate into the candidate pool being ranked

## Evidence

- [readme] Rank candidates by Tanimoto similarity to the predicted fingerprint and select top-M.: [evidence span withheld — anchor status: external_doc]
- [methods] we filter out [M+Na]+ adducts before fine-tuning and alignment, retaining only the dominant [M+H]+ data.: [evidence span withheld — anchor status: not_found]
