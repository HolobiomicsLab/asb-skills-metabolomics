---
name: cosine-similarity-spectrum-matching
description: Use when experimental MS/MS spectra (e.g. 29.1M spectra from human blood
  analyses on MetaboLights) must be annotated by spectral matching against a predicted
  in silico library, and you need to quantify how much library expansion (HMDB vs
  HMDB + DeepMet structures) increases matched spectra at a.
when_to_use_negative:
- When the goal is de novo structure elucidation or targeted discovery of predicted
  metabolites rather than spectral-library annotation of experimental spectra
- When candidate structures must be prioritized using mass-to-charge ratio alone without
  a spectral library, since the paper treats m/z-based prioritization and library
  matching as distinct integration steps
- When a genuine experimental reference spectral library (e.g. experimental standards)
  is available and in silico prediction is unnecessary
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: DeepMet
  role: Chemical language model used to generate the 2.4 million predicted metabolite
    structures that expand the in silico MS/MS spectral library
  repo: https://github.com/skinniderlab/CLM
- name: Python 3.10
  role: Runtime for implementing the cosine similarity spectral matching searches
    and writing the match-count comparison output
  repo: https://github.com/skinniderlab/CLM
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:183fb8b118bfcc173bcc997c3dea38b354ef814c4baa2530c6fe140f8b4a6d4d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/skinniderlab/CLM
  tools:
  - DeepMet
  - Python
  - Python 3.10
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cosine-similarity-spectrum-matching

## Summary

Annotates experimental MS/MS spectra by cosine similarity spectral matching against in silico predicted spectral libraries, and quantifies annotation gain from expanding the library with DeepMet-predicted metabolite structures.

## When to use

Use when experimental MS/MS spectra (e.g. 29.1M spectra from human blood analyses on MetaboLights) must be annotated by spectral matching against a predicted in silico library, and you need to quantify how much library expansion (HMDB vs HMDB + DeepMet structures) increases matched spectra at a cosine cutoff.

## When NOT to use

- When the goal is de novo structure elucidation or targeted discovery of predicted metabolites rather than spectral-library annotation of experimental spectra
- When candidate structures must be prioritized using mass-to-charge ratio alone without a spectral library, since the paper treats m/z-based prioritization and library matching as distinct integration steps
- When a genuine experimental reference spectral library (e.g. experimental standards) is available and in silico prediction is unnecessary

## Inputs

- Experimental MS/MS spectra compiled from 4,510 human blood analyses deposited on MetaboLights (29.1 million spectra)
- In silico predicted MS/MS spectral library built from HMDB known metabolite structures only
- In silico predicted MS/MS spectral library built from HMDB plus 2.4 million DeepMet-anticipated metabolite structures

## Outputs

- Count of experimental spectra matched at cosine similarity >= 0.8 for each library
- Fold-change in matched spectra between combined and HMDB-only searches
- msms_annotation_comparison.csv containing match counts and fold-gain comparison

## How to apply

Compile the experimental MS/MS spectra and construct two in silico predicted MS/MS libraries under identical conditions: one restricted to HMDB known metabolite structures, and one combining HMDB with the 2.4 million metabolites anticipated by DeepMet. Search every experimental spectrum against each library with the same cosine similarity spectral matching, scoring matches and retaining those at cosine similarity >= 0.8. Count matched experimental spectra per library and compute the fold-change in matched spectra (combined vs HMDB-only); in the source study this approximately tripled the number of annotated spectra. Because both searches score the identical spectra with the same scoring function, the fold-gain isolates the effect of library expansion rather than the matching algorithm.

## Related tools

- **DeepMet** (Chemical language model used to generate the 2.4 million predicted metabolite structures that expand the in silico MS/MS spectral library) — https://github.com/skinniderlab/CLM
- **Python 3.10** (Runtime for implementing the cosine similarity spectral matching searches and writing the match-count comparison output) — https://github.com/skinniderlab/CLM

## Evaluation signals

- Matched-spectrum counts must be computed with the same cosine similarity scoring for both libraries over the identical set of experimental spectra
- Matches are called at the cosine similarity >= 0.8 threshold; report counts at exactly this cutoff
- Fold-gain (combined / HMDB-only) should approximate ~3x; a value near 1.0 suggests the DeepMet structures were not actually included in the combined library
- Output file msms_annotation_comparison.csv must contain per-library match counts and the computed fold-change

## Limitations

- Predicting MS/MS spectra or retention times from chemical structures remain open problems, so annotation quality depends entirely on the in silico spectrum predictor used to build the libraries
- Only a small fraction of peaks detected in metabolomics can be identified, and library expansion via predicted structures does not guarantee structural correctness of annotations
- DeepMet predictions were trained on human metabolites; applications to evolutionarily distant metabolism (e.g. plants, bacteria) likely require bespoke models, limiting library expansion scope

## Evidence

- [readme] CLM repository provides the Python codebase used to train and evaluate the chemical language models behind the library structures: [evidence span withheld — anchor status: external_doc]
