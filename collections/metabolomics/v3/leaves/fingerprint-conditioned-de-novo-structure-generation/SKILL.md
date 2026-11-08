---
name: fingerprint-conditioned-de-novo-structure-generation
description: Use when a tandem MS2 spectrum has a confidently predicted formula and
  fingerprint (SIRIUS/CSI:FingerID) but no convincing database candidate, and you
  want de novo SMILES candidates instead of library lookup.
when_to_use_negative:
- A convincing database annotation already exists — database search with CSI:FingerID
  ranked the correct structure first for 39% of GNPS instances, so prefer lookup before
  de novo generation.
- The spectrum was acquired in negative mode — MSNovelist is currently only trained
  for positive-mode data and negative-mode performance is expected to be lower.
- The spectrum has fewer than three fragment ions or very low-confidence formula prediction
  (error rates rise to >50% for compounds with m/z up to 800 considering individual
  spectra).
edam_operation: http://edamontology.org/operation_3454
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MSNovelist
  role: Generates de novo SMILES candidate structures from the CSI:FingerID fingerprint
    xF under formula constraints via an encoder-decoder RNN, then validates, dereplicates
    and re-ranks them by modified Platt score.
  repo: https://github.com/meowcat/MSNovelist
- name: SIRIUS
  role: Predicts the molecular formula xM from the MS2 spectrum (may be bypassed if
    the formula is provided directly).
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: CSI:FingerID
  role: Predicts the structural fingerprint xF from the MS2 spectrum that serves as
    the input blueprint for structure generation.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-022-01486-3
    title: 'MSNovelist: de novo structure generation from mass spectra'
  source_content_hash: sha256:068b52baa205a065af157465eee36e67633abcd472764f3ca0d649d3f8b4f3df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/meowcat/MSNovelist
  tools:
  - CSI:FingerID
  - MSNovelist
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3454
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# fingerprint-conditioned-de-novo-structure-generation

## Summary

De novo molecular structure generation with MSNovelist: a structural fingerprint predicted from an MS2 spectrum by CSI:FingerID acts as a blueprint fed to an encoder-decoder RNN that emits candidate SMILES, which are validated, dereplicated, and re-ranked by the modified Platt score.

## When to use

Use when a tandem MS2 spectrum has a confidently predicted formula and fingerprint (SIRIUS/CSI:FingerID) but no convincing database candidate, and you want de novo SMILES candidates instead of library lookup.

## When NOT to use

- A convincing database annotation already exists — database search with CSI:FingerID ranked the correct structure first for 39% of GNPS instances, so prefer lookup before de novo generation.
- The spectrum was acquired in negative mode — MSNovelist is currently only trained for positive-mode data and negative-mode performance is expected to be lower.
- The spectrum has fewer than three fragment ions or very low-confidence formula prediction (error rates rise to >50% for compounds with m/z up to 800 considering individual spectra).

## Inputs

- MS2 tandem mass spectrum (MGF, MS/SIRIUS format, or a pre-processed SIRIUS 4.4.29 workspace)
- Predicted molecular formula xM from SIRIUS (optional external knowledge)
- Predicted structural fingerprint xF from CSI:FingerID
- Fingerprint cache (fingerprint_cache.db) optionally reused across runs

## Outputs

- k validated, dereplicated candidate structures in SMILES format
- Ranked candidate table (decode-RUNID.csv and .pkl) with score_mod_platt and rank_score_lim_mod_platt columns
- SIRIUS intermediate results in DATAFOLDER/sirius-RUNID and configuration in msnovelist-config-RUNID.yaml

## How to apply

Run SIRIUS on the MS2 spectrum to predict a molecular formula xM (or supply the formula directly to bypass SIRIUS), then run CSI:FingerID to predict a structural fingerprint xF from the spectrum. Feed xF and xM to the encoder-decoder RNN model (retrained on 1,232,184 structures from HMDB 4.0, COCONUT and DSSTox together with 14,047 predicted fingerprints used to parametrize fingerprint simulation) so it generates k SMILES strings under the constraints imposed by the formula xM. Validate and dereplicate the k generated structures, then re-rank candidates by calculating the match to the query fingerprint xF using the modified Platt score; order results by score_mod_platt descending (or filter rank_score_lim_mod_platt == 1) to get the top candidate. Note that final re-ranking with the modified Platt score is important for best results.

## Related tools

- **MSNovelist** (Generates de novo SMILES candidate structures from the CSI:FingerID fingerprint xF under formula constraints via an encoder-decoder RNN, then validates, dereplicates and re-ranks them by modified Platt score.) — https://github.com/meowcat/MSNovelist
- **SIRIUS** (Predicts the molecular formula xM from the MS2 spectrum (may be bypassed if the formula is provided directly).) — https://bio.informatik.uni-jena.de/software/sirius/
- **CSI:FingerID** (Predicts the structural fingerprint xF from the MS2 spectrum that serves as the input blueprint for structure generation.)

## Examples

```
docker run --init -v "$(pwd)":/msnovelist-data stravsm/msnovelist predict.sh 377.mgf
```

## Evaluation signals

- The candidate table contains non-empty, chemically valid SMILES strings consistent with the predicted formula xM.
- Ranking follows score_mod_platt descendingly; the top candidate satisfies rank_score_lim_mod_platt == 1.
- On GNPS evaluation data, correct structures were retrieved for 45% and ranked first for 25% of instances — deviations far below these rates suggest a pipeline misconfiguration.
- For the bryophyte dataset, the MSNovelist structure should score higher than the best database candidate in 169 of 224 spectra (75%).
- Results files decode-RUNID.csv and .pkl appear in DATAFOLDER/results-RUNID, and SIRIUS intermediates in DATAFOLDER/sirius-RUNID.

## Limitations

- Certain isomers fragment (almost) indistinguishably, so structural elucidation from MS2 data is not possible in all cases.
- The model is trained only for positive-mode data; negative-mode performance will likely be lower.
- Formula determination errors rise to >50% for compounds with m/z up to 800 when considering individual spectra.
- The conservative chemical space may limit the model's ability to discover chemistry extremely different from known molecules.
- Best results require richer spectra (minimum three fragment ions) and joint consideration of all spectra in a biological sample.

## Evidence

- [abstract] encoder-decoder RNN generates SMILES from fingerprint xF: "we trained an encoder–decoder RNN model to predict structures in the form of a SMILES sequence from the fingerprint xF under the constraints imposed by the formula xM"
- [discussion] CSI:FingerID fingerprint as blueprint: "structural fingerprint predictions from MS2 spectra (by CSI:FingerID) directly encode structural information and may act as a blueprint for a molecule"
- [discussion] re-ranking with modified Platt score is important: "final re-ranking with the modified Platt score is important for best results"
- [readme] order results by score_mod_platt: [evidence span withheld — anchor status: external_doc]
- [abstract] RNN training data: "the RNN model was trained on a dataset of 1,232,184 chemical structures compiled from the databases HMDB (4.0)29, COCONUT30 and DSSTox31"
