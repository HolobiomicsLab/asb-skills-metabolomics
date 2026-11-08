---
name: morgan-fingerprint-tanimoto-similarity-scoring
description: Use when evaluating ranked SMILES candidate lists from spectra-to-structure
  inference (e.g. SpecTUS top-1/top-10 predictions) where exact-match Acc1/Acc10 alone
  is too harsh and partial structural similarity must be quantified.
when_to_use_negative:
- When only exact-structure reconstruction (Acc1/Acc10 with ground truth among top-k
  predictions) is required and partial-credit similarity would mask the true exact-match
  rate
- When query molecules have no valid SMILES/molecular graph from which a Morgan fingerprint
  can be computed
edam_topics:
- http://edamontology.org/topic_3345
tools:
- name: SpecTUS
  role: Produces the ranked SMILES candidate predictions per EI-MS spectrum that the
    Tanimoto scoring evaluates
  repo: https://github.com/hejjack/SpecTUS
- name: conda
  role: Environment setup for the evaluation pipeline via trainSpectus_build.yaml
  repo: https://github.com/hejjack/SpecTUS
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:9fa24400ba089b1066f7e89bd373261014572d8f7922724160eead06e465d6bc
  source_papers:
  - doi: 10.48550/arxiv.2502.05114
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hejjack/SpecTUS
  tools:
  - NEIMS
  - SpecTUS
  - conda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3345
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# morgan-fingerprint-tanimoto-similarity-scoring

## Summary

Scores de novo EI-MS structure predictions with a graded, partial-credit metric: per query, the highest Tanimoto similarity between Morgan fingerprints of the ground-truth molecule and each of the top-k predicted SMILES candidates (Sim1/Sim10), averaged over all queries.

## When to use

Use when evaluating ranked SMILES candidate lists from spectra-to-structure inference (e.g. SpecTUS top-1/top-10 predictions) where exact-match Acc1/Acc10 alone is too harsh and partial structural similarity must be quantified.

## When NOT to use

- When only exact-structure reconstruction (Acc1/Acc10 with ground truth among top-k predictions) is required and partial-credit similarity would mask the true exact-match rate
- When query molecules have no valid SMILES/molecular graph from which a Morgan fingerprint can be computed

## Inputs

- Ranked SMILES candidate lists (top-1 and top-10) produced by SpecTUS autoregressive inference on held-out EI-MS spectra
- Ground-truth SMILES structures for the test spectra (e.g. the 5% held-out synthetic test partition of synth1/synth2)

## Outputs

- Per-query best Tanimoto similarity values for k=1 and k=10
- Averaged Sim1 and Sim10 per generator (NEIMS, RASSP) in synthetic_test_metrics.csv (columns Acc1, Acc10, Sim1, Sim10)

## How to apply

Run autoregressive inference to generate ranked SMILES candidates per spectrum (k=1 and k=10), then compute, per query, the maximum Tanimoto similarity between the Morgan fingerprint of the ground-truth structure and the Morgan fingerprint of each top-k candidate; average over all queries to obtain Sim1 and Sim10. Candidates whose SMILES matches ground truth exactly yield Tanimoto = 1.0, so Sim-k upper-bounds Acc-k and captures near-misses. Evaluate separately for the NEIMS-generated and RASSP-generated halves of the synthetic test data and aggregate into synthetic_test_metrics.csv alongside Acc1/Acc10. This mirrors the paper's convention of judging predictions strictly better than hybrid database search in terms of Tanimoto similarity of Morgan fingerprint.

## Related tools

- **SpecTUS** (Produces the ranked SMILES candidate predictions per EI-MS spectrum that the Tanimoto scoring evaluates) — https://github.com/hejjack/SpecTUS
- **conda** (Environment setup for the evaluation pipeline via trainSpectus_build.yaml) — https://github.com/hejjack/SpecTUS

## Evaluation signals

- Any candidate exactly matching the ground-truth SMILES forces the per-query similarity to 1.0, so Sim-k >= Acc-k must hold for both k=1 and k=10
- All Tanimoto values fall in [0, 1]; Sim10 >= Sim1 for every generator split since more candidates can only increase the maximum
- Reproduced Sim1/Sim10 for the NEIMS- and RASSP-generated halves should be consistent with the paper's reported exact-reconstruction rates for synthetic spectra (38% RASSP, 29% NEIMS at Acc1)
- Metrics follow the paper's definitions: highest Morgan-fingerprint Tanimoto similarity per query, averaged over all queries, reported separately per generator

## Limitations

- Tanimoto similarity of Morgan fingerprints rewards substructure overlap but cannot confirm an exact identity claim; a high score does not mean the correct structure was recovered
- SpecTUS performance is sensitive to input spectra quality — poorer on Cayman and MONA test sets than the rigorously curated NIST or SWGDRUG libraries — which propagates into lower Sim scores
- SpecTUS lacks intermediate representations (peak annotations, fragmentation trees) and reference spectra for candidates, so low similarity scores are hard to diagnose manually

## Evidence
