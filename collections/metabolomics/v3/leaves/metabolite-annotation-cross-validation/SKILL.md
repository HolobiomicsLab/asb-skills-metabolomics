---
name: metabolite-annotation-cross-validation
description: Use when evaluating whether generated candidate structures plus predicted
  MS/MS spectra improve exact-structure assignment of held-out metabolites over accurate-mass
  prioritization alone.
when_to_use_negative:
- Applying to plant, bacterial, or other evolutionarily distant metabolism — the language
  model was trained and evaluated only on human metabolite structures and would require
  a bespoke model
- When the goal is annotating metabolites from divergent, undiscovered biosynthetic
  routes — the model has inherent limitations anticipating these
- When no candidate structure set or predicted/in silico MS/MS library exists — the
  evaluation requires spectral matching against CFM-ID-predicted candidates
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CFM-ID
  role: Predicts ESI MS/MS spectra in positive and negative modes for each retained
    candidate structure to build the in silico spectral library used for dot-product
    matching
- name: DeepMet
  role: Chemical language model whose 34.4 million generated structures and sampling
    frequencies supply candidates and the confidence signal integrated by the meta-learning
    combiner
  repo: https://github.com/skinniderlab/CLM
- name: RDKit
  role: Computes chemical-similarity (Tanimoto) metrics and handles SMILES-to-molecule
    conversion for candidates and annotations
- name: Python
  role: Runs the cross-validation, dot-product scoring, and meta-learning evaluation
    scripts
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:c6c0838bf5ceff708aedf625cbc56427719537b347a8eb79a19ac8360a6c15fe
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# metabolite-annotation-cross-validation

## Summary

Benchmark language-model-guided metabolite annotation by cross-validating held-out metabolites against an in silico MS/MS library built from generated candidate structures, reporting exact-structure assignment, top-k hit rates, and chemical similarity.

## When to use

Use when evaluating whether generated candidate structures plus predicted MS/MS spectra improve exact-structure assignment of held-out metabolites over accurate-mass prioritization alone.

## When NOT to use

- Applying to plant, bacterial, or other evolutionarily distant metabolism — the language model was trained and evaluated only on human metabolite structures and would require a bespoke model
- When the goal is annotating metabolites from divergent, undiscovered biosynthetic routes — the model has inherent limitations anticipating these
- When no candidate structure set or predicted/in silico MS/MS library exists — the evaluation requires spectral matching against CFM-ID-predicted candidates

## Inputs

- Deposited DeepMet resource of 34.4 million generated structures with sampling frequencies (filtered to 2.4 million candidates sampled >=5 times)
- Experimental MS/MS peak lists from human blood analyses (29.1 million spectra, 4,510 analyses), preprocessed by peak listing and m/z binning
- Held-out metabolite set for cross-validation
- CFM-ID-predicted ESI spectra per candidate in positive and negative ionization modes

## Outputs

- Exact-structure assignment rates per ionization mode (target ~52% positive, ~49% negative)
- Top-3 and top-10 hit rates for held-out metabolites
- Chemical-similarity metrics (e.g., Tanimoto) for top-ranked annotations
- Calibrated annotation probabilities from the meta-learning combiner
- msms_annotation_evaluation.csv summary table

## How to apply

Filter the deposited DeepMet resource of 34.4 million generated structures down to the 2.4 million candidates sampled at least five times, treating the recorded sampling frequency as the language-model confidence. Predict ESI MS/MS spectra for every retained structure with CFM-ID in both positive and negative ionization modes to build the in silico spectral library, then preprocess experimental peak lists from the 29.1 million spectra compiled from 4,510 human blood analyses (peak listing, m/z binning). Score each experimental spectrum against library candidates using spectral dot-product matching, and integrate the spectral score with the sampling-frequency confidence via a meta-learning combiner that produces calibrated annotation probabilities. Cross-validate on held-out metabolites, computing exact-structure assignment rates in each ion mode, top-3 and top-10 hit rates, and Tanimoto chemical similarity of top-ranked annotations. The reproduced evaluation is considered correct when exact-structure assignment reaches approximately 52% in positive mode and 49% in negative mode, with top-k and similarity metrics reported in msms_annotation_evaluation.csv.

## Related tools

- **CFM-ID** (Predicts ESI MS/MS spectra in positive and negative modes for each retained candidate structure to build the in silico spectral library used for dot-product matching)
- **DeepMet** (Chemical language model whose 34.4 million generated structures and sampling frequencies supply candidates and the confidence signal integrated by the meta-learning combiner) — https://github.com/skinniderlab/CLM
- **RDKit** (Computes chemical-similarity (Tanimoto) metrics and handles SMILES-to-molecule conversion for candidates and annotations)
- **Python** (Runs the cross-validation, dot-product scoring, and meta-learning evaluation scripts)

## Examples

```
git clone https://github.com/skinniderlab/CLM.git; pip install -r requirements.txt; snakemake -s Snakefile
```

## Evaluation signals

- Exact-structure assignment of held-out metabolites reaches approximately 52% in positive mode and 49% in negative mode
- Top-3 and top-10 hit rates are computed and exceed accurate-mass-prioritization-only baselines
- Meta-learning combiner outputs are well-calibrated probabilities, not just rankings
- All metrics (exact-structure, top-3/top-10, Tanimoto similarity) are present in msms_annotation_evaluation.csv
- The filtered candidate set contains the 2.4 million structures sampled at least five times (not the full 34.4 million)

## Limitations

- Predicting MS/MS spectra or retention times from chemical structures remains an open problem, capping annotation accuracy
- Only a small fraction of peaks detected in metabolomics can typically be identified, and structure elucidation of novel metabolites is low-throughput
- MS/MS integration improves but does not guarantee annotation; there remain opportunities to more tightly integrate DeepMet with analytical data

## Evidence

- [readme] software availability for training and evaluation: [evidence span withheld — anchor status: external_doc]
