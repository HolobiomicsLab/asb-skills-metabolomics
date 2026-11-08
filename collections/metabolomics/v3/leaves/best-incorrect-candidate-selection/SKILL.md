---
name: best-incorrect-candidate-selection
description: Use when comparing annotation methods that return multiple candidate
  structures per MS2 spectrum and you need to characterize the quality of their incorrect
  prediction rather than only top-1 accuracy.
when_to_use_negative:
- The goal is plain top-1 retrieval accuracy or annotation rate — this skill characterizes
  near-miss quality, not the fraction of correct annotations.
- Candidates are not ranked by the modified Platt score or another common fingerprint-match
  score, so 'best incorrect' ordering across candidate sets is not comparable.
- The evaluation lacks true reference structures, so Tanimoto similarity to the correct
  answer cannot be computed.
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: MSNovelist
  role: Source of the de novo candidate structures whose best incorrect predictions
    are scored
  repo: https://github.com/meowcat/MSNovelist
- name: CSI:FingerID
  role: Provides the fingerprint-match (modified Platt) score used to rank candidates
    and identify the best incorrect candidate
- name: RDKit
  role: Computes molecular fingerprints and Tanimoto similarity between the best incorrect
    candidate and the true structure
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-022-01486-3
    title: 'MSNovelist: de novo structure generation from mass spectra'
  source_content_hash: sha256:4bbf32da085b4d9ce8ffb9e0ca0a763701112bf9bd516f76121d4d3e78ea71fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/meowcat/MSNovelist
  tools:
  - CSI:FingerID
  - MSNovelist
  - RDKit
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# best-incorrect-candidate-selection

## Summary

Identifies, for each MS2 spectrum, the best-ranked incorrect candidate from each candidate set (de novo, database, training-set) and quantifies how wrong the top candidate is via Tanimoto similarity to the true structure and the modified Platt score, as done in MSNovelist's GNPS evaluation (Fig. 2c,d).

## When to use

Use when comparing annotation methods that return multiple candidate structures per MS2 spectrum and you need to characterize the quality of their best incorrect prediction rather than only top-1 accuracy.

## When NOT to use

- The goal is plain top-1 retrieval accuracy or annotation rate — this skill characterizes near-miss quality, not the fraction of correct annotations.
- Candidates are not ranked by the modified Platt score or another common fingerprint-match score, so 'best incorrect' ordering across candidate sets is not comparable.
- The evaluation lacks true reference structures, so Tanimoto similarity to the correct answer cannot be computed.

## Inputs

- Per-spectrum candidate sets: MSNovelist de novo predictions, naïve generator (no fingerprint input) predictions, CSI:FingerID database candidates, best-in-training-set and random-training-set candidates
- True reference structures for each evaluated MS2 spectrum (e.g. the 3,863-spectrum GNPS evaluation set)
- Molecular fingerprints (e.g. CSI:FingerID fingerprint vectors or RDKit fingerprints) for each candidate and reference structure
- Candidate rankings (top-1 identification per candidate set, e.g. rank_score_lim_mod_platt == 1 or score_mod_platt sorted descending)

## Outputs

- Per-spectrum table of best incorrect candidate per candidate set with Tanimoto similarity to the true structure and modified Platt score
- Aggregated median Tanimoto similarity and median modified Platt score per candidate set (e.g. candidate_comparison.csv)

## How to apply

For each spectrum in the evaluation set, gather the candidate sets (MSNovelist de novo predictions, the naïve generator without fingerprint input, database search via CSI:FingerID, best-in-training-set and random-training-set references) alongside the true reference structure. From each candidate set pick the highest-ranked candidate whose structure differs from the true structure — this is the best incorrect candidate. Compute Tanimoto similarity between that candidate and the true structure using molecular fingerprints (RDKit) for every spectrum, and compute the modified Platt score, the fingerprint-match score used by CSI:FingerID for re-ranking, for the top-ranked candidates. Aggregate medians of both metrics per candidate set; the article reports medians of 0.80 (de novo) vs 0.76 (best-in-training-set) vs 0.84 (best incorrect database structure) for Tanimoto similarity on the 3,863-spectrum GNPS set, which serve as validation anchors. Export the per-spectrum and aggregated values as a comparison table (candidate_comparison.csv).

## Related tools

- **MSNovelist** (Source of the de novo candidate structures whose best incorrect predictions are scored) — https://github.com/meowcat/MSNovelist
- **CSI:FingerID** (Provides the fingerprint-match (modified Platt) score used to rank candidates and identify the best incorrect candidate)
- **RDKit** (Computes molecular fingerprints and Tanimoto similarity between the best incorrect candidate and the true structure)

## Examples

```
docker run --init -v "$(pwd)":/msnovelist-data msnovelist predict.sh 377.mgf && python -c "import pandas as pd; df=pd.read_csv('msnovelist-data/results-RUNID/decode-RUNID.csv'); top=df[df.rank_score_lim_mod_platt==1]; print(top[['query','score_mod_platt','smiles']])"
```

## Evaluation signals

- Median Tanimoto similarity of best incorrect de novo predictions on the GNPS set should land near 0.80, exceeding the best-in-training-set value (~0.76) and approaching the best incorrect database structure (~0.84).
- Median modified Platt scores for top-ranked candidates should be close across sets (e.g. de novo ≈ database magnitudes, around -5,880 to -5,930).
- Every selected 'best incorrect' candidate must differ structurally from the true structure (non-zero structural mismatch verified by fingerprint comparison).
- The output table must cover one best incorrect candidate per candidate set per spectrum, matching the 3,863-spectrum GNPS evaluation set size.

## Limitations

- High Tanimoto similarity of incorrect candidates means errors are often near-misses among very similar structures; certain isomers fragment almost indistinguishably, so distinguishing them from MS2 alone is not always possible.
- The modified Platt score is a fingerprint-match metric from CSI:FingerID; candidate sets ranked by other scores (e.g. CFM-ID simulated-spectrum match) are not directly comparable.
- In the article this analysis was done on positive-mode MS2 spectra; MSNovelist is currently only trained for positive-mode data.

## Evidence

- [abstract] Best incorrect de novo predictions had median Tanimoto similarity 0.80, higher than best-in-training-set (0.76) and nearly as high as best incorrect database structure (0.84): "The best incorrect de novo predictions (median 0.80) scored higher than the best-in-training set (0.76) and nearly as high as the best incorrect database structure (0.84)"
- [readme] MSNovelist README instructs to order results by score_mod_platt descendingly to get top candidate: [evidence span withheld — anchor status: external_doc]
- [discussion] Re-ranking generated candidates with the modified Platt score: "final re-ranking with the modified Platt score is important for best results"
- [abstract] Article abstract describes GNPS evaluation set size: "In an evaluation with 3,863 MS2 spectra from the Global Natural Product Social Molecular Networking site"
