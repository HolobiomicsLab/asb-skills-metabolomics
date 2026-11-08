---
name: tandem-ms-spectrum-processing
description: Use when starting from raw MS2 spectra (MGF or SIRIUS .ms format) that
  need molecular formula assignment, fingerprint prediction, and de novo candidate
  structure generation and ranking before annotation.
when_to_use_negative:
- Input spectra are negative-mode — MSNovelist is trained only for positive-mode data,
  and CSI:FingerID performance is slightly lower on negative-mode MS2 spectra
- Spectra contain fewer than three fragment ions — the pipeline processes MS2 spectra
  with a minimum of three fragment ions
- The analysis is already at a feature-table or annotated stage; this skill operates
  upstream, on raw MS2 spectra requiring formula and structure assignment
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3318
tools:
- name: SIRIUS
  role: Predicts the molecular formula xM from the MS2 spectrum and provides ZODIAC
    scores used for high-confidence filtering
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: CSI:FingerID
  role: Predicts the structural fingerprint xF from the MS2 spectrum, which serves
    as the blueprint for de novo structure generation and for re-ranking by the modified
    Platt score
- name: MSNovelist
  role: Encoder–decoder RNN that generates SMILES structures de novo from fingerprint
    and formula; packaged as Docker/Singularity containers (stravsm/msnovelist, docker://stravsm/msnovelist6)
  repo: https://github.com/meowcat/MSNovelist
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-022-01486-3
    title: 'MSNovelist: de novo structure generation from mass spectra'
  source_content_hash: sha256:4534a44ac8c26c5a20f718db6e87f5fece6b8979dce47e41d30aa87e3299532c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CSI:FingerID
  - MSNovelist
  - SIRIUS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# tandem-ms-spectrum-processing

## Summary

Process consolidated tandem MS2 spectra into high-confidence molecular formulas, predicted structural fingerprints, and re-ranked de novo candidate structures using SIRIUS, CSI:FingerID, and MSNovelist, with final ordering by the modified Platt score.

## When to use

Use when starting from raw MS2 spectra (MGF or SIRIUS .ms format) that need molecular formula assignment, fingerprint prediction, and de novo candidate structure generation and ranking before annotation.

## When NOT to use

- Input spectra are negative-mode — MSNovelist is trained only for positive-mode data, and CSI:FingerID performance is slightly lower on negative-mode MS2 spectra
- Spectra contain fewer than three fragment ions — the pipeline processes MS2 spectra with a minimum of three fragment ions
- The analysis is already at a feature-table or annotated stage; this skill operates upstream, on raw MS2 spectra requiring formula and structure assignment

## Inputs

- Consolidated MS2 tandem mass spectra in .mgf or SIRIUS .ms format
- A pre-processed SIRIUS 4.4.29 workspace folder (usable directly as MSNovelist input)
- Optional SIRIUS settings string (default: 'formula -p qtof structure -d ALL_BUT_INSILICO')
- Optional known molecular formulas supplied directly, bypassing SIRIUS formula prediction

## Outputs

- SIRIUS results workspace (sirius-RUNID) with per-spectrum molecular formula assignments and ZODIAC scores
- Predicted structural fingerprints (xF) from CSI:FingerID, optionally cached in fingerprint_cache.db
- Validated and dereplicated de novo candidate structures as SMILES (decode-RUNID.csv / .pkl)
- Per-spectrum candidate table with modified Platt scores (score_mod_platt) and rank (rank_score_lim_mod_platt)

## How to apply

Consolidate MS2 spectra and run them through the SIRIUS/MSNovelist pipeline: SIRIUS predicts the molecular formula xM (optionally bypassed by supplying a known formula), CSI:FingerID predicts the structural fingerprint xF, and an encoder–decoder RNN generates SMILES candidates under the formula constraint. Keep only spectra with at least three fragment ions, and for high-confidence formula filtering apply the thresholds of ≥80% explained peaks, ≥90% explained intensity, and ≥0.9 ZODIAC score (this reduced the bryophyte MTBLS709 dataset from 576 consolidated MS2 spectra with m/z < 500 to 224). Validate and dereplicate the k generated structures, then re-rank them by the match to the query fingerprint xF using the modified Platt score — the paper notes this final re-ranking is important for best results. Results are ordered by score_mod_platt descending, or filtered by rank_score_lim_mod_platt == 1, to select the top candidate per spectrum.

## Related tools

- **SIRIUS** (Predicts the molecular formula xM from the MS2 spectrum and provides ZODIAC scores used for high-confidence filtering) — https://bio.informatik.uni-jena.de/software/sirius/
- **CSI:FingerID** (Predicts the structural fingerprint xF from the MS2 spectrum, which serves as the blueprint for de novo structure generation and for re-ranking by the modified Platt score)
- **MSNovelist** (Encoder–decoder RNN that generates SMILES structures de novo from fingerprint and formula; packaged as Docker/Singularity containers (stravsm/msnovelist, docker://stravsm/msnovelist6)) — https://github.com/meowcat/MSNovelist

## Examples

```
docker run --init -v "$(pwd)":/msnovelist-data msnovelist predict.sh 377.mgf
```

## Evaluation signals

- Molecular formulas retained after filtering must satisfy ≥80% explained peaks, ≥90% explained intensity, and ≥0.9 ZODIAC score (e.g., 224 of 576 bryophyte spectra with m/z < 500)
- Every processed spectrum should yield a result row keyed by the 'query' column linking candidates to their source spectrum
- Top candidate per spectrum identified by rank_score_lim_mod_platt == 1 or by descending order of score_mod_platt
- Reproducing the bryophyte analysis should yield 169 of 224 spectra (75%) where the de novo structure outscores the best database (PubChem/KEGG) candidate
- Feature 377 (m/z 381.1020, C21H16O7) should reproduce a flavonoid-core de novo prediction, as in the sample-data 377.mgf example

## Limitations

- MSNovelist is currently only trained for positive-mode data; performance on negative-mode spectra is expected to be lower
- SIRIUS formula determination error rates increase to >50% for compounds with m/z up to 800 when considering individual spectra
- Certain isomers fragment almost indistinguishably, so structural elucidation from MS2 data is not possible in all cases
- For biological datasets, joint consideration of all spectra in a sample is recommended for best results
- The published MSNovelist container relies on an old SIRIUS version; users are directed to SIRIUS 6, where MSNovelist is integrated

## Evidence

- [abstract] Filter steps / thresholds: "we obtained a molecular formula with high confidence (≥80% explained peaks, ≥90% explained intensity, ≥0.9% ZODIAC score)"
- [abstract] Re-ranking step: "the candidate structures are re-ranked by calculating the match to the query fingerprint xF using the modified Platt score"
- [discussion] Minimum fragment ions: "processes MS2 spectra with a minimum of three fragment ions; richer spectra are desirable for best performance."
- [readme] Input formats: [evidence span withheld — anchor status: external_doc]
- [readme] Result ordering: [evidence span withheld — anchor status: external_doc]
