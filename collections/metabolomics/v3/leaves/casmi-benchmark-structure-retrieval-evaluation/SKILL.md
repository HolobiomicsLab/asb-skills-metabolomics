---
name: casmi-benchmark-structure-retrieval-evaluation
description: Use when you have CASMI 2016 positive-mode MS2 spectra with reference
  structure annotations and de novo candidate SMILES from MSNovelist, and need retrieval/top-1
  metrics.
when_to_use_negative:
- 'The spectra are negative-mode: MSNovelist is only trained for positive-mode data
  and CSI:FingerID performance is slightly lower on negative-mode MS2 spectra, so
  results will not be comparable.'
- 'The spectra have fewer than three fragment ions: MSNovelist processes MS2 spectra
  with a minimum of three fragment ions, so candidate generation is unreliable.'
- The task is validating an established spectral-library annotation rather than evaluating
  de novo structure generation on a benchmark set.
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MSNovelist
  role: Generates de novo candidate SMILES structures from MS2 spectra via fingerprint
    prediction plus an encoder-decoder RNN; evaluation scores its outputs against
    CASMI reference structures.
  repo: https://github.com/meowcat/MSNovelist
- name: SIRIUS
  role: Predicts the molecular formula from each CASMI MS2 spectrum as the first step
    of candidate generation.
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: CSI:FingerID
  role: Predicts the structural fingerprint from the MS2 spectrum; this fingerprint
    is the blueprint the encoder-decoder model conditions on and the query fingerprint
    used for Platt-score re-ranking.
