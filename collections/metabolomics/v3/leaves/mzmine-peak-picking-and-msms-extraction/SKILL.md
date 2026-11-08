---
name: mzmine-peak-picking-and-msms-extraction
description: Use when raw LC-MS/MS challenge or study data arrives as mzML files and
  you need discrete, exported MS/MS spectra (e.g., an MGF-style set) as input for
  formula or structure annotation tools like MIST-CF or SIRIUS.
when_to_use_negative:
- Input is already exported, preprocessed spectra (e.g., the demo_specs.mgf MGF shipped
  in the mist-cf repo or a feature table) — re-running MZmine would be redundant
- The downstream task is negative-mode formula annotation — the released MIST-CF pipeline
  targets positive mode only, so positive-mode extraction assumptions apply
- You need MS1 isotopic-pattern scoring — MIST-CF uses zero MS1 isotopic information,
  so isotope-focused peak picking is out of scope for this pipeline
tools:
- name: MZmine
  role: Feature detection and MS/MS export tool that converts raw challenge mzML files
    into the 304 positive-mode spectra used for benchmarking
- name: MIST-CF
  role: Downstream consumer of the exported spectra; scores formula/adduct candidates
    via src/mist_cf/mist_cf_score/predict_mgf.py using SIRIUS-decomp formula candidates
  repo: https://github.com/samgoldman97/mist-cf
- name: SIRIUS
  role: Comparison tool run on the identical exported spectra (run_scripts/casmi22_eval/run_sirius.sh)
    and provider of the SIRIUS decomp module used to enumerate formula candidates
  repo: https://github.com/boecker-lab/sirius
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:9eb6940df923f119e4823fa0eed8430c9fabcc192f3d7b937c941f8a118d25b7
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# mzmine-peak-picking-and-msms-extraction

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Extract per-feature MS/MS spectra from raw challenge mzML files using MZmine 3 feature detection/export, as done to prepare the 304 positive-mode CASMI 2022 challenge spectra for downstream chemical formula benchmarking with MIST-CF and SIRIUS.

## When to use

Use when raw LC-MS/MS challenge or study data arrives as mzML files and you need discrete, exported MS/MS spectra (e.g., an MGF-style set) as input for formula or structure annotation tools like MIST-CF or SIRIUS.

## When NOT to use

- Input is already exported, preprocessed spectra (e.g., the demo_specs.mgf MGF shipped in the mist-cf repo or a feature table) — re-running MZmine would be redundant
- The downstream task is negative-mode formula annotation — the released MIST-CF pipeline targets positive mode only, so positive-mode extraction assumptions apply
- You need MS1 isotopic-pattern scoring — MIST-CF uses zero MS1 isotopic information, so isotope-focused peak picking is out of scope for this pipeline

## Inputs

- Raw challenge LC-MS/MS mzML files (e.g., CASMI 2022 mzML archives from Zenodo record 8151490, extracted under ./data/casmi22)
- MZmine 3 feature detection/export batch configuration

## Outputs

- Exported per-feature MS/MS spectra for the 304 positive-mode challenge spectra, consumable by MIST-CF (MGF) and SIRIUS
- Feature tables with precursor m/z / retention-time metadata for downstream wrangling (wrangle_sirius.py, evaluate_pred.py)

## How to apply

Run MZmine 3 feature detection over the challenge mzML files (here the CASMI 2022 mzML archives from Zenodo record 8151490), then export the detected features' associated MS/MS spectra so that each of the 304 positive-mode challenge spectra becomes a single annotated query. Export should preserve precursor m/z, retention time, and the peak list needed by downstream tools: MIST-CF's predict_mgf.py consumes exported spectra as MGF with SIRIUS-decomp formula candidates, while SIRIUS/CSI:FingerID is run separately on the same spectra per run_scripts/casmi22_eval/run_sirius.sh. The rationale is that MZmine is the widely used front end for MS/MS peak picking and feature export, and consistent preprocessing guarantees that MIST-CF and SIRIUS are scored on identical spectra, so top-1 formula and joint formula+adduct accuracies (0.842 vs 0.516; 0.862 vs SIRIUS submission) are directly comparable via analysis/evaluate_pred.py.

## Related tools

- **MZmine** (Feature detection and MS/MS export tool that converts raw challenge mzML files into the 304 positive-mode spectra used for benchmarking)
- **MIST-CF** (Downstream consumer of the exported spectra; scores formula/adduct candidates via src/mist_cf/mist_cf_score/predict_mgf.py using SIRIUS-decomp formula candidates) — https://github.com/samgoldman97/mist-cf
- **SIRIUS** (Comparison tool run on the identical exported spectra (run_scripts/casmi22_eval/run_sirius.sh) and provider of the SIRIUS decomp module used to enumerate formula candidates) — https://github.com/boecker-lab/sirius

## Evaluation signals

- The exported set contains exactly the 304 positive-mode MS/MS challenge spectra referenced in the CASMI 2022 evaluation
- Each exported spectrum carries precursor m/z and peak-list metadata sufficient for SIRIUS-decomp candidate generation and MIST-CF scoring
- Running analysis/evaluate_pred.py on both tools' top-1 predictions reproduces the Table 2 numbers (MIST-CF joint formula+adduct ≈ 0.862; formula-only 0.842 vs SIRIUS CSI:FingerID 0.516)
- Per-spectrum predictions for both MIST-CF and SIRIUS are written to results_table.tsv and keyed to the same spectrum identifiers

## Limitations

- MIST-CF models were trained only on positive-mode data, so this extraction pipeline supports positive-mode benchmarking only
- The quickstart MIST-CF model is trained on the public NPLIB1/CANOPUS dataset and may be less performant on Orbitrap or higher-resolution data than the NIST20-trained model (available only upon reasonable request with a NIST license)
- MIST-CF still relies on SIRIUS's algorithmic decomposition of exact masses into formula candidates, so the export must be compatible with SIRIUS decomp input

## Evidence

- [methods] Extract the 304 positive-mode MS/MS spectra from the challenge mzML files using MZmine 3 feature detection/export.: [evidence span withheld — anchor status: not_found]
