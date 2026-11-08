---
name: casmi2022-positive-mode-evaluation
description: 'Use when evaluating a formula-annotation model on prospective challenge
  spectra: you have CASMI 2022 mzML files, formula candidates, and need head-to-head
  accuracy comparison vs SIRIUS without manual curation.'
when_to_use_negative:
- The spectra are negative-ion mode — models were trained only for positive-mode data,
  so CASMI 2022 positive-mode benchmarks do not transfer
- The goal is compound structure identification rather than formula/adduct annotation,
  since MIST-CF does not reference spectrum databases or predict fingerprints
- Low-resolution MS/MS data is the target — the quickstart public model may be less
  performant than the NIST20-trained model, particularly for Orbitrap or higher resolution
  data
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3343
tools:
- name: MIST-CF
  role: Pretrained energy-based transformer that ranks chemical formula and adduct
    assignments per positive-mode MS/MS spectrum without database lookup
  repo: https://github.com/samgoldman97/mist-cf
- name: SIRIUS
  role: Provides 'decomp' dynamic-programming formula candidates for MIST-CF scoring,
    and is run end-to-end (CSI:FingerID) as the out-of-the-box baseline
  repo: https://github.com/boecker-lab/sirius/releases/download/
- name: MZmine
  role: Feature detection and export of the 304 positive-mode CASMI 2022 MS/MS spectra
    from challenge mzML files
- name: CSI:FingerID
  role: SIRIUS's structure-to-fingerprint search used as the baseline whose top-1
    formula accuracy (0.516) is compared against MIST-CF (0.842)
  repo: https://github.com/sirius-ms/sirius
- name: analysis/evaluate_pred.py
  role: Scoring script that computes joint formula+adduct and formula-only accuracy
    against CASMI 2022 ground truth and writes results_table.tsv
  repo: https://github.com/samgoldman97/mist-cf
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:312553bad1e22d4f60fcda34b66c2685394a8ca122f826d9cc641dfa95869ee3
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/samgoldman97/mist-cf
  tools:
  - CSI:FingerID
  - MIST-CF
  - MZmine
  - SIRIUS
  - analysis/evaluate_pred.py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3343
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# casmi2022-positive-mode-evaluation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Benchmark MIST-CF de novo chemical formula/adduct prediction against out-of-the-box SIRIUS/CSI:FingerID on the 304 positive-mode CASMI 2022 challenge spectra, scoring joint formula+adduct accuracy and top-1 formula accuracy against ground-truth assignments.

## When to use

Use when evaluating a formula-annotation model on prospective challenge spectra: you have CASMI 2022 mzML files, formula candidates, and need head-to-head accuracy comparison vs SIRIUS without manual curation.

## When NOT to use

- The spectra are negative-ion mode — models were trained only for positive-mode data, so CASMI 2022 positive-mode benchmarks do not transfer
- The goal is compound structure identification rather than formula/adduct annotation, since MIST-CF does not reference spectrum databases or predict fingerprints
- Low-resolution MS/MS data is the target — the quickstart public model may be less performant than the NIST20-trained model, particularly for Orbitrap or higher resolution data

## Inputs

- CASMI 2022 challenge mzML files (casmi22.zip from Zenodo record 8151490)
- 304 positive-mode MS/MS spectra exported as MGF (data/demo_specs.mgf-style)
- Pretrained MIST-CF model checkpoint (quickstart/download_model.sh)
- SIRIUS 5.x binary with SIRIUS_PATH set, for 'decomp' formula candidate generation
- CASMI 2022 ground-truth formula and adduct assignments

## Outputs

- Per-spectrum top-1 formula and adduct predictions for MIST-CF and SIRIUS
- Comparison table (results_table.tsv): joint formula+adduct accuracy (MIST-CF 0.862 vs SIRIUS Submission) and top-1 formula accuracy (0.842 vs 0.516 for CSI:FingerID)

## How to apply

Download the preprocessed CASMI 2022 data (casmi22.zip) from Zenodo record 8151490 and extract under ./data/casmi22, then extract the 304 positive-mode MS/MS spectra from the challenge mzML files using MZmine 3 feature detection/export. Run the pretrained MIST-CF model (src/mist_cf/mist_cf_score/predict_mgf.py), which ranks formula and adduct assignments generated from SIRIUS 'decomp' formula candidates, on the exported MGF spectra. Run SIRIUS/CSI:FingerID out-of-the-box on the same spectra via run_scripts/casmi22_eval/run_sirius.sh and normalize its output with wrangle_sirius.py. Score both systems' top-1 formula predictions (and joint formula+adduct where applicable) against the CASMI 2022 ground truth using analysis/evaluate_pred.py, writing per-spectrum predictions and the comparison table to results_table.tsv. Rationale: MIST-CF needs no MS1 isotopic information or manual refinement, yet was shown to tie the winning SIRIUS CASMI 2022 positive-mode entry while reaching 0.842 vs 0.516 top-1 formula accuracy over CSI:FingerID.

## Related tools

- **MIST-CF** (Pretrained energy-based transformer that ranks chemical formula and adduct assignments per positive-mode MS/MS spectrum without database lookup) — https://github.com/samgoldman97/mist-cf
- **SIRIUS** (Provides 'decomp' dynamic-programming formula candidates for MIST-CF scoring, and is run end-to-end (CSI:FingerID) as the out-of-the-box baseline) — https://github.com/boecker-lab/sirius/releases/download/
- **MZmine** (Feature detection and export of the 304 positive-mode CASMI 2022 MS/MS spectra from challenge mzML files)
- **CSI:FingerID** (SIRIUS's structure-to-fingerprint search used as the baseline whose top-1 formula accuracy (0.516) is compared against MIST-CF (0.842)) — https://github.com/sirius-ms/sirius
- **analysis/evaluate_pred.py** (Scoring script that computes joint formula+adduct and formula-only accuracy against CASMI 2022 ground truth and writes results_table.tsv) — https://github.com/samgoldman97/mist-cf

## Examples

```
. run_scripts/casmi22_eval/run_sirius.sh && python src/mist_cf/mist_cf_score/predict_mgf.py --output-dir results/casmi22 && python analysis/evaluate_pred.py --results-dir results/casmi22
```

## Evaluation signals

- MIST-CF joint formula+adduct accuracy on the 304 positive-mode spectra should land near 0.862, within noise of SIRIUS (Submission)
- MIST-CF top-1 formula accuracy should be near 0.842 versus roughly 0.516 for SIRIUS (CSI:FingerID), matching the Table 2 comparison (86.8% vs 67.8% joint figures)
- results_table.tsv must contain one row per spectrum with both systems' top-1 predictions and correct/incorrect flags against CASMI 2022 ground truth
- Every spectrum must have a non-empty formula candidate list from SIRIUS decomp before MIST-CF scoring; spectra with empty candidate sets indicate an SIRIUS_PATH or preprocessing failure
- No manual curation or postprocessing should be applied to either system's outputs — rankings must come directly from model/tool scores

## Limitations

- Models were trained only for positive-mode data; the benchmark cannot be extended to negative-mode CASMI spectra without retraining
- The quickstart public model may underperform on Orbitrap or higher-resolution data compared to the NIST20-trained models, which are available only upon reasonable request to licensed users
- MIST-CF ignores MS1 isotopic information and does not integrate its scores with MS1 isotopic scores, and does not consider adduct switching in MS2 subformula assignment

## Evidence
