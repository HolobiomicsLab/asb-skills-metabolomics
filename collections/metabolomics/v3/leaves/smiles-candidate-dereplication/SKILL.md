---
name: smiles-candidate-dereplication
description: Use when the RNN generator returns multiple decoded SMILES strings per
  spectrum (e.g. k candidate structures under a fixed molecular formula) and a unique
  candidate list is needed before Platt-score re-ranking.
when_to_use_negative:
- Dereplicating candidates that come from database search (e.g. PubChem, KEGG structures
  retrieved via CSI:FingerID) — this skill applies to de novo generated structures
  only.
- Input is a single fixed structure per query (already dereplicated), e.g. training-set
  structures.
- Running fingerprint prediction (CSI:FingerID) rather than structure generation —
  there are no generated SMILES to deduplicate.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_0154
tools:
- name: MSNovelist
  role: encoder–decoder RNN that generates the k candidate SMILES structures to be
    dereplicated; dereplication is part of the MSNovelist de novo pipeline
  repo: https://github.com/meowcat/MSNovelist
- name: SIRIUS
  role: predicts molecular formula xM from MS2, which constrains which candidate SMILES
    are valid; dereplicated candidates are evaluated under this formula constraint
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: CSI:FingerID
  role: predicts structural fingerprint xF that drives generation and re-ranking;
    dereplicated candidates are re-scored against this fingerprint
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-022-01486-3
    title: 'MSNovelist: de novo structure generation from mass spectra'
  source_content_hash: sha256:87549403dda9415ef3c303a1f67ccd506c2979e8b1010259b236cd8c95ab7fcb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/meowcat/MSNovelist
  tools:
  - CSI:FingerID
  - MSNovelist
  - SIRIUS
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# smiles-candidate-dereplication

## Summary

Deduplicate the k de novo SMILES candidates produced by MSNovelist's encoder–decoder RNN so each unique chemical structure is kept once before fingerprint-based re-ranking.

## When to use

Use when the RNN generator returns multiple decoded SMILES strings per spectrum (e.g. k candidate structures under a fixed molecular formula) and a unique candidate list is needed before Platt-score re-ranking.

## When NOT to use

- Dereplicating candidates that come from database search (e.g. PubChem, KEGG structures retrieved via CSI:FingerID) — this skill applies to de novo generated structures only.
- Input is a single fixed structure per query (already dereplicated), e.g. training-set structures.
- Running fingerprint prediction (CSI:FingerID) rather than structure generation — there are no generated SMILES to deduplicate.

## Inputs

- k generated SMILES candidate structures per query spectrum (from the RNN decoder, e.g. results-RUNID/decode-RUNID.csv)

## Outputs

- Unique, validated SMILES candidate list per spectrum with mod_platt / score_mod_platt re-ranking scores and rank fields

## How to apply

After the encoder–decoder RNN decodes the fingerprint xF into k SMILES sequences under the molecular formula constraint xM, the candidate pool can contain duplicates — repeated decodes often converge on the same molecule, including 'prolific compounds' that appear repeatedly across spectra. Dereplication collapses the k generated structures onto a unique set: SMILES strings are canonicalized, invalid or formula-violating structures are dropped during validation, and identical structures are merged so each candidate is represented once. The unique candidate list is then handed to the re-ranking step, where the modified Platt score is computed for each candidate against the query fingerprint xF and used to order candidates so the top hit (rank_score_lim_mod_platt == 1) can be selected. Rationale: ranking the redundant pre-dereplication pool would inflate apparent recovery and distort the top-1 statistics (e.g. 25% top-1 correct on GNPS, 45% overall retrieval), so deduplication must precede re-ranking to make per-spectrum candidate counts and rank statistics meaningful.

## Related tools

- **MSNovelist** (encoder–decoder RNN that generates the k candidate SMILES structures to be dereplicated; dereplication is part of the MSNovelist de novo pipeline) — https://github.com/meowcat/MSNovelist
- **SIRIUS** (predicts molecular formula xM from MS2, which constrains which candidate SMILES are valid; dereplicated candidates are evaluated under this formula constraint) — https://bio.informatik.uni-jena.de/software/sirius/
- **CSI:FingerID** (predicts structural fingerprint xF that drives generation and re-ranking; dereplicated candidates are re-scored against this fingerprint)

## Examples

```
docker run --init -v "$(pwd)":/msnovelist-data msnovelist predict.sh 377.mgf
```

## Evaluation signals

- Unique-candidate count per spectrum should equal the number of distinct valid SMILES after canonicalization — no duplicate inchikeys/SMILES remain in the candidate list.
- Order the results by score_mod_platt descendingly and the top candidate should correspond to rank_score_lim_mod_platt == 1, confirming dereplication happened before re-ranking.
- Every surviving candidate's SMILES must decode to a structure consistent with the predicted formula xM from SIRIUS.
- Re-running dereplication on the same decode output (decode-RUNID.csv / .pkl) should produce an identical unique-candidate set (deterministic dedup).
- On GNPS the pipeline downstream of dereplication should show ~45% overall retrieval and ~25% top-1 correct, with naive (fingerprint-free) generation dropping to ~31%/17% — large deviations suggest a dereplication/ranking bug.

## Limitations

- Certain isomers fragment almost indistinguishably, so structural elucidation is not possible in all cases and dereplication cannot resolve stereoisomer/isomer ambiguity beyond SMILES identity.
- The model's chemical space is conservative and anchored to known molecules, limiting its ability to propose structures very different from known chemistry.
- Model is trained only for positive-mode MS2 data; negative-mode performance is expected to be lower.
- Formula determination error rates rise to >50% for individual spectra up to m/z 800, so the formula constraint used during generation/validation can be wrong.
- Requires at least three fragment ions per MS2 spectrum; richer spectra are desirable.
- The published version of MSNovelist relied on an old version of SIRIUS for which the backend is not running anymore; users should use SIRIUS 6 / MSNovelist integration instead.

## Evidence

- [methods] Dereplication of the k generated structures collapses duplicates before fingerprint re-ranking.: "the k structures are validated and dereplicated"
- [methods] Re-ranking of dereplicated candidates uses the modified Platt score against the query fingerprint.: "the candidate structures are re-ranked by calculating the match to the query fingerprint xF using the modified Platt score8."
- [readme] Ordering the results by score_mod_platt descendingly gives the top candidate.: [evidence span withheld — anchor status: external_doc]
- [abstract] Encoder–decoder RNN decodes fingerprint xF to SMILES under formula constraint.: "we trained an encoder–decoder RNN model to predict structures in the form of a SMILES sequence from the fingerprint xF under the constraints imposed by the formula xM"
- [abstract] Fingerprint guidance increased retrieval of true structures by 13 to 31 percentage points relative to naive generation.: [evidence span withheld — anchor status: not_found]
