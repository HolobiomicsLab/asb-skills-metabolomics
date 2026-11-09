---
name: fingerprint-based-candidate-re-ranking
description: Use when you have multiple candidate structures (de novo SMILES or database
  hits) for a positive-mode MS2 spectrum and must rank them against a CSI:FingerID-predicted
  query fingerprint to pick the top candidate.
when_to_use_negative:
- 'Input spectra are negative-mode: MSNovelist is trained only on positive-mode data,
  and re-ranking quality will degrade with the lower CSI:FingerID negative-mode fingerprint
  performance'
- Spectra have fewer than three fragment ions, so formula/fingerprint prediction is
  unreliable and re-ranking has no meaningful blueprint
- You only need database search results (e.g. CSI:FingerID structure search alone)
  without de novo candidates to rank
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MSNovelist
  role: Generates k candidate SMILES structures and re-ranks them by fingerprint match
    using the modified Platt score; results carry score_mod_platt for ordering
  repo: https://github.com/meowcat/MSNovelist
- name: CSI:FingerID
  role: Predicts the query structural fingerprint xF from the MS2 spectrum that serves
    as the re-ranking blueprint
- name: SIRIUS
  role: Predicts the molecular formula xM constraining candidate generation; SIRIUS
    6 integrates MSNovelist de novo suggestions directly
  repo: https://bio.informatik.uni-jena.de/software/sirius/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-022-01486-3
    title: 'MSNovelist: de novo structure generation from mass spectra'
  source_content_hash: sha256:94a3cf320ea16a923579c6b05caefc1eb20fa3ab849099539eadcb9c3cb03881
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/meowcat/MSNovelist
  tools:
  - AWS CLI
  - CSI:FingerID
  - MSNovelist
  - SIRIUS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# fingerprint-based-candidate-re-ranking

## Summary

Re-ranks de novo or database candidate structures by scoring their predicted structural fingerprint against the query fingerprint predicted by CSI:FingerID from the MS2 spectrum, using the modified Platt score. This is the step that orders MSNovelist's generated SMILES candidates so the top rank reflects fingerprint compatibility rather than generation order.

## When to use

Use when you have multiple candidate structures (de novo SMILES or database hits) for a positive-mode MS2 spectrum and must rank them against a CSI:FingerID-predicted query fingerprint to pick the top candidate.

## When NOT to use

- Input spectra are negative-mode: MSNovelist is trained only on positive-mode data, and re-ranking quality will degrade with the lower CSI:FingerID negative-mode fingerprint performance
- Spectra have fewer than three fragment ions, so formula/fingerprint prediction is unreliable and re-ranking has no meaningful blueprint
- You only need database search results (e.g. CSI:FingerID structure search alone) without de novo candidates to rank

## Inputs

- Positive-mode MS2 spectra (MGF, SIRIUS .ms, or SIRIUS workspace)
- SIRIUS-predicted molecular formula (xM) per spectrum
- CSI:FingerID-predicted structural fingerprint (xF) per spectrum
- k generated/dereplicated candidate structures as SMILES (e.g. results-RUNID/decode-RUNID.csv)

## Outputs

- Candidate structures re-ranked by modified Platt match to the query fingerprint
- Result table column score_mod_platt (descending) with top candidate at rank_score_lim_mod_platt == 1
- Retrieval % and top-1 % metrics per spectrum set (e.g. casmi2016_evaluation_metrics.csv)

## How to apply

First predict the molecular formula xM with SIRIUS and the structural fingerprint xF with CSI:FingerID from the MS2 spectrum; xF acts as the structural blueprint. Generate (or collect) k candidate structures and validate/dereplicate them. For each candidate, compute its fingerprint match to xF using the modified Platt score, then sort candidates descending by score_mod_platt; the article stresses this final re-ranking is important for best results. Count a retrieval when the CASMI/GNPS reference structure appears among candidates and a top-1 hit when it has rank_score_lim_mod_platt == 1; aggregate over all spectra (e.g. 127 CASMI 2016 spectra, plus the 47-spectrum CASMI-OK subset) to compute retrieval % and top-1 %.

## Related tools

- **MSNovelist** (Generates k candidate SMILES structures and re-ranks them by fingerprint match using the modified Platt score; results carry score_mod_platt for ordering) — https://github.com/meowcat/MSNovelist
- **CSI:FingerID** (Predicts the query structural fingerprint xF from the MS2 spectrum that serves as the re-ranking blueprint)
- **SIRIUS** (Predicts the molecular formula xM constraining candidate generation; SIRIUS 6 integrates MSNovelist de novo suggestions directly) — https://bio.informatik.uni-jena.de/software/sirius/

## Examples

```
docker run --init -v "$(pwd)":/msnovelist-data msnovelist predict.sh 377.mgf  # then order decode-RUNID.csv by score_mod_platt descending
```

## Evaluation signals

- Re-ranked table has score_mod_platt per candidate and exactly one candidate with rank_score_lim_mod_platt == 1 per spectrum (query column maps each row to its spectrum)
- On CASMI 2016 (127 positive-mode spectra), aggregated top-1/retrieval should approach 26% top-1 / 57% retrieval overall and 64% top-1 / 74% retrieval on the CASMI-OK subset of 47 spectra
- On the 3,863-spectrum GNPS benchmark, retrieval ~45% and top-1 ~25%; recovery of correct database annotations ~61-64%
- Naive re-ranking ablation check: without fingerprint-based re-ranking, GNPS retrieval drops to ~31% and top-1 to ~17%
- Top incorrect predictions should show median Tanimoto similarity ~0.80 to the reference, between best-in-training (0.76) and best incorrect database structure (0.84)

## Limitations

- Isomers with (almost) indistinguishable fragmentation cannot be elucidated, so re-ranking cannot resolve them regardless of score
- Re-ranking operates on formula determination by SIRIUS, whose per-spectrum error rates rise to >50% for compounds up to m/z 800, propagating errors into fingerprint prediction
- Scores should be interpreted over the conservative known chemical space the model learned from HMDB 4.0/COCONUT/DSSTox; chemistry very different from training data may not surface as top-ranked candidates

## Evidence

- [abstract] the candidate structures are re-ranked by calculating the match to the query fingerprint xF using the modified Platt score8.: "the candidate structures are re-ranked by calculating the match to the query fingerprint xF using the modified Platt score8."
- [discussion] final re-ranking with the modified Platt score is important for best results: "final re-ranking with the modified Platt score is important for best results"
- [discussion] structural fingerprint predictions from MS2 spectra (by CSI:FingerID) directly encode structural information and may act as a blueprint: "structural fingerprint predictions from MS2 spectra (by CSI:FingerID) directly encode structural information and may act as a blueprint"
- [readme] Order the results by `score_mod_platt`, descendingly, to get the top candidate (or filter by `rank_score_lim_mod_platt == 1`): [evidence span withheld — anchor status: external_doc]
