---
name: ms2-spectrum-subformula-assignment
description: Use when performing database-independent, de novo chemical formula assignment
  from MS/MS spectra where fragmentation trees (SIRIUS) are too slow or unavailable,
  and you need peak-to-subformula mappings to feed an energy-based formula scorer.
when_to_use_negative:
- Negative-ion-mode spectra — models were trained only for positive-mode data and
  negative-mode support is absent
- When MS1 isotopic information must be integrated into the score — MIST-CF does not
  directly address integration of MIST-CF scores with MS1 isotopic scores
- When adduct switching within the MS2 subformula assignment itself must be modeled
  — the current routine does not consider adduct switching in MS2 subformula assignment
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: SIRIUS
  role: Dynamic programming formula candidate enumeration (SIRIUS decomp) providing
    the precursor formula pool that subformula assignment is performed against
  repo: https://github.com/boecker-lab/sirius
- name: MIST-CF
  role: Energy-based neural network (Chemical Formula Transformer) that consumes the
    subformula assignments and ranks candidate formulae/adducts per spectrum
  repo: https://github.com/samgoldman97/mist-cf
- name: FastFilter
  role: Light-weight neural network that pre-ranks formula candidates down to the
    top-256 before full MIST-CF scoring
  repo: https://github.com/samgoldman97/mist-cf
- name: Chemical Formula Transformer
  role: Architecture underlying MIST-CF that embeds fragment and neutral-loss formulae
    sinusoidally for scoring
  repo: https://github.com/samgoldman97/mist-cf
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:029d14b4688516e1fc1055ca50e6cf49e12200fcbcd4bd951debba3d66a27f87
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/boecker-lab/sirius
  tools:
  - Chemical Formula Transformer
  - FastFilter
  - MIST-CF
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# MS2-spectrum-subformula-assignment

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

A fast peak subformula assignment routine that replaces SIRIUS fragmentation tree construction by assigning plausible molecular subformulas to each MS2 peak and neutral loss, conditioned on the precursor formula candidate, to score de novo chemical formulae with MIST-CF. It powers the MIST-CF energy-based scoring pipeline on NPLIB1/NIST20 and CASMI2022 positive-mode tandem mass spectra.

## When to use

Use when performing database-independent, de novo chemical formula assignment from MS/MS spectra where fragmentation trees (SIRIUS) are too slow or unavailable, and you need peak-to-subformula mappings to feed an energy-based formula scorer.

## When NOT to use

- Negative-ion-mode spectra — models were trained only for positive-mode data and negative-mode support is absent
- When MS1 isotopic information must be integrated into the score — MIST-CF does not directly address integration of MIST-CF scores with MS1 isotopic scores
- When adduct switching within the MS2 subformula assignment itself must be modeled — the current routine does not consider adduct switching in MS2 subformula assignment

## Inputs

- Preprocessed MS/MS spectra (MGF format) with precursor mass, instrument type, and peak m/z / intensity lists (e.g. from GNPS/GNPS-Library NPLIB1, NIST20, or CASMI2022 challenge sets)
- Candidate molecular formulae enumerated by SIRIUS decomp for the precursor mass
- Element constraint table ( allowable elements and count ranges) as used by SIRIUS decomp
- Trained MIST-CF model checkpoint (public NPLIB1/CANOPUS-trained model via quickstart/download_model.sh, or NIST20-trained model on request)

## Outputs

- Per-spectrum ranked list of candidate chemical formulae with adduct assignments (JSON/CSV-style output as produced by MIST-CF)
- Peak-to-subformula assignment map for each formula candidate (fragment formula + neutral loss formula per MS2 peak)
- Evaluation metrics: top-1 formula accuracy, joint formula+adduct accuracy, SIRIUS failure rate, and per-spectrum concordance (e.g. results_table.csv)

## How to apply

Enumerate candidate molecular formulae for the observed MS1 precursor mass using SIRIUS's dynamic programming decomposition module (SIRIUS decomp), then for each formula candidate assign every MS2 peak (and its neutral loss) a plausible subformula derived from that candidate, using a chemical subformula assignment protocol that enforces element-count constraints. MIST-CF scores the agreement between each candidate and the spectrum via a Chemical Formula Transformer with sinusoidal formula embeddings, treating instrument type and neutral loss formula as additional model covariates, and considers multiple positive-mode adduct types beyond [M+H]+. Run MIST-CF end-to-end via the provided scripts (e.g. `run_scripts/public_data_train/train_mist_cf.sh` for training; `quickstart/run_model.sh` for inference on preprocessed spectra in MGF format), using FastFilter to pre-rank formula candidates down to the top-256 before full scoring. Evaluate top-1 formula accuracy against ground-truth labels (e.g. on NPLIB1 or CASMI2022) where MIST-CF achieves ~71% top-1 accuracy versus SIRIUS 5.6.3's 48% on NPLIB1 with full RDBE decoys.

## Related tools

- **SIRIUS** (Dynamic programming formula candidate enumeration (SIRIUS decomp) providing the precursor formula pool that subformula assignment is performed against) — https://github.com/boecker-lab/sirius
- **MIST-CF** (Energy-based neural network (Chemical Formula Transformer) that consumes the subformula assignments and ranks candidate formulae/adducts per spectrum) — https://github.com/samgoldman97/mist-cf
- **FastFilter** (Light-weight neural network that pre-ranks formula candidates down to the top-256 before full MIST-CF scoring) — https://github.com/samgoldman97/mist-cf
- **Chemical Formula Transformer** (Architecture underlying MIST-CF that embeds fragment and neutral-loss formulae sinusoidally for scoring) — https://github.com/samgoldman97/mist-cf

## Examples

```
. quickstart/run_model.sh  # predicts formulae for data/demo_specs.mgf using the public NPLIB1/CANOPUS-trained model
```

## Evaluation signals

- Top-1 formula accuracy should reach ~71% on NPLIB1 with full RDBE decoys (vs 48% for SIRIUS 5.6.3)
- Joint formula+adduct accuracy should be ~80% on NPLIB1, confirming the multi-adduct scoring path is working
- MIST-CF should predict every spectrum in the test set (no formula failures), contrasting with SIRIUS's ~10.77% failure rate
- Concordance analysis: MIST-CF should correctly predict ~36% of spectra where SIRIUS cannot, indicating the subformula-assignment route captures spectra fragmentation trees miss
- Output JSON/CSV must contain a ranked formula list per spectrum with adduct labels and confidence scores matching the expected schema

## Limitations

- Models were trained only on positive-mode data; negative-mode spectra are unsupported
- MIST-CF does not integrate MS1 isotopic scores, which can limit performance on Orbitrap or other high-resolution instruments where isotope patterns are informative
- Adduct switching within the MS2 subformula assignment is not considered
- The public NPLIB1/CANOPUS-trained model may be less performant than NIST20-trained models, particularly on Orbitrap/high-resolution data; NIST20 models require a NIST license and are available upon reasonable request
- The routine still relies on SIRIUS's fast algorithmic decomposition of exact masses into formula candidates as the candidate-generation step

## Evidence

- [results] MIST-CF 71% vs SIRIUS 48% top-1 accuracy on NPLIB1: [evidence span withheld — anchor status: not_found]
