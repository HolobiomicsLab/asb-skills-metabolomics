---
name: rnn-sequence-probability-scoring
description: Use when you need to isolate the contribution of the modified Platt re-ranking
  step by ranking MSNovelist de novo candidates purely by RNN decoder sequence probability
  on MS2 spectra (GNPS or GNPS-OK), e.g. for ablation or benchmarking.
when_to_use_negative:
- When the goal is best identification performance rather than ablation analysis -
  the article states 'final re-ranking with the modified Platt score is important
  for best results', so the default pipeline with Platt re-ranking should be used
  instead
- For negative-mode MS2 spectra - MSNovelist is 'currently only trained for positive-mode
  data' and performance is expected to be lower
- For spectra lacking at least three fragment ions or lacking a confident SIRIUS formula,
  since the RNN candidates depend on formula and fingerprint predictions
edam_topics:
- http://edamontology.org/topic_3336
tools:
- name: MSNovelist
  role: Provides the encoder-decoder RNN that generates the candidate SMILES whose
    raw sequence probabilities are used for ranking; run via its Singularity/Docker
    container with run_evaluation.sh/evaluation.py
  repo: https://github.com/meowcat/MSNovelist
- name: SIRIUS
  role: Predicts the molecular formula constraint xM under which the RNN candidates
    were generated; the stravsm/msnovelist6 image bundles the SIRIUS 6 integration
- name: CSI:FingerID
  role: Predicts the structural fingerprint xF that conditioned candidate generation;
    here used only to define the candidate set, not for re-ranking
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-022-01486-3
    title: 'MSNovelist: de novo structure generation from mass spectra'
  source_content_hash: sha256:ce55ae4396a95666401fbd30b4c7cb60141197e570e478c56479185cfbcd906c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/meowcat/MSNovelist
  tools:
  - CSI:FingerID
  - MSNovelist
  - SIRIUS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# rnn-sequence-probability-scoring

## Summary

An ablation evaluation of MSNovelist that ranks the k validated and dereplicated candidate SMILES by the raw encoder-decoder RNN sequence probability alone, deliberately skipping the modified Platt fingerprint-match re-ranking, to quantify how much re-ranking contributes to top-1 identification rates.

## When to use

Use when you need to isolate the contribution of the modified Platt re-ranking step by ranking MSNovelist de novo candidates purely by RNN decoder sequence probability on MS2 spectra (GNPS or GNPS-OK), e.g. for ablation or benchmarking.

## When NOT to use

- When the goal is best identification performance rather than ablation analysis - the article states 'final re-ranking with the modified Platt score is important for best results', so the default pipeline with Platt re-ranking should be used instead
- For negative-mode MS2 spectra - MSNovelist is 'currently only trained for positive-mode data' and performance is expected to be lower
- For spectra lacking at least three fragment ions or lacking a confident SIRIUS formula, since the RNN candidates depend on formula and fingerprint predictions

## Inputs

- Query MS2 spectra from the GNPS (3,863 spectra) and GNPS-OK evaluation sets
- SIRIUS-predicted molecular formula per spectrum
- CSI:FingerID-predicted structural fingerprint per spectrum
- k validated and dereplicated candidate SMILES per spectrum with their raw decoder sequence probabilities
- Reference structures for each query spectrum

## Outputs

- Sequence-probability-ranked candidate list per spectrum (top-1 structure)
- Top-1 identification rate and overall retrieval rate on GNPS and GNPS-OK
- Median modified Platt score of the RNN-alone rankings (expected: -5,910)
- Tanimoto similarities between predicted and reference fingerprints for best incorrect de novo candidates (expected median: 0.74)

## How to apply

Launch the MSNovelist Singularity container (built from docker://stravsm/msnovelist6) with run_evaluation.sh/evaluation.py via sbatch --array=0-9 so each of the 10 cross-validation folds is evaluated in a CPU-only SLURM job. For every query MS2 spectrum, take the k validated and dereplicated candidate SMILES that the encoder-decoder RNN generated from the SIRIUS-predicted molecular formula and the CSI:FingerID-predicted structural fingerprint, then rank them by raw decoder sequence probability without computing the modified Platt score against the query fingerprint. Compare the top-ranked structure to the reference structure per spectrum and compute top-1 and overall retrieval rates on GNPS and GNPS-OK. As auxiliary analysis, compute Tanimoto similarities between predicted and reference fingerprints for the best incorrect de novo candidates and aggregate the Platt-score statistics of the rankings, which the paper reports alongside the ranking ablation.

## Related tools

- **MSNovelist** (Provides the encoder-decoder RNN that generates the candidate SMILES whose raw sequence probabilities are used for ranking; run via its Singularity/Docker container with run_evaluation.sh/evaluation.py) — https://github.com/meowcat/MSNovelist
- **SIRIUS** (Predicts the molecular formula constraint xM under which the RNN candidates were generated; the stravsm/msnovelist6 image bundles the SIRIUS 6 integration)
- **CSI:FingerID** (Predicts the structural fingerprint xF that conditioned candidate generation; here used only to define the candidate set, not for re-ranking)

## Examples

```
sbatch --array=0-9 run_evaluation.sh
```

## Evaluation signals

- Top-1 identification rate with raw sequence-probability ranking reproduces 19% on GNPS and 39% on GNPS-OK
- Best RNN-alone candidates show a median modified Platt score of -5,910
- Best incorrect de novo candidates under this ranking have a median chemical (Tanimoto) similarity of 0.74
- Results should be worse than the default pipeline (25% top-1 on GNPS), confirming the re-ranking step adds performance rather than the numbers matching the re-ranked pipeline
- All 10 cross-validation folds complete on CPU-only SLURM jobs and per-spectrum results (query column) map back to the input spectra

## Limitations

- Raw RNN sequence-probability ranking is substantially worse than the default modified Platt re-ranking, so this skill is for ablation/analysis, not production identification
- Naive generation without fingerprint input retrieved only 31% of correct structures (17% ranked first), so results depend strongly on the CSI:FingerID fingerprint conditioning
- MSNovelist is trained only on positive-mode data and its conservative chemical space 'might limit the model's ability to discover chemistry extremely different from known molecules'
- The published container relies on an old SIRIUS backend; new users should use the SIRIUS 6 integration (stravsm/msnovelist6) and cannot reuse the original repo directly without retraining

## Evidence

- [abstract] Re-rank candidate structures by match to the query fingerprint using the modified Platt score: "the candidate structures are re-ranked by calculating the match to the query fingerprint xF using the modified Platt score8."
- [discussion] Re-ranking with the modified Platt score is important for best results: "final re-ranking with the modified Platt score is important for best results"
- [readme] Order results by score_mod_platt to get top candidate: [evidence span withheld — anchor status: external_doc]
- [abstract] Generate SMILES structures from fingerprint and formula with an encoder-decoder RNN model: [evidence span withheld — anchor status: not_found]
