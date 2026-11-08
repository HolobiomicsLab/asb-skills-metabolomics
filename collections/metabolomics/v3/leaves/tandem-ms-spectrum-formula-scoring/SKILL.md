---
name: tandem-ms-spectrum-formula-scoring
description: Use when you have an MS1 precursor mass and an MS/MS spectrum (MGF) and
  need a de novo chemical formula assignment without spectrum databases or fragmentation
  trees.
when_to_use_negative:
- Input spectra are negative-mode or you require negative-ion adduct handling — models
  were trained only on positive-mode data
- You need to integrate MIST-CF scores with MS1 isotopic pattern scores — this is
  not supported in the current implementation
- You must annotate adduct switching between MS2 subformula assignments — the subformula
  assignment protocol does not consider adduct switching
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MIST-CF
  role: Primary energy-based scoring model that assigns subformulae to MS2 peaks and
    ranks formula-adduct candidates
  repo: https://github.com/samgoldman97/mist-cf
- name: SIRIUS
  role: Provides SIRIUS decomp module to enumerate candidate chemical formulae from
    exact MS1 mass before scoring
  repo: https://github.com/boecker-lab/sirius/releases/download/
- name: FastFilter
  role: Lightweight neural network formula filtering model developed alongside MIST-CF
    for preliminary candidate screening
  repo: https://github.com/samgoldman97/mist-cf
- name: Chemical Formula Transformer
  role: Base transformer architecture that MIST-CF extends with subformula assignment
    and adduct diversity
  repo: https://github.com/samgoldman97/mist-cf
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:e79ff12018a67ab221a33d3d7c4357c1943bf2d8999cdfd21c0309f4ff4365e3
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/samgoldman97/mist-cf
  tools:
  - Chemical Formula Transformer
  - FastFilter
  - MIST-CF
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# tandem-ms-spectrum-formula-scoring

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Score chemical formula and adduct candidate assignments for a tandem mass spectrum using the MIST-CF energy-based model, then re-rank formula candidates enumerated by SIRIUS decomp to rank database-independent formula predictions.

## When to use

Use when you have an MS1 precursor mass and an MS/MS spectrum (MGF) and need a de novo chemical formula assignment without spectrum databases or fragmentation trees.

## When NOT to use

- Input spectra are negative-mode or you require negative-ion adduct handling — models were trained only on positive-mode data
- You need to integrate MIST-CF scores with MS1 isotopic pattern scores — this is not supported in the current implementation
- You must annotate adduct switching between MS2 subformula assignments — the subformula assignment protocol does not consider adduct switching

## Inputs

- MGF-format tandem mass spectra with MS1 precursor masses (e.g. data/demo_specs.mgf from CASMI22)
- Enumerated formula candidates per precursor mass (from SIRIUS decomp)
- Pretrained MIST-CF model checkpoint (e.g. quickstart model trained on public NPLIB1/CANOPUS data)
- SIRIUS executable v5.5.7 headless for linux64 with SIRIUS_PATH environment variable set

## Outputs

- Ranked list of formula-adduct assignments per input spectrum, scored by MIST-CF energy
- Top-1 and top-k formula predictions, optionally written to quickstart/mist_cf_out/
- Benchmark top-1 accuracy table (benchmarking_results.csv) when evaluated across architectures

## How to apply

First enumerate candidate formulae for the observed MS1 mass using the SIRIUS decomp dynamic-programming module (SIRIUS v5.5.7 headless, SIRIUS_PATH set). For each candidate, run the MIST-CF scorer: the model internally assigns subformulae to MS2 peaks and neutral losses (circumventing fragmentation tree construction), embeds the fragment and neutral-loss formulae with sinusoidal formula embeddings, considers multiple positive-mode adduct types beyond [M+H]+, and conditions on the instrument type as a model covariate. Rank candidates by the model's energy score of spectrum-candidate agreement rather than by any isotope tree score. Output top-1 (and optionally top-k) formula assignments per spectrum; validate on held-out NPLIB1 spectra where MIST-CF yields ~0.741 top-1 accuracy training on NPLIB1 alone, and ~0.769 when NIST20 Orbitrap data are added.

## Related tools

- **MIST-CF** (Primary energy-based scoring model that assigns subformulae to MS2 peaks and ranks formula-adduct candidates) — https://github.com/samgoldman97/mist-cf
- **SIRIUS** (Provides SIRIUS decomp module to enumerate candidate chemical formulae from exact MS1 mass before scoring) — https://github.com/boecker-lab/sirius/releases/download/
- **FastFilter** (Lightweight neural network formula filtering model developed alongside MIST-CF for preliminary candidate screening) — https://github.com/samgoldman97/mist-cf
- **Chemical Formula Transformer** (Base transformer architecture that MIST-CF extends with subformula assignment and adduct diversity) — https://github.com/samgoldman97/mist-cf

## Examples

```
. quickstart/download_model.sh && . quickstart/run_model.sh   # scores formula candidates for spectra in data/demo_specs.mgf, writes to quickstart/mist_cf_out/
```

## Evaluation signals

- Held-out NPLIB1 test-set top-1 accuracy should be ~0.741 when training on NPLIB1 only, and ~0.769 when NIST20 Orbitrap spectra are added to training
- MIST-CF should beat the next-best baseline architecture (FFN, MS1-Only, plain Transformer) by >10% absolute top-1 accuracy margin on re-ranking up to 256 candidates per spectrum
- On positive-mode CASMI2022 challenge spectra, MIST-CF should match the winning SIRIUS solution without MS1 isotopic information or manual refinement
- Predictions should be written to the specified output directory (e.g. quickstart/mist_cf_out/) with a score per enumerated formula-adduct candidate

## Limitations

- Models trained only for positive-mode data; negative-mode support is absent
- No integration of MIST-CF scores with MS1 isotopic scores
- Adduct switching in MS2 subformula assignment is not considered
- Still relies on SIRIUS's algorithmic decomposition of exact masses into formula candidates
- Quickstart model trained on public NPLIB1/CANOPUS data may be less performant than the NIST20-trained model, particularly for Orbitrap or higher-resolution data

## Evidence

- [methods] Re-ranking up to 256 candidates per held-out test spectrum; NPLIB1-only top-1 accuracy of 0.741 versus 0.769 when NIST20 is added.: [evidence span withheld — anchor status: not_found]
- [discussion] SIRIUS decomp is used to enumerate candidate formulae from the observed MS1 mass.: [evidence span withheld — anchor status: not_found]
