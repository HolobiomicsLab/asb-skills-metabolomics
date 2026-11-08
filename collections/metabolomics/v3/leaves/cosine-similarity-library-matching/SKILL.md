---
name: cosine-similarity-library-matching
description: Use when query EI mass spectra must be identified against a spectral
  library that has coverage gaps, and candidate molecular structures (SMILES/SDF)
  are available so predicted spectra can fill those gaps.
when_to_use_negative:
- The query molecule already has an observed spectrum in the reference library, so
  no coverage gap exists and matching against observed spectra alone suffices
- The molecular mass of the sample cannot be estimated, since the 5 Da mass filter
  that drives the reported recall cannot be applied
- Isotope-resolved identification is required, because NEIMS has no method to model
  intensity peaks corresponding to isotopes in ion fragments
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: NEIMS
  role: Predicts full m/z-intensity vectors for candidate molecules to fill library
    coverage gaps in the augmented reference library
  repo: github.com/brain-research/deep-molecular-massspec
- name: RDKit Cheminformatics package
  role: Generates additive ECFP fingerprints (length 4096, radius 2) encoding each
    candidate molecule for the prediction model
- name: Tensorflow
  role: Constructs and runs the pretrained bidirectional MLP that outputs the predicted
    spectra used in library matching
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acscentsci.9b00085
    title: Rapid Prediction of Electron–Ionization Mass Spectrometry Using Neural
      Networks
  source_content_hash: sha256:551f66127cb2d667573cd469ef9f1edf725c799e15d993925c53ac5da426287e
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Adam optimizer
  - NEIMS
  - RDKit Cheminformatics package
  - Tensorflow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# cosine-similarity-library-matching

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Identify small molecules by matching an observed electron-ionization mass spectrum against an augmented reference library (observed NIST spectra plus NEIMS-predicted spectra) using weighted cosine similarity, restricted by a 5 Da mass filter.

## When to use

Use when query EI mass spectra must be identified against a spectral library that has coverage gaps, and candidate molecular structures (SMILES/SDF) are available so predicted spectra can fill those gaps.

## When NOT to use

- The query molecule already has an observed spectrum in the reference library, so no coverage gap exists and matching against observed spectra alone suffices
- The molecular mass of the sample cannot be estimated, since the 5 Da mass filter that drives the reported recall cannot be applied
- Isotope-resolved identification is required, because NEIMS has no method to model intensity peaks corresponding to isotopes in ion fragments

## Inputs

- Query EI mass spectra (e.g., from the NIST 2017 replicates library)
- Candidate molecular structures for library gaps as SDF or SMILES (e.g., training_splits inchikey/smiles files)
- Observed reference spectra from the NIST main library

## Outputs

- Ranked library-match list per query spectrum (candidate identity by rank, cosine similarity scores)
- recall@10 statistic for the augmented library vs. the observed-only baseline
- Augmented reference library combining observed and NEIMS-predicted spectra

## How to apply

Build the augmented reference library by editing the NIST main library: remove observed spectra for query-set molecules and replace them with spectra predicted by the NEIMS bidirectional MLP (seven layers of 2000 nodes, residual connections, dropout 0.25), which consumes 4096-length radius-2 additive ECFP fingerprints generated with RDKit and takes ~5 ms per molecule. Restrict each search with the mass filter set to a tolerance of 5 Da of the query molecule's mass, which shrinks the library to a median of 6696 spectra per query. Compute the weighted cosine similarity (eq 1) between each query spectrum and every remaining library spectrum, and rank candidates by similarity score. Judge success by recall@10 — the rank of the correct hit among the top 10 — with the acceptance criterion of improved recall@10 over the observed-only baseline (the paper reports 91.7% with the 5 Da mass filter, 91.8% overall).

## Related tools

- **NEIMS** (Predicts full m/z-intensity vectors for candidate molecules to fill library coverage gaps in the augmented reference library) — github.com/brain-research/deep-molecular-massspec
- **RDKit Cheminformatics package** (Generates additive ECFP fingerprints (length 4096, radius 2) encoding each candidate molecule for the prediction model)
- **Tensorflow** (Constructs and runs the pretrained bidirectional MLP that outputs the predicted spectra used in library matching)

## Examples

```
python make_spectra_prediction.py --input_file=examples/pentachlorobenzene.sdf --output_file=/tmp/annotated.sdf --weights_dir=$MODEL_WEIGHTS_DIR/massspec_weights
```

## Evaluation signals

- recall@10 reaches ≥91.7% on the NIST 2017 replicates query set when the 5 Da mass filter is applied, matching the paper's reported value
- After mass filtering, the library size per query is on the order of a median of 6696 spectra
- Observed spectra for query-set molecules are verified absent from the augmented library before matching (otherwise the evaluation is leaked)
- Per-molecule spectral prediction wall-clock time is around 5 ms, confirming the lightweight model is being used rather than an expensive alternative

## Limitations

- NEIMS does not model intensity peaks corresponding to isotopes in ion fragments, which can penalize matching for molecules with distinctive isotope patterns
- Only one peak (the base peak) has exactly matching intensity in the forward/reverse prediction design; overall intensity fidelity is approximate
- The mass filter helps only in experimental setups where the molecular mass of the sample is known with some accuracy

## Evidence

- [results] Mass filter tolerance and library size: "The mass filter was set to a tolerance of 5 Da of the query molecule's mass; this reduces the size of the library to a median of 6696 spectra for each query molecule"
- [methods] Matching procedure: "We then perform library matching and calculate the similarity between each query spectrum and every spectrum from the augmented library."
- [methods] Augmented library construction: "an augmented reference library consisting of a combination of observed spectra and model-predicted spectra"
- [abstract] Reported recall metric: "quickly predicts mass spectra for small molecules, averaging 5 ms per molecule with a recall-at-10 accuracy of 91.8%"
- [readme] README baseline task description: [evidence span withheld — anchor status: external_doc]
