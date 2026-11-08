---
name: in-silico-msms-spectrum-prediction
description: Use when a large set of candidate metabolite structures needs predicted
  ESI tandem mass spectra in both ionization modes so experimental MS/MS spectra can
  be matched to structures without experimental reference libraries.
when_to_use_negative:
- Experimental reference spectra already exist for the metabolites — an experimental
  spectral library will outperform predicted spectra, which remain an open problem
  per the article
- The candidate structures come from evolutionarily distant organisms (plant or bacterial
  metabolism) — the language model behind the candidates was trained and evaluated
  only on human metabolite structures
- No tandem MS data are available and only accurate-mass prioritization is possible
  — use mass-to-charge filtering of the DeepMet resource instead
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: CFM-ID
  role: Predicts ESI MS/MS spectra for each retained candidate structure in both positive
    and negative ionization modes to construct the in silico spectral library
- name: DeepMet
  role: Generates the candidate metabolite structures whose spectra are predicted;
    its sampling frequencies serve as the confidence signal integrated with spectral
    scores
  repo: https://github.com/skinniderlab/CLM
- name: RDKit
  role: Converts input SMILES to molecules and standardizes structures (removing light
    fragments, neutralizing charges, filtering valid elements) before spectrum prediction
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:dc2c2057f756393096fbd0f3ce0dd219cd7ad6c5e131a246bb80edf7b16cb6cc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/skinniderlab/CLM
  tools:
  - CFM-ID
  - DeepMet
  - Python
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# in-silico-msms-spectrum-prediction

## Summary

Predicts ESI MS/MS spectra for candidate metabolite structures (e.g., DeepMet-generated SMILES) in positive and negative ionization modes using CFM-ID, producing an in silico spectral library that enables exact-structure assignment of held-out metabolites beyond accurate-mass prioritization alone.

## When to use

Use when a large set of candidate metabolite structures needs predicted ESI tandem mass spectra in both ionization modes so experimental MS/MS spectra can be matched to structures without experimental reference libraries.

## When NOT to use

- Experimental reference spectra already exist for the metabolites — an experimental spectral library will outperform predicted spectra, which remain an open problem per the article
- The candidate structures come from evolutionarily distant organisms (plant or bacterial metabolism) — the language model behind the candidates was trained and evaluated only on human metabolite structures
- No tandem MS data are available and only accurate-mass prioritization is possible — use mass-to-charge filtering of the DeepMet resource instead

## Inputs

- SMILES strings of candidate metabolite structures (e.g., 2.4 million DeepMet-generated candidates filtered from 34.4 million by sampling frequency >= 5)
- Ionization mode specification (positive and negative ESI)
- Preprocessed experimental MS/MS spectra (peak lists, binned m/z) from metabolomics analyses

## Outputs

- In silico MS/MS spectral library covering both ionization modes
- Spectral dot-product match scores between experimental and predicted spectra
- Calibrated per-annotation probabilities from the meta-learning combiner
- Exact-structure assignment rates and top-3/top-10 hit rates (expected ~52% positive / ~49% negative mode)

## How to apply

Start from the filtered set of candidate structures — e.g., the 2.4 million DeepMet-generated metabolites sampled at least five times, using sampling frequency as language-model confidence. Predict ESI MS/MS spectra for every retained candidate with CFM-ID in both positive and negative ionization modes to construct the in silico spectral library. Preprocess experimental MS/MS spectra from compiled human blood analyses (peak listing, m/z binning) before library searching. Score each experimental spectrum against library candidates via spectral dot-product matching, then integrate the spectral score with the DeepMet sampling-frequency confidence through the meta-learning combiner to yield calibrated annotation probabilities. Because spectrum prediction from structure remains an open problem, treat predicted spectra as approximate and always report top-3/top-10 hit rates and Tanimoto chemical-similarity of top-ranked annotations alongside exact-match rates.

## Related tools

- **CFM-ID** (Predicts ESI MS/MS spectra for each retained candidate structure in both positive and negative ionization modes to construct the in silico spectral library)
- **DeepMet** (Generates the candidate metabolite structures whose spectra are predicted; its sampling frequencies serve as the confidence signal integrated with spectral scores) — https://github.com/skinniderlab/CLM
- **RDKit** (Converts input SMILES to molecules and standardizes structures (removing light fragments, neutralizing charges, filtering valid elements) before spectrum prediction)

## Evaluation signals

- Exact-structure assignment of held-out metabolites reaches approximately 52% in positive mode and 49% in negative mode
- Top-3 and top-10 hit rates for held-out metabolites exceed those from accurate-mass prioritization alone and are reported in msms_annotation_evaluation.csv
- Chemical similarity (e.g., Tanimoto) of top-ranked annotations to true structures is computed and reported
- Meta-learning combined probabilities are well-calibrated, not merely ranked
- Predicted spectra are generated for all retained candidates in both ionization modes before any library searching

## Limitations

- Predicting MS/MS spectra (or retention times) from chemical structures remains an open problem despite extensive effort, so predicted spectra are approximations
- The language model generating the candidate structures was trained and evaluated only on human metabolites; plant or bacterial applications likely require bespoke models
- The approach has inherent limitations anticipating metabolites from divergent, as-of-yet undiscovered biosynthetic routes
- Tighter integration of DeepMet with analytical data is an unrealized opportunity, limiting how far the confidence signal can compensate for weak spectral matches

## Evidence
