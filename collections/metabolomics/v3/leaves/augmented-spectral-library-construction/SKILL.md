---
name: augmented-spectral-library-construction
description: Use when unknown EI mass spectra must be identified against a library
  that lacks the query molecule, and predicted spectra can fill coverage gaps.
when_to_use_negative:
- The query molecule is already present in the NIST main library, so an unaugmented
  library match suffices
- Input spectra were acquired with an ionization method other than electron ionization,
  unless the model is retrained for that method
- The analysis requires modeling isotope intensity peaks in ion fragments, which NEIMS
  cannot produce
tools:
- name: NEIMS
  role: Lightweight neural network framework used to generate predicted spectra for
    large numbers of molecular candidates that populate the augmented library
  repo: github.com/brain-research/deep-molecular-massspec
- name: Tensorflow
  role: Framework used to construct, train, and run the NEIMS model that produces
    the predicted spectra inserted into the augmented library
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acscentsci.9b00085
    title: Rapid Prediction of Electron–Ionization Mass Spectrometry Using Neural
      Networks
  source_content_hash: sha256:505614063be336d385d760b83c4b841c7aa1fa7c034bc9e9ee0be4c7c4857468
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - NEIMS
  - Tensorflow
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# augmented-spectral-library-construction

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Builds an augmented reference library for mass-spectral library matching by removing observed spectra of query-set molecules from the NIST main library and replacing them with NEIMS-predicted spectra, enabling retrieval of compounds not present in the library.

## When to use

Use when unknown EI mass spectra must be identified against a library that lacks the query molecule, and predicted spectra can fill coverage gaps.

## When NOT to use

- The query molecule is already present in the NIST main library, so an unaugmented library match suffices
- Input spectra were acquired with an ionization method other than electron ionization, unless the model is retrained for that method
- The analysis requires modeling isotope intensity peaks in ion fragments, which NEIMS cannot produce

## Inputs

- NIST 2017 Mass Spectral Main Library spectra (ground truth)
- NIST 2017 replicates library query spectra
- NEIMS-predicted spectra for query-set molecules
- Molecular masses of query molecules (for mass filtering)

## Outputs

- Augmented reference library combining observed and predicted spectra
- Per-query similarity scores against the augmented library
- Rank of the correct spectrum per query
- recall@10 percentage

## How to apply

Train or load the NEIMS MLP (ECFP fingerprints, seven layers of 2000 nodes with residual connections, ReLU activation, dropout 0.25) and generate predicted spectra for the query molecules at roughly 5 ms each. Edit the NIST main library by removing spectra corresponding to the query-set molecules and inserting the NEIMS predictions in their place, forming an augmented reference library. Compute the weighted cosine similarity (eq 1) between each query spectrum and every spectrum from the augmented library, with the mass filter set to a tolerance of 5 Da of the query molecule's mass, which reduces the library to a median of 6696 spectra per query. Record the rank of the correct spectrum and aggregate recall@10; the article validates this setup with a reported recall-at-10 accuracy of 91.8%.

## Related tools

- **NEIMS** (Lightweight neural network framework used to generate predicted spectra for large numbers of molecular candidates that populate the augmented library) — github.com/brain-research/deep-molecular-massspec
- **Tensorflow** (Framework used to construct, train, and run the NEIMS model that produces the predicted spectra inserted into the augmented library)

## Examples

```
python make_spectra_prediction.py --input_file=examples/pentachlorobenzene.sdf --output_file=/tmp/annotated.sdf --weights_dir=$MODEL_WEIGHTS_DIR/massspec_weights
```

## Evaluation signals

- The augmented library contains NEIMS predictions in place of the query-set molecules' spectra removed from the NIST main library
- Recall@10 for the augmented library match is consistent with the paper's reported 91.8% for the MLP bidirectional model
- Mass filter with 5 Da tolerance reduces the library size to a median of 6696 spectra per query molecule
- Prediction throughput is on the order of 5 ms per molecule
- Roughly half of the molecules show a predicted-similarity-to-overall-similarity ratio greater than 0.9, indicating room for further model improvement

## Limitations

- NEIMS currently does not have a method to model intensity peaks corresponding to isotopes in ion fragments
- Correct identification is impossible if the sample molecule is absent from both the observed library and the augmented predictions
- The augmented-library approach presumes the molecular mass of the sample is known with some accuracy for effective mass filtering

## Evidence

- [methods] Construct augmented reference library: "an augmented reference library consisting of a combination of observed spectra and model-predicted spectra"
- [results] Mass filter with 5 Da tolerance of query molecule's mass: "The mass filter was set to a tolerance of 5 Da of the query molecule’s mass; this reduces the size of the library to a median of 6696 spectra for each query molecule"
- [abstract] A lightweight neural network model predicts mass spectra for small molecules averaging 5 ms per molecule with recall-at-10 accuracy of 91.8%: "We contribute a lightweight neural network model that quickly predicts mass spectra for small molecules, averaging 5 ms per molecule with a recall-at-10 accuracy of 91.8%."
- [discussion] NEIMS lacks a method to model isotope intensity peaks in ion fragments: "NEIMS currently does not have a method to model intensity peaks corresponding to isotopes in ion fragments."
- [readme] To test our predictions, we replace portions of the library with spectra predictions from our model.: [evidence span withheld — anchor status: external_doc]
