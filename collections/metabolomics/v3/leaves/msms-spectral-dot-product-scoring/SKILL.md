---
name: msms-spectral-dot-product-scoring
description: Use when experimental MS/MS peak lists from small-molecule metabolomics
  must be matched against candidate chemical structures to produce ranked, calibrated
  exact-structure assignments.
when_to_use_negative:
- When only accurate mass is available and no tandem-MS spectra were acquired — mass-to-charge-based
  prioritization alone is the applicable route
- When the true metabolite originates from divergent, undiscovered biosynthetic routes
  outside the human metabolome the DeepMet model was trained on — candidates will
  be absent from the library
- When candidates are drawn from arbitrary chemical space rather than the DeepMet-generated
  metabolite candidate resource, since the meta-learning combiner is calibrated to
  sampling-frequency confidence from that resource
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_3173
tools:
- name: CFM-ID
  role: Predicts in silico ESI MS/MS spectra for each retained candidate structure
    in positive and negative modes to build the reference library being dot-product
    matched
- name: DeepMet
  role: Supplies the 2.4 million metabolite candidate structures and their sampling
    frequencies, which serve as the confidence prior integrated with the spectral
    score
  repo: https://github.com/skinniderlab/CLM
- name: RDKit
  role: Computes chemical-similarity (e.g., Tanimoto) metrics for top-ranked annotations
    during evaluation
- name: Python
  role: Implements peak preprocessing, m/z binning, dot-product scoring, and meta-learning
    combination
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:19664a2e90023830f00322bfdf6b4f1baa54ea80af63e75ac43c831d8cb68f11
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/skinniderlab/CLM
  tools:
  - CFM-ID
  - DeepMet
  - Python
  - RDKit
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# MS/MS spectral dot-product scoring

## Summary

Score experimental tandem-MS spectra against a library of CFM-ID-predicted in silico spectra using spectral dot-product matching, then integrate the match score with DeepMet sampling-frequency confidence via a meta-learning combiner to rank exact-structure annotations of unknown metabolites.

## When to use

Use when experimental MS/MS peak lists from small-molecule metabolomics must be matched against candidate chemical structures to produce ranked, calibrated exact-structure assignments.

## When NOT to use

- When only accurate mass is available and no tandem-MS spectra were acquired — mass-to-charge-based prioritization alone is the applicable route
- When the true metabolite originates from divergent, undiscovered biosynthetic routes outside the human metabolome the DeepMet model was trained on — candidates will be absent from the library
- When candidates are drawn from arbitrary chemical space rather than the DeepMet-generated metabolite candidate resource, since the meta-learning combiner is calibrated to sampling-frequency confidence from that resource

## Inputs

- Preprocessed experimental MS/MS spectra (peak lists with binned m/z / intensity vectors) from metabolomics analyses
- In silico MS/MS spectra predicted by CFM-ID for each candidate structure, in positive and negative ionization modes
- DeepMet sampling frequencies for each of the 2.4 million retained metabolite candidates (used as model confidence)

## Outputs

- Dot-product match scores of each experimental spectrum against every library candidate
- Meta-learning-integrated calibrated annotation probabilities with ranked candidate structures
- Evaluation table (msms_annotation_evaluation.csv) with exact-structure assignment rates, top-3/top-10 hit rates, and chemical-similarity metrics

## How to apply

Preprocess experimental MS/MS spectra into peak lists and bin m/z values from each acquired analysis. Build the in silico spectral library by predicting ESI spectra for every retained candidate structure with CFM-ID in both positive and negative ionization modes. Score each experimental spectrum against library candidates by spectral dot-product matching of the binned intensity vectors. Combine the spectral dot-product score with the language-model sampling-frequency confidence (from the DeepMet-generated candidate resource) using the meta-learning combiner, which integrates outputs of multiple independent machine-learning models to distinguish correct from incorrect annotations and yields calibrated probabilities. Evaluate on held-out metabolites by exact-structure assignment rate, top-3/top-10 hit rates, and Tanimoto chemical similarity of top-ranked annotations; the reference reproduction reached ~52% exact-structure assignment in positive mode and ~49% in negative mode.

## Related tools

- **CFM-ID** (Predicts in silico ESI MS/MS spectra for each retained candidate structure in positive and negative modes to build the reference library being dot-product matched)
- **DeepMet** (Supplies the 2.4 million metabolite candidate structures and their sampling frequencies, which serve as the confidence prior integrated with the spectral score) — https://github.com/skinniderlab/CLM
- **RDKit** (Computes chemical-similarity (e.g., Tanimoto) metrics for top-ranked annotations during evaluation)
- **Python** (Implements peak preprocessing, m/z binning, dot-product scoring, and meta-learning combination)

## Examples

```
score = np.dot(exp_spec, lib_spec) / (np.linalg.norm(exp_spec) * np.linalg.norm(lib_spec))
```

## Evaluation signals

- Exact-structure assignment on held-out metabolites reaches ~52% in positive ion mode and ~49% in negative ion mode
- Top-3 and top-10 hit rates improve over accurate-mass prioritization alone
- Tanimoto chemical-similarity metrics for top-ranked annotations are reported and exceed the mass-only baseline
- Annotation probabilities produced by the meta-learning combiner are well-calibrated (predicted confidence matches observed correctness)
- Every experimental spectrum is scored against the same binned m/z representation used to construct the CFM-ID in silico library

## Limitations

- Predicting MS/MS spectra (and retention times) from chemical structures remains an open problem, so library quality is bounded by CFM-ID prediction accuracy
- The language model was trained and evaluated on human metabolites; applications to evolutionarily distant organisms (e.g., plant or bacterial metabolism) likely require bespoke models
- The approach has inherent limitations anticipating metabolites from divergent, as-of-yet undiscovered biosynthetic routes
- MS/MS matching is layered on a language-model prior; tight integration of DeepMet with analytical data was noted as an unrealized opportunity

## Evidence
