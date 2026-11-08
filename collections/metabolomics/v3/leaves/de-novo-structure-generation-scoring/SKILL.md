---
name: de-novo-structure-generation-scoring
description: Use when tandem MS2 spectra must be annotated with de novo SMILES candidates
  (no spectral-library hit available), and when de novo candidates must be compared
  to the database (PubChem/KEGG) candidate by modified Platt score.
when_to_use_negative:
- Input spectra are negative-mode MS2 data — MSNovelist is trained only for positive-mode
  data and performance is expected to be lower
- Spectra have fewer than three fragment ions — the pipeline requires a minimum of
  three fragment ions
- The structure is already identifiable via a spectral library or database search
  — de novo generation is meant for candidates absent from spectral libraries
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MSNovelist
  role: Encodes the CSI:FingerID fingerprint prediction and generates candidate SMILES
    structures de novo from the MS2 spectrum, then validates, dereplicates, and re-ranks
    them by the modified Platt score
  repo: https://github.com/meowcat/MSNovelist
- name: SIRIUS
  role: Predicts the molecular formula xM from the MS2 spectrum, imposing the formula
    constraint on de novo generation
- name: CSI:FingerID
  role: Predicts the structural fingerprint xF from the MS2 spectrum, which serves
    as the generation blueprint and the basis for modified Platt re-ranking
