---
name: cross-dataset-model-benchmarking
description: Use when you must quantify how much a model component (e.g. predicted
  fingerprint input) contributes by re-running the identical generation/re-ranking
  pipeline with that component removed on held-out spectra from multiple reference
  datasets.
when_to_use_negative:
- You want the full fingerprint-guided model's performance rather than the fingerprint-input
  ablation — run the guided pipeline instead.
- 'Spectra are in negative-ion mode: MSNovelist is trained only for positive-mode
  data, so benchmark numbers will not reflect true performance.'
- MS2 spectra have fewer than three fragment ions; the pipeline processes a minimum
  of three fragment ions and richer spectra are desirable.
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: MSNovelist
  role: Fingerprint-free de novo SMILES generator; trained encoder–decoder RNN whose
    candidates are validated, dereplicated and re-ranked under the ablation
  repo: https://github.com/meowcat/MSNovelist
- name: SIRIUS
  role: Predicts molecular formula xM from the MS2 spectrum, providing the formula
    constraint under which the generator emits candidates
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: CSI:FingerID
  role: Predicts structural fingerprint xF from the MS2 spectrum; the modified Platt
    score against xF is used to re-rank dereplicated candidates
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-022-01486-3
    title: 'MSNovelist: de novo structure generation from mass spectra'
  source_content_hash: sha256:8679c1a1e4a21d84de95e969d3da853efff70a1a82cdcb10af08b73403f9704f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/meowcat/MSNovelist
  tools:
  - CSI:FingerID
  - MSNovelist
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cross-dataset-model-benchmarking

## Summary

Benchmark a fingerprint-free (naïve) de novo structure generator against the full fingerprint-guided MSNovelist model by running an identical generation-plus-Platt-reranking pipeline over GNPS and CASMI MS2 spectra and measuring structure retrieval and top-1 rates per 10-fold cross-validation split.

## When to use

Use when you must quantify how much a model component (e.g. predicted fingerprint input) contributes by re-running the identical generation/re-ranking pipeline with that component removed on held-out spectra from multiple reference datasets.

## When NOT to use

- You want the full fingerprint-guided model's performance rather than the fingerprint-input ablation — run the guided pipeline instead.
- Spectra are in negative-ion mode: MSNovelist is trained only for positive-mode data, so benchmark numbers will not reflect true performance.
- MS2 spectra have fewer than three fragment ions; the pipeline processes a minimum of three fragment ions and richer spectra are desirable.

## Inputs

- Tandem MS2 spectra in .mgf or SIRIUS format (.ms) with predicted molecular formula xM and query fingerprint xF (SIRIUS/CSI:FingerID outputs)
- Dataset splits for 10-fold cross-validation (e.g. the s6 dataset hosted at s3://sirius-novelist/dataset-s6-202311)
- Fingerprint cache file fingerprint_cache.db for dereplication

## Outputs

- Per-fold, per-spectrum candidate SMILES lists with rank and score_mod_platt values (decode-RUNID.csv / .pkl)
- Aggregated retrieval rate and top-1 rate per dataset variant (GNPS, GNPS-OK, CASMI, CASMI-OK) for the ablated generator

## How to apply

Configure DATA_LOC to a scratch copy of the dataset (e.g. s3://sirius-novelist/dataset-s6-202311), SIF_LOC to msnovelist.sif (built via `singularity build ... docker://stravsm/msnovelist6` on a job node, not the login node), CODE_LOC to the MSNovelist-private checkout, and RESULTS_LOC to the results directory, then launch run_evaluation.sh with `sbatch --array=0-9` so evaluation.py processes each of the 10 cross-validation folds without a GPU. In each fold, generate k candidate SMILES de novo with the fingerprint-free generator under the molecular formula constraint from SIRIUS, validate and dereplicate them, and re-rank with the modified Platt score computed against the query fingerprint xF — the same reranking used by the fingerprint-guided pipeline — so the only difference is the generator input. Report two metrics per dataset (GNPS, GNPS-OK, CASMI, CASMI-OK): retrieval rate (correct structure present among candidates) and top-1 rate (correct structure ranked first by score_mod_platt, descending). The expected ablation gap is fingerprint guidance increasing recovery by 13–31 percentage points (naïve: 31%/17% on GNPS, 37%/35% on GNPS-OK, 52%/24% on CASMI, 56%/51% on CASMI-OK).

## Related tools

- **MSNovelist** (Fingerprint-free de novo SMILES generator; trained encoder–decoder RNN whose candidates are validated, dereplicated and re-ranked under the ablation) — https://github.com/meowcat/MSNovelist
- **SIRIUS** (Predicts molecular formula xM from the MS2 spectrum, providing the formula constraint under which the generator emits candidates) — https://bio.informatik.uni-jena.de/software/sirius/
- **CSI:FingerID** (Predicts structural fingerprint xF from the MS2 spectrum; the modified Platt score against xF is used to re-rank dereplicated candidates)

## Examples

```
docker run --init -v "$(pwd)":/msnovelist-data msnovelist predict.sh 377.mgf
```

## Evaluation signals

- Retrieval rate and top-1 rate per dataset match the reported naïve values: 31%/17% (GNPS), 37%/35% (GNPS-OK), 52%/24% (CASMI), 56%/51% (CASMI-OK).
- Re-ranking must use the identical modified Platt score computed against the query fingerprint xF as the fingerprint-guided pipeline; only the generator input differs.
- All 10 cross-validation folds are evaluated (array job indices 0–9) with no GPU required.
- Per-spectrum outputs expose rank_score_lim_mod_platt and score_mod_platt so candidates can be ordered and top-1 filtered.
- Fingerprint guidance raises recovery of true structures by 13–31 percentage points relative to the naïve generator.

## Limitations

- MSNovelist is trained only for positive-mode MS2 data; negative-mode performance is expected to be lower, mirroring CSI:FingerID behaviour.
- Formula determination error rates rise to >50% for individual spectra with m/z up to 800, which propagates into the formula-constrained generation.
- Structural elucidation is not always possible because certain isomers fragment almost indistinguishably, capping attainable top-1 accuracy.
- The model's conservative chemical space may limit discovery of chemistry extremely different from known molecules.

## Evidence

- [abstract] Naive generation retrieval and top-1 rates on GNPS; fingerprint guidance effect: "naïve generation retrieved only 31% of all correct structures, and 17% were ranked first"
- [abstract] Pipeline definition: formula constraint, validation, dereplication, Platt reranking: "the k structures are validated and dereplicated"
- [abstract] Re-ranking metric definition: "the candidate structures are re-ranked by calculating the match to the query fingerprint xF using the modified Platt score"
- [methods] Infrastructure: singularity image from docker hub: "singularity build $SCRATCH_PATH/MSNovelist-image/msnovelist.sif docker://stravsm/msnovelist6"
- [methods] Code location env var used by run_evaluation.sh: "CODE_LOC=/cluster/home/username/MSNovelist-private"
- [readme] README guidance on result ordering / top-1 extraction: [evidence span withheld — anchor status: external_doc]
- [discussion] Caveat: only positive mode supported: "MSNovelist is currently only trained for positive-mode data."
- [discussion] Caveat: formula errors rise for larger m/z: "they increase to >50% for compounds with m/z up to 800"
