---
name: molecular-formula-distance-reranking
description: Use when you have multiple molecular candidates sampled from a generative
  model and need to rank them by how close their molecular formula is to the reference
  formula, using log-probability as a tie-breaker.
when_to_use_negative:
- When only a single candidate molecule is available (no ranking signal).
- When formula information is not available at inference time - the article notes
  formula is used only for re-ranking and not for training the model.
- When candidate diversity is so low that all sampled candidates share the same molecular
  formula, making distance-based ranking uninformative.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3474
tools:
- name: MS-BART
  role: Generates n candidate SELFIES sequences per fingerprint input using beam-search
    multinomial sampling; ranking feeds its chemical-feedback alignment stage.
  repo: github.com/OpenDFM/MS-BART
- name: RDKit
  role: Computes Morgan fingerprints (radius 2, bit length 2048) for Tanimoto similarity
    scoring used as the ranking criterion.
- name: SELFIES
  role: String-based molecular representation format in which candidates are generated
    and ranked.
provenance:
  source_task_ids:
  - task_008
  source_content_hash: sha256:33473a680964ceafc941cf489d41c10b712c7b438b7f4eed36aceacd6f3ad77f
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# molecular-formula-distance-reranking

## Summary

Ranks n sampled SELFIES candidate molecules by molecular-formula distance (sum of per-atom count differences) with log-probability as tie-breaker, so that higher-Tanimoto structures receive more probability mass during chemical-feedback alignment of MS-BART.

## When to use

Use when you have multiple molecular candidates sampled from a generative model and need to rank them by how close their molecular formula is to the reference formula, using log-probability as a tie-breaker.

## When NOT to use

- When only a single candidate molecule is available (no ranking signal).
- When formula information is not available at inference time - the article notes formula is used only for re-ranking and not for training the model.
- When candidate diversity is so low that all sampled candidates share the same molecular formula, making distance-based ranking uninformative.

## Inputs

- n sampled SELFIES candidate sequences per fingerprint input from the fine-tuned MS-BART decoder
- ground-truth molecular formula of the reference structure
- Tanimoto similarity scores (RDKit Morgan, radius 2, 2048 bits) between each candidate and the reference
- model log-probabilities for each sampled candidate

## Outputs

- ordered candidate list ranked by molecular-formula distance with log-probability tie-breaker
- rank-scaled margins gamma_ij = (j-i)*gamma for the contrastive rank loss
- aligned MS-BART decoder checkpoint with encoder frozen

## How to apply

Given a fine-tuned MS-BART checkpoint (Pretrain-FT stage, BART-Base backbone) and thresholded fingerprint-SELFIES finetuning data, generate n candidate SELFIES sequences per fingerprint input via beam-search multinomial sampling. Score each candidate with Tanimoto similarity to the ground truth using RDKit Morgan fingerprints (radius 2, 2048 bits), then rank candidates by molecular-formula distance computed as the sum of per-atom count differences between candidate and reference formulae, breaking ties with the model's log-probability. Feed this ranking into a contrastive rank loss L = L_ce + alpha*L_rank where rank-scaled margins gamma_ij = (j-i)*gamma push probability toward higher-Tanimoto candidates; freeze the encoder and update only the decoder during alignment. Evaluate Top-1/Top-10 Tanimoto, accuracy, and MCES on the validation split to confirm improvement over the Pretrain-FT stage (Fig. 3 Stage 3).

## Related tools

- **MS-BART** (Generates n candidate SELFIES sequences per fingerprint input using beam-search multinomial sampling; ranking feeds its chemical-feedback alignment stage.) — github.com/OpenDFM/MS-BART
- **RDKit** (Computes Morgan fingerprints (radius 2, bit length 2048) for Tanimoto similarity scoring used as the ranking criterion.)
- **SELFIES** (String-based molecular representation format in which candidates are generated and ranked.)

## Examples

```
from rdkit import Chem, DataStructs; fps = [Chem.RDKFingerprint(Chem.MolFromSmiles(s)) for s in smiles_list]; sim = [DataStructs.TanimotoSimilarity(fps[0], f) for f in fps]
```

## Evaluation signals

- Tanimoto similarity computed with RDKit Morgan fingerprints (radius 2, 2048 bits) must be consistent with the ranking order (higher similarity should correspond to lower rank index).
- Molecular-formula distance must be computed as the sum of per-atom count differences between candidate and reference formulae.
- Tie-breaker check: candidates with equal formula distance must be ordered by descending log-probability.
- After alignment training with the rank loss, Top-1/Top-10 Tanimoto and accuracy on the validation split must exceed the Pretrain-FT baseline (Fig. 3 Stage 3).

## Limitations

- Formula information is used only for re-ranking and not for training the model; incorporating it into training is deferred to future work.
- The rank loss margin scheme (gamma_ij = (j-i)*gamma) requires careful tuning of gamma and alpha; no default values are reported in the article.
- Ranking quality is bounded by the accuracy of the upstream fingerprint predictions (MIST), which propagate through the sampled candidates.

## Evidence

- [other] rank sampled candidates by molecular-formula distance: [evidence span withheld — anchor status: not_found]
- [methods] SELFIES and SMILES as molecular representations: [evidence span withheld — anchor status: not_found]
- [methods] Morgan fingerprint definition: [evidence span withheld — anchor status: not_found]
