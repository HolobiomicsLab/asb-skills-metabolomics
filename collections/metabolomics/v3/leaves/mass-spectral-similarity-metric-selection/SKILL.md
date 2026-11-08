---
name: mass-spectral-similarity-metric-selection
description: Use when you have predicted spectra and ground-truth library spectra
  for the same molecules and need to score similarity directly (cosine similarity
  ratios) or via augmented-library matching with recall@10.
when_to_use_negative:
- You need to train or tune the spectral prediction model itself rather than score
  prediction quality against ground truth
- Your spectra include isotope intensity peaks in ion fragments — NEIMS does not model
  them, so similarity ratios may be misleading
- You cannot supply a reasonable molecular mass for queries, so the 5 Da mass-filtering
  option cannot be applied and library matching accuracy will drop
tools:
- name: NEIMS
  role: Supplies the predicted spectra that are inserted into the augmented reference
    library and scored by the similarity metric
  repo: github.com/brain-research/deep-molecular-massspec
- name: Tensorflow
  role: Framework in which the trained MLP model is run to generate predicted spectra
    prior to similarity computation
  repo: github.com/brain-research/deep-molecular-massspec
- name: RDKit
  role: Generates the 4096-length radius-2 Extended Circular Fingerprints (ECFPs)
    used as model inputs to produce the spectra being compared
  repo: github.com/brain-research/deep-molecular-massspec
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acscentsci.9b00085
    title: Rapid Prediction of Electron–Ionization Mass Spectrometry Using Neural
      Networks
  source_content_hash: sha256:0b937dcef60be810145d740e517963eb79a96a178be593a7b6d9cbf369c9d3c4
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - NEIMS
  - RDKit
  - Tensorflow
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# mass-spectral-similarity-metric-selection

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Quantify agreement between predicted and ground-truth electron-ionization mass spectra using weighted cosine similarity, both directly (predicted-vs-ground-truth similarity ratios) and indirectly via library matching (recall@10). Provides the metric and filtering conventions used to validate NEIMS spectral predictions.

## When to use

Use when you have predicted spectra and ground-truth library spectra for the same molecules and need to score similarity directly (cosine similarity ratios) or via augmented-library matching with recall@10.

## When NOT to use

- You need to train or tune the spectral prediction model itself rather than score prediction quality against ground truth
- Your spectra include isotope intensity peaks in ion fragments — NEIMS does not model them, so similarity ratios may be misleading
- You cannot supply a reasonable molecular mass for queries, so the 5 Da mass-filtering option cannot be applied and library matching accuracy will drop

## Inputs

- Query mass spectra (e.g., NIST 2017 replicates library, noisier spectra for molecules in the NIST main library)
- Ground-truth reference spectra (NIST main library / NIST 2017 Mass Spectral Main Library)
- NEIMS-predicted spectra for the query molecules (generated with the trained TensorFlow MLP model)
- Molecular masses of query molecules (for the 5 Da mass filter)

## Outputs

- Per-query weighted cosine similarity scores against every spectrum in the augmented library
- Rank of the correct spectrum for each query
- recall@10 (percentage of queries whose correct spectrum ranks <= 10)
- Distribution of predicted-similarity / overall-similarity ratios for direct prediction-quality assessment

## How to apply

Load the query spectra (e.g., NIST 2017 replicates library) and the ground-truth spectra (NIST main library), then build an augmented reference library by removing the query molecules' spectra from the main library and inserting model-predicted spectra in their place. Compute the weighted cosine similarity (eq 1) between each query spectrum and every spectrum in the library, applying a mass filter with a 5 Da tolerance of the query molecule's mass (this reduces the library to a median of 6696 spectra per query). For library matching, record the rank of the correct spectrum per query and aggregate into recall@10, the percentage of queries whose correct spectrum ranks <= 10 (NEIMS reports 91.8%). For direct assessment of prediction quality, compute the ratio of the predicted spectrum's similarity to the ground-truth spectrum over the overall similarity; for the MLP bidirectional model roughly half of molecules exceed a ratio of 0.9, which serves as a sanity benchmark.

## Related tools

- **NEIMS** (Supplies the predicted spectra that are inserted into the augmented reference library and scored by the similarity metric) — github.com/brain-research/deep-molecular-massspec
- **Tensorflow** (Framework in which the trained MLP model is run to generate predicted spectra prior to similarity computation) — github.com/brain-research/deep-molecular-massspec
- **RDKit** (Generates the 4096-length radius-2 Extended Circular Fingerprints (ECFPs) used as model inputs to produce the spectra being compared) — github.com/brain-research/deep-molecular-massspec

## Examples

```
python make_spectra_prediction.py --input_file=examples/pentachlorobenzene.sdf --output_file=/tmp/annotated.sdf --weights_dir=$MODEL_WEIGHTS_DIR/massspec_weights
```

## Evaluation signals

- recall@10 from the augmented-library matching should be consistent with the paper's reported 91.8%
- Applying the 5 Da mass filter should reduce the per-query library to a median of 6696 spectra
- For the MLP bidirectional model, roughly half of the molecules should show a predicted/overall similarity ratio greater than 0.9
- Each query's similarity must be computed against every spectrum in the augmented library, with the correct spectrum's rank recorded for aggregation

## Limitations

- NEIMS currently does not have a method to model intensity peaks corresponding to isotopes in ion fragments, which biases similarity scores for molecules with prominent isotope patterns
- Mass filtering improves matching only for experimental setups where the molecular mass of the sample is known with some accuracy
- Recall@10 below the reported 91.8% may reflect prediction gaps rather than metric errors — roughly half of molecules have similarity ratios indicating potential for further improvement

## Evidence

- [readme] In this task we identify molecules by matching its spectra to a library of labeled spectra.: [evidence span withheld — anchor status: external_doc]