- name: Singularity
  role: Container runtime used to build the MSNovelist image (docker://stravsm/msnovelist6)
    and run the evaluation workflow on the job node.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-022-01486-3
    title: 'MSNovelist: de novo structure generation from mass spectra'
  source_content_hash: sha256:7d7235512a04fddec41e8863ed25e4f9e398fe34aa7a1e33f0e4ce8386b94ab9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/meowcat/MSNovelist
  tools:
  - AWS CLI
  - CSI:FingerID
  - MSNovelist
  - SIRIUS
  - Singularity
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# casmi-benchmark-structure-retrieval-evaluation

## Summary

Evaluate de novo structure identification candidates from MSNovelist on the CASMI 2016 positive-mode MS2 challenge spectra, scoring retrieval (correct structure among candidates) and top-1 rank (correct structure ranked first) both overall and on the CASMI-OK subset where database search succeeded.

## When to use

Use when you have CASMI 2016 positive-mode MS2 spectra with reference structure annotations and de novo candidate SMILES from MSNovelist, and need retrieval/top-1 metrics.

## When NOT to use

- The spectra are negative-mode: MSNovelist is only trained for positive-mode data and CSI:FingerID performance is slightly lower on negative-mode MS2 spectra, so results will not be comparable.
- The spectra have fewer than three fragment ions: MSNovelist processes MS2 spectra with a minimum of three fragment ions, so candidate generation is unreliable.
- The task is validating an established spectral-library annotation rather than evaluating de novo structure generation on a benchmark set.

## Inputs

- CASMI 2016 positive-mode MS2 spectra with reference structure annotations (127 spectra)
- Pre-processed SIRIUS 4.4.29 workspace folder (SPECTRA as folder) or .mgf/.ms spectra file
- Trained MSNovelist model weights (e.g. m-36719628-msnovelist-sirius6-weights from the S3 sirius-novelist bucket)
- fingerprint_cache.db (existing or created at DATAFOLDER/fingerprint_cache.db)

## Outputs

- casmi2016_evaluation_metrics.csv with per-spectrum retrieval and top-1 results
- results-RUNID/decode-RUNID.csv and .pkl containing candidate SMILES, scores (score_mod_platt), and ranks
- msnovelist-config-RUNID.yaml recording the run configuration

## How to apply

Pull the CASMI 2016 positive-mode MS2 spectra with reference structures and the trained weights (e.g. m-36719628-msnovelist-sirius6-weights) from the S3 sirius-novelist bucket into cluster scratch space, then pull the MSNovelist Singularity image (docker://stravsm/msnovelist6) on a job node and set .env paths (DATA_LOC, SIF_LOC, CODE_LOC, RESULTS_LOC). Submit run_evaluation.sh via sbatch --array=0-10 so evaluation.py processes each of 10 cross-validation folds on CPU: for every spectrum, SIRIUS predicts the molecular formula, CSI:FingerID predicts the structural fingerprint, the encoder-decoder model generates candidate SMILES, and candidates are re-ranked by fingerprint match using the modified Platt score. Score each spectrum against its CASMI reference structure, counting a retrieval when the correct structure appears among candidates and a top-1 hit when it is ranked first (order results by score_mod_platt descending, or filter rank_score_lim_mod_platt == 1). Aggregate per-fold results over all 127 spectra and separately over the CASMI-OK subset of 47 spectra where database search succeeded, writing casmi2016_evaluation_metrics.csv.

## Related tools

- **MSNovelist** (Generates de novo candidate SMILES structures from MS2 spectra via fingerprint prediction plus an encoder-decoder RNN; evaluation scores its outputs against CASMI reference structures.) — https://github.com/meowcat/MSNovelist
- **SIRIUS** (Predicts the molecular formula from each CASMI MS2 spectrum as the first step of candidate generation.) — https://bio.informatik.uni-jena.de/software/sirius/
- **CSI:FingerID** (Predicts the structural fingerprint from the MS2 spectrum; this fingerprint is the blueprint the encoder-decoder model conditions on and the query fingerprint used for Platt-score re-ranking.)
- **Singularity** (Container runtime used to build the MSNovelist image (docker://stravsm/msnovelist6) and run the evaluation workflow on the job node.)

## Examples

```
singularity build $SCRATCH_PATH/MSNovelist-image/msnovelist.sif docker://stravsm/msnovelist6 && sbatch --array=0-9 run_evaluation.sh
```

## Evaluation signals

- Aggregated metrics reproduce the reported CASMI 2016 performance: 57% retrieval / 26% top-1 overall and 74% retrieval / 64% top-1 on the CASMI-OK subset.
- The CASMI-OK subset contains exactly 47 spectra (of 127 total), matching the subset where database search correctly identified the structure.
- Every generated candidate has a formula consistent with the SIRIUS-predicted formula constraint imposed during decoding.
- Results are ordered by score_mod_platt descendingly; the top-1 candidate satisfies rank_score_lim_mod_platt == 1.
- All 127 spectra are covered across the 10 evaluation folds, with no missing queries in decode-RUNID.csv.

## Limitations

- Formula determination error rates rise to >50% for compounds with m/z up to 800 when considering individual spectra, which propagates into incorrect candidate generation.
- Certain isomers fragment (almost) indistinguishably, so structural elucidation from MS2 data alone is not possible in all cases and retrieval will cap below 100%.
- The model's chemical space is limited to known molecules, which may limit discovery of chemistry extremely different from known molecules.
- Best results require joint consideration of all spectra in a biological sample rather than single-spectrum runs.
- This repo expects a pre-processed SIRIUS 4.4.29 workspace; data processed with the current SIRIUS version cannot be used directly.

## Evidence

- [abstract] On CASMI 2016, MSNovelist correctly predicted 26% and retrieved 57% of structures, recovering 64% of correct database annotations: "Similarly, for the CASMI 2016 challenge, MSNovelist correctly predicted 26% and retrieved 57% of structures"
- [abstract] the candidate structures are re-ranked by calculating the match to the query fingerprint xF using the modified Platt score8.: "the candidate structures are re-ranked by calculating the match to the query fingerprint xF using the modified Platt score8."
- [abstract] First, it relies on SIRIUS and CSI:FingerID to predict a molecular formula xM and a structural fingerprint xF, respectively, from the MS2 spectrum9.: "First, it relies on SIRIUS and CSI:FingerID to predict a molecular formula xM and a structural fingerprint xF, respectively, from the MS2 spectrum9."
- [readme] Order the results by `score_mod_platt`, descendingly, to get the top candidate (or filter by `rank_score_lim_mod_platt == 1`): [evidence span withheld — anchor status: external_doc]
- [discussion] MSNovelist processes MS2 spectra with a minimum of three fragment ions; richer spectra are desirable for best performance.: "processes MS2 spectra with a minimum of three fragment ions; richer spectra are desirable for best performance."
- [discussion] error rates of formula determination by SIRIUS are <10%; however, they increase to >50% for compounds with m/z up to 800: "error rates of formula determination by SIRIUS are <10%; however, they increase to >50% for compounds with m/z up to 800"
