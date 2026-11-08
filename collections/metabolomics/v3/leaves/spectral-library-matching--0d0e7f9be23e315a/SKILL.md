---
name: spectral-library-matching
description: Use when annotating unknown metabolite features from experimental tandem
  MS/MS peak lists and a set of candidate chemical structures is available for which
  predicted spectra can be generated.
when_to_use_negative:
- The metabolites of interest come from evolutionarily distant sources (e.g., plant
  or bacterial metabolism) with no trained language model, so candidate generation
  and confidence weighting do not apply
- The task requires de novo structure elucidation of metabolites from divergent, undiscovered
  biosynthetic routes that no candidate library contains
- Retention time or other analytical dimensions are required to distinguish candidates,
  since predicting retention times from chemical structures remains an open problem
  in this workflow
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0157
tools:
- name: CFM-ID
  role: Predicts ESI MS/MS spectra for each candidate structure in positive and negative
    ionization modes to construct the in silico spectral library
- name: DeepMet
  role: Supplies the 2.4 million metabolite candidate structures (sampled at least
    five times from 34.4 million generated structures) whose predicted spectra populate
    the library
  repo: https://github.com/skinniderlab/CLM
- name: RDKit
  role: Structure handling for candidates (e.g., canonicalization and Tanimoto similarity
    computation of top-ranked annotations)
- name: Python
  role: Implements m/z binning, spectral dot-product matching, the meta-learning combiner,
    and cross-validation metrics
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:e393f0921932f866ba1ff68ebd6c861b3fc9f9ff368478f2c040f8db7b283e50
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
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# spectral-library-matching

## Summary

Matches experimental ESI MS/MS spectra against an in silico spectral library built from candidate structures, scoring candidates by spectral dot-product and integrating the match with language-model sampling confidence via meta-learning to rank metabolite annotations.

## When to use

Use when annotating unknown metabolite features from experimental tandem MS/MS peak lists and a set of candidate chemical structures is available for which predicted spectra can be generated.

## When NOT to use

- The metabolites of interest come from evolutionarily distant sources (e.g., plant or bacterial metabolism) with no trained language model, so candidate generation and confidence weighting do not apply
- The task requires de novo structure elucidation of metabolites from divergent, undiscovered biosynthetic routes that no candidate library contains
- Retention time or other analytical dimensions are required to distinguish candidates, since predicting retention times from chemical structures remains an open problem in this workflow

## Inputs

- Candidate chemical structures with sampling-frequency confidence (e.g., 2.4 million DeepMet-generated metabolites filtered from 34.4 million generated structures)
- Experimental MS/MS spectra / peak lists from compiled human blood analyses (positive and negative ESI modes)
- Held-out metabolite labels for cross-validation

## Outputs

- In silico MS/MS spectral library (CFM-ID predicted spectra per candidate structure, per ionization mode)
- Ranked candidate annotations per spectrum with calibrated probabilities
- Evaluation table msms_annotation_evaluation.csv with exact-structure assignment rates, top-3/top-10 hit rates, and Tanimoto chemical-similarity metrics

## How to apply

Build the in silico library by predicting ESI spectra with CFM-ID for each candidate structure in both positive and negative ionization modes (here, the 2.4 million DeepMet-generated metabolites sampled at least five times out of 34.4 million generated structures). Preprocess the experimental MS/MS spectra (peak listing, m/z binning) from the compiled analyses into searchable peak lists. Score each experimental spectrum against library candidates using spectral dot-product matching, then combine the spectral score with the sampling-frequency confidence of the candidate via the meta-learning combiner to produce calibrated annotation probabilities. Evaluate on held-out metabolites by exact-structure assignment rate, top-3/top-10 hit rates, and chemical-similarity (e.g., Tanimoto) of the top-ranked annotations, and report results in msms_annotation_evaluation.csv. The rationale is that language-model-guided prioritization plus spectral matching outperforms accurate-mass prioritization alone.

## Related tools

- **CFM-ID** (Predicts ESI MS/MS spectra for each candidate structure in positive and negative ionization modes to construct the in silico spectral library)
- **DeepMet** (Supplies the 2.4 million metabolite candidate structures (sampled at least five times from 34.4 million generated structures) whose predicted spectra populate the library) — https://github.com/skinniderlab/CLM
- **RDKit** (Structure handling for candidates (e.g., canonicalization and Tanimoto similarity computation of top-ranked annotations))
- **Python** (Implements m/z binning, spectral dot-product matching, the meta-learning combiner, and cross-validation metrics)

## Examples

```
score = np.dot(exp_peaks, lib_peaks) / (np.linalg.norm(exp_peaks) * np.linalg.norm(lib_peaks))  # spectral dot-product match per candidate
```

## Evaluation signals

- Exact-structure assignment of held-out metabolites reaches approximately 52% in positive mode and 49% in negative mode
- msms_annotation_evaluation.csv is produced and contains top-3 and top-10 hit rates plus chemical-similarity (Tanimoto) metrics for top-ranked annotations
- The library covers exactly the filtered candidate set: 2.4 million structures sampled at least five times out of the 34.4 million generated structures
- Spectra are predicted and matched separately in both positive and negative ionization modes
- Combined (meta-learning) annotations show higher accuracy and better-calibrated probabilities than spectral matching alone

## Limitations

- The language model was trained and evaluated on human metabolites; evolutionarily distant applications (e.g., plant or bacterial metabolism) will likely require bespoke models
- The model has inherent limitations anticipating metabolites that originate from divergent, as-of-yet undiscovered biosynthetic routes
- Predicting MS/MS spectra or retention times from chemical structures remain open problems, constraining library fidelity
- Without MS/MS data, only mass-to-charge-based prioritization is available, which is well-calibrated but less informative than the integrated workflow

## Evidence

- [full_text] Workflow requires CFM-ID spectra in both ionization modes and dot-product scoring with meta-learning combiner: [evidence span withheld — anchor status: not_found]
