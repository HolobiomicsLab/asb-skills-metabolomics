---
name: ms2-spectra-de-novo-structure-generation
description: Use when you have positive-mode MS2 spectra (MGF or SIRIUS .ms) with
  no library match, need de novo candidate structures, or want to evaluate whether
  de novo candidates rival database-search annotations.
when_to_use_negative:
- Input spectra are negative-mode MS2 — the model is only trained for positive-mode
  data, so performance is expected to be lower
- You only need formula annotation or compound-class level insight — SIRIUS/CANOPUS
  alone suffice and formula errors can exceed 50% for individual spectra up to m/z
  800
- Spectra have fewer than three fragment ions or isomers fragment indistinguishably,
  making structural elucidation not possible in all cases
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3174
tools:
- name: MSNovelist
  role: Encoder–decoder RNN that generates candidate SMILES from the CSI:FingerID
    fingerprint under formula constraints; distributed as Docker/Singularity container
    (stravsm/msnovelist6)
  repo: https://github.com/meowcat/MSNovelist
- name: SIRIUS
  role: Predicts the molecular formula xM from the MS2 spectrum; with SIRIUS 6, MSNovelist
    de novo suggestions are integrated directly into the SIRIUS GUI/API
- name: CSI:FingerID
  role: Predicts the structural fingerprint xF from the MS2 spectrum that acts as
    the blueprint for molecule generation
- name: Singularity
  role: Container runtime used on HPC (build from docker://stravsm/msnovelist6 on
    a job node, redirect SINGULARITY_CACHEDIR to cluster scratch)
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41592-022-01486-3
    title: 'MSNovelist: de novo structure generation from mass spectra'
  source_content_hash: sha256:d56434c8b341f9aaec522efc2f2b417ee9740948a3ac9333be13a31e2d69eafd
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
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# ms2-spectra-de-novo-structure-generation

## Summary

De novo molecular structure identification from tandem MS (MS2) spectra: SIRIUS predicts the molecular formula, CSI:FingerID predicts a structural fingerprint, and an encoder–decoder RNN (MSNovelist) generates candidate SMILES constrained by formula, re-ranked by fingerprint match using the modified Platt score.

## When to use

Use when you have positive-mode MS2 spectra (MGF or SIRIUS .ms) with no library match, need de novo candidate structures, or want to evaluate whether de novo candidates rival database-search annotations.

## When NOT to use

- Input spectra are negative-mode MS2 — the model is only trained for positive-mode data, so performance is expected to be lower
- You only need formula annotation or compound-class level insight — SIRIUS/CANOPUS alone suffice and formula errors can exceed 50% for individual spectra up to m/z 800
- Spectra have fewer than three fragment ions or isomers fragment indistinguishably, making structural elucidation not possible in all cases

## Inputs

- Positive-mode MS2 spectra in MGF or SIRIUS .ms format (minimum three fragment ions per spectrum)
- A pre-processed SIRIUS 4.4.29 workspace folder (or raw spectra processed first with SIRIUS, default settings 'formula -p qtof structure -d ALL_BUT_INSILICO')
- Reference structure annotations per spectrum (for benchmarking, e.g. GNPS 3,863-spectra evaluation set or CASMI 2016 challenge spectra)

## Outputs

- Candidate SMILES structures per spectrum ranked by score_mod_platt (results-RUNID/decode-RUNID.csv and .pkl)
- SIRIUS formula/fingerprint predictions stored in DATAFOLDER/sirius-RUNID
- Aggregate metrics: overall retrieval rate and top-1 accuracy (e.g. 45% retrieval / 25% top-1 on GNPS; 57% retrieval / 26% top-1 on CASMI 2016)

## How to apply

Run SIRIUS on each MS2 spectrum to determine the molecular formula (optionally bypassed with external knowledge; needs ≥3 fragment ions per spectrum), then predict the structural fingerprint with CSI:FingerID. Feed the fingerprint (as blueprint) and formula constraints into the MSNovelist encoder–decoder RNN to generate k SMILES candidates, which are validated and dereplicated. Re-rank the candidates by calculating the match to the query fingerprint xF using the modified Platt score — this final re-ranking is important for best results; order results by 'score_mod_platt' descending or filter by 'rank_score_lim_mod_platt == 1'. Benchmark against reference structures by counting a retrieval when the correct structure appears among candidates and a top-1 hit when it ranks first.

## Related tools

- **MSNovelist** (Encoder–decoder RNN that generates candidate SMILES from the CSI:FingerID fingerprint under formula constraints; distributed as Docker/Singularity container (stravsm/msnovelist6)) — https://github.com/meowcat/MSNovelist
- **SIRIUS** (Predicts the molecular formula xM from the MS2 spectrum; with SIRIUS 6, MSNovelist de novo suggestions are integrated directly into the SIRIUS GUI/API)
- **CSI:FingerID** (Predicts the structural fingerprint xF from the MS2 spectrum that acts as the blueprint for molecule generation)
- **Singularity** (Container runtime used on HPC (build from docker://stravsm/msnovelist6 on a job node, redirect SINGULARITY_CACHEDIR to cluster scratch))

## Examples

```
docker run --init -v "$(pwd)":/msnovelist-data msnovelist predict.sh 377.mgf
```

## Evaluation signals

- On the GNPS evaluation set (3,863 spectra), aggregated metrics should reach ~45% retrieval and ~25% top-1 accuracy against the reported values
- On the CASMI 2016 challenge (127 positive-mode spectra), expect ~57% retrieval / 26% top-1 overall and ~74% retrieval / 64% top-1 on the CASMI-OK subset of 47 spectra
- In the results CSV, correct top candidates satisfy rank_score_lim_mod_platt == 1 and are ordered by score_mod_platt descending
- Best incorrect de novo predictions should show median Tanimoto similarity around 0.80 to the true structure (higher than best-in-training-set at 0.76)
- Fingerprint cache file fingerprint_cache.db is created/reused in the data folder and msnovelist-config-RUNID.yaml records the used configuration

## Limitations

- The SIRIUS formula prediction may be bypassed but the method relies on existing methods or external knowledge for molecular formula determination; formula errors rise to >50% for individual spectra up to m/z 800
- Only trained for positive-mode data; CSI:FingerID performance is slightly lower on negative-mode MS2 spectra and this will likely be reflected in MSNovelist performance
- Processes MS2 spectra with a minimum of three fragment ions; conservative chemical space may limit discovery of chemistry extremely different from known molecules, and indistinguishable isomer fragmentation makes structural elucidation impossible in some cases
- The published version relied on an old SIRIUS backend that is no longer running — use SIRIUS 6 integration or the stravsm/msnovelist6 container instead
- Joint consideration of all spectra in a biological sample is recommended for best results with biological datasets

## Evidence

- [abstract] First, it relies on SIRIUS and CSI:FingerID to predict a molecular formula xM and a structural fingerprint xF, respectively, from the MS2 spectrum: "First, it relies on SIRIUS and CSI:FingerID to predict a molecular formula xM and a structural fingerprint xF, respectively, from the MS2 spectrum"
- [methods] Modified Platt re-ranking: "the candidate structures are re-ranked by calculating the match to the query fingerprint xF using the modified Platt score"
- [readme] Container CLI invocation: [evidence span withheld — anchor status: external_doc]
- [readme] SIRIUS 6 integration note: "The published version of MSNovelist relied on an old version of SIRIUS for which the backend is not running anymore."
