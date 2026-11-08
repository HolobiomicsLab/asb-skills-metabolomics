---
name: combined-method-score-integration
description: Use when benchmarking molecular formula identification and you need to
  merge FIDDLE predictions with external SIRIUS/BUDDY outputs to measure combined
  top-1/top-5 accuracy on the same spectra.
when_to_use_negative:
- The goal is standalone FIDDLE prediction on new samples rather than benchmark comparison
  — combined integration requires SIRIUS/BUDDY outputs on the same spectra and adds
  no benefit otherwise.
- Spectra contain compounds that overlap the training set by canonical SMILES — these
  must be filtered out before computing combined accuracies or results will be inflated.
- Input spectra have precursor m/z above 1500 or fall outside the supported precursor
  types/adducts covered by FIDDLE's training data.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: FIDDLE
  role: Primary deep learning formula predictor; its rescore checkpoint provides confidence
    scores that are combined with SIRIUS/BUDDY outputs via run_fiddle.py
  repo: https://github.com/JosieHong/FIDDLE
- name: SIRIUS
  role: Top-down baseline whose formula rankings (output_sirius.csv) are passed via
    --sirius_path for combined scoring
  repo: https://github.com/boecker-lab/sirius
- name: BUDDY
  role: Bottom-up baseline whose formula rankings (output_buddy.csv) are passed via
    --buddy_path for combined scoring
  repo: https://github.com/Philipbear/msbuddy
- name: MIST-CF
  role: Baseline whose scores are merged in the accuracy comparison step
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:c217cbebc8482e9db36a95a191d193f63c525eb638ebbabd05b4b8676327c104
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/JosieHong/FIDDLE
  repo_url: https://github.com/JosieHong/FIDDLE
  tools:
  - BUDDY
  - FIDDLE
  - MIST-CF
  - Q-TOF
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Combined-method score integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Integrates FIDDLE's deep learning confidence scores with SIRIUS and BUDDY candidate formula rankings on the same benchmark spectra to produce a combined prediction (output_fiddle_all.csv) that raises top-5 accuracy beyond any single method.

## When to use

Use when benchmarking molecular formula identification and you need to merge FIDDLE predictions with external SIRIUS/BUDDY outputs to measure combined top-1/top-5 accuracy on the same spectra.

## When NOT to use

- The goal is standalone FIDDLE prediction on new samples rather than benchmark comparison — combined integration requires SIRIUS/BUDDY outputs on the same spectra and adds no benefit otherwise.
- Spectra contain compounds that overlap the training set by canonical SMILES — these must be filtered out before computing combined accuracies or results will be inflated.
- Input spectra have precursor m/z above 1500 or fall outside the supported precursor types/adducts covered by FIDDLE's training data.

## Inputs

- Benchmark MS/MS spectra in mgf format with title, precursor_mz, precursor_type, and collision_energy fields (CASMI 2016, CASMI 2017, EMBL-MCF 2.0)
- output_sirius.csv (SIRIUS formula predictions on the same spectra)
- output_buddy.csv (BUDDY formula predictions on the same spectra)
- FIDDLE pre-trained checkpoints (fiddle_tcn_orbitrap.pt, fiddle_rescore_orbitrap.pt or Q-TOF equivalents) in ./check_point/

## Outputs

- output_fiddle_all.csv (combined FIDDLE + SIRIUS + BUDDY formula rankings)
- Per-tool, per-ion-mode top-1 and top-5 accuracy table after training-overlap filtering

## How to apply

First run FIDDLE standalone on the benchmark mgf spectra (with title, precursor_mz, precursor_type, collision_energy fields) using the Orbitrap TCN config and rescore checkpoint, applying a 5 ppm precursor mass tolerance for Orbitrap (10 ppm for Q-TOF) and confining precursor m/z to (0, 1500]. Then run SIRIUS and BUDDY on the identical spectra and format their results as output_sirius.csv and output_buddy.csv. Rerun run_fiddle.py with --buddy_path and --sirius_path so the combined-method results are written to output_fiddle_all.csv. Finally filter out benchmark compounds overlapping the training set by canonical SMILES and compute top-1/top-5 accuracy per tool and per ion mode, merging MIST-CF baseline scores.

## Related tools

- **FIDDLE** (Primary deep learning formula predictor; its rescore checkpoint provides confidence scores that are combined with SIRIUS/BUDDY outputs via run_fiddle.py) — https://github.com/JosieHong/FIDDLE
- **SIRIUS** (Top-down baseline whose formula rankings (output_sirius.csv) are passed via --sirius_path for combined scoring) — https://github.com/boecker-lab/sirius
- **BUDDY** (Bottom-up baseline whose formula rankings (output_buddy.csv) are passed via --buddy_path for combined scoring) — https://github.com/Philipbear/msbuddy
- **MIST-CF** (Baseline whose scores are merged in the accuracy comparison step)

## Examples

```
python run_fiddle.py --mgf casmi2016.mgf --buddy_path output_buddy.csv --sirius_path output_sirius.csv
```

## Evaluation signals

- Combined average top-5 accuracies reach ~80.0% (positive) and ~73.8% (negative) across the three external test sets, within a few percentage points of Fig. 5.
- FIDDLE-alone averages reach ~75.1% (positive) and ~66.2% (negative) top-5 accuracy before combination.
- Combined results consistently exceed SIRIUS (69.3%/67.6%), BUDDY (69.9%/61.4%), and MIST-CF (66.5% positive) top-5 accuracies.
- output_fiddle_all.csv is produced only when --buddy_path and --sirius_path are supplied; standalone runs do not generate it.
- All benchmark compounds overlapping the training set by canonical SMILES have been excluded before accuracy computation.

## Limitations

- FDR control via confidence score thresholds is proposed but not implemented in the current workflow.
- Combined performance was benchmarked only under Orbitrap (5 ppm) and Q-TOF (10 ppm) tolerances with precursor m/z confined to (0, 1500].
- FIDDLE's accuracy varies across experimental conditions, especially those with less training data, which limits combined-method gains there.

## Evidence

- [other] Run SIRIUS and BUDDY on the same benchmark spectra and format their outputs as output_sirius.csv and output_buddy.csv, then rerun run_fiddle.py with --buddy_path and --sirius_path to obtain the combined-method results written to output_fiddle_all.csv.: [evidence span withheld — anchor status: not_found]
- [readme] FIDDLE repos provide file-based workflows incorporating BUDDY and SIRIUS candidate outputs.: [evidence span withheld — anchor status: external_doc]