- name: SIRIUS 6
  role: Current containerized deployment (docker://stravsm/msnovelist6) integrating
    MSNovelist de novo structure suggestions into SIRIUS
  repo: docker://stravsm/msnovelist6
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1038/s41592-022-01486-3
    title: 'MSNovelist: de novo structure generation from mass spectra'
  source_content_hash: sha256:d8743036d0542cc8daed4cfd8311fbfe60ea562d104e30ba6ccea818cdfca962
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/meowcat/MSNovelist
  tools:
  - CSI:FingerID
  - MSNovelist
  - SIRIUS
  - SIRIUS 6
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# de-novo-structure-generation-scoring

## Summary

Generates candidate molecular structures de novo from tandem MS2 spectra via SIRIUS formula prediction, CSI:FingerID fingerprint prediction, and an encoder–decoder RNN, then re-ranks the validated, dereplicated candidates by the modified Platt score against the query fingerprint.

## When to use

Use when tandem MS2 spectra must be annotated with de novo SMILES candidates (no spectral-library hit available), and when de novo candidates must be compared to the best database (PubChem/KEGG) candidate by modified Platt score.

## When NOT to use

- Input spectra are negative-mode MS2 data — MSNovelist is trained only for positive-mode data and performance is expected to be lower
- Spectra have fewer than three fragment ions — the pipeline requires a minimum of three fragment ions
- The structure is already identifiable via a spectral library or database search — de novo generation is meant for candidates absent from spectral libraries

## Inputs

- Tandem MS2 spectra in .mgf, .ms (SIRIUS format), or a pre-processed SIRIUS 4.4.29 workspace (e.g. bryophytes.mgf, 576 consolidated spectra from MTBLS709)
- Optional SIRIUS settings string (default: formula -p qtof structure -d ALL_BUT_INSILICO)
- Molecular formulas with high confidence (≥80% explained peaks, ≥90% explained intensity, ≥0.9 ZODIAC score)
- Structural databases for comparison, e.g. PubChem or KEGG

## Outputs

- Ranked de novo candidate SMILES structures per spectrum, ordered by score_mod_platt (filter rank_score_lim_mod_platt == 1 for the top candidate)
- Per-spectrum table of top de novo candidate score versus best database candidate score
- Results files decode-RUNID.csv and .pkl under $DATAFOLDER/results-RUNID/

## How to apply

Predict the molecular formula xM from the MS2 spectrum with SIRIUS (optionally supply a formula directly) and the structural fingerprint xF with CSI:FingerID; use the fingerprint as a blueprint for an encoder–decoder RNN that generates SMILES sequences under the formula constraint. Validate and dereplicate the k generated structures, then re-rank all candidates (de novo and database) by calculating their match to the query fingerprint xF using the modified Platt score — final re-ranking is stated to be important for best results. Compare per-spectrum the top de novo candidate score with the best database candidate score in a per-spectrum table; on the bryophyte MTBLS709 benchmark the expected outcome is 169 of 224 high-confidence spectra (75%) where the de novo structure outscores the database hit, with feature 377 (m/z 381.1020, C21H16O7) among the seven largest score gaps. Filter spectra before applying the skill using the high-confidence thresholds of ≥80% explained peaks, ≥90% explained intensity, and ≥0.9 ZODIAC score.

## Related tools

- **MSNovelist** (Encodes the CSI:FingerID fingerprint prediction and generates candidate SMILES structures de novo from the MS2 spectrum, then validates, dereplicates, and re-ranks them by the modified Platt score) — https://github.com/meowcat/MSNovelist
- **SIRIUS** (Predicts the molecular formula xM from the MS2 spectrum, imposing the formula constraint on de novo generation)
- **CSI:FingerID** (Predicts the structural fingerprint xF from the MS2 spectrum, which serves as the generation blueprint and the basis for modified Platt re-ranking)
- **SIRIUS 6** (Current containerized deployment (docker://stravsm/msnovelist6) integrating MSNovelist de novo structure suggestions into SIRIUS) — docker://stravsm/msnovelist6

## Examples

```
docker run --init -v "$(pwd)":/msnovelist-data msnovelist predict.sh 377.mgf
```

## Evaluation signals

- On the bryophyte (MTBLS709) benchmark, the comparison must yield 169 of 224 high-confidence spectra (75%) where the de novo structure outscores the best database candidate
- Feature 377 (m/z 381.1020, formula C21H16O7) must appear among the seven spectra with the largest de novo vs. database score gaps, with a flavonoid-core prediction outscoring the database hit
- On GNPS, the skill should retrieve ~45% of correct structures overall and rank ~25% first; the naïve (no fingerprint input) ablation must drop to 31% retrieved / 17% ranked first
- Results CSV must contain the columns score_mod_platt and rank_score_lim_mod_platt, with the top candidate identified by rank_score_lim_mod_platt == 1
- Best incorrect de novo predictions should show a median Tanimoto similarity of ~0.80 to the true structure, between best-in-training-set (0.76) and best incorrect database structure (0.84)

## Limitations

- The model is trained only for positive-mode data; CSI:FingerID performance is slightly lower on negative-mode MS2 spectra and this is expected to carry over to MSNovelist
- The pipeline requires a minimum of three fragment ions per MS2 spectrum, and richer spectra are desirable
- SIRIUS formula determination error rates rise to >50% for compounds with m/z up to 800 when considering individual spectra, so the formula constraint may be wrong at higher masses
- Certain isomers fragment almost indistinguishably, so structural elucidation from this data is not possible in all cases
- The model's chemical space is conservative and may be unable to discover chemistry extremely different from known molecules

## Evidence

- [abstract] Re-rank candidate structures by match to the query fingerprint using the modified Platt score: "the candidate structures are re-ranked by calculating the match to the query fingerprint xF using the modified Platt score8."
- [abstract] Encoder–decoder SMILES generation under formula constraint: "we trained an encoder–decoder RNN model to predict structures in the form of a SMILES sequence from the fingerprint xF under the constraints imposed by the formula xM"
- [discussion] Modified Platt re-ranking is essential for best results: "final re-ranking with the modified Platt score is important for best results"
- [abstract] Bryophyte benchmark: de novo outscores database in 75% of spectra: "For 169 cases (75%), the MSNovelist structure scored higher than the database"
- [readme] Result interpretation: order by modified Platt score column: [evidence span withheld — anchor status: external_doc]
