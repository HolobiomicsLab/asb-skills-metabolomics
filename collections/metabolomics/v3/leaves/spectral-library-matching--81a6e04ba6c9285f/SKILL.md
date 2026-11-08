---
name: spectral-library-matching
description: Use when query spectra (e.g., NIST 2017 replicates) must be identified
  against a reference library whose coverage is extended with model-predicted spectra,
  and ranked-match accuracy (recall@10) is the target metric.
when_to_use_negative:
- The sample molecule's molecular mass is unknown with reasonable accuracy, so the
  5 Da mass filter cannot be applied to prune candidates
- Spectra come from a non-electron-ionization method, since the trained NEIMS model
  was fit to EI fragmentation patterns (retraining would be required)
- Isotope intensity peaks in ion fragments are critical to the match, as NEIMS currently
  does not model isotope peaks
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: NEIMS
  role: Supplies the predicted spectra used to augment the reference library before
    matching
  repo: github.com/brain-research/deep-molecular-massspec
- name: Tensorflow
  role: Constructs and runs the trained MLP model that generates predicted spectra
    for library augmentation
  repo: github.com/brain-research/deep-molecular-massspec
- name: RDKit Cheminformatics package
  role: Generates the 4096-length radius-2 additive ECFP molecular fingerprints that
    feed spectral prediction and supply query molecular mass for filtering
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_005
  source_papers:
  - doi: 10.1021/acscentsci.9b00085
    title: Rapid Prediction of Electron–Ionization Mass Spectrometry Using Neural
      Networks
  source_content_hash: sha256:8d2d4e8ce20d6bdc40cc2fcef5b6dda7d7bb8fbd36fed114528c6fb5d10f1892
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Adam optimizer
  - NEIMS
  - RDKit Cheminformatics package
  - Tensorflow
  - Vizier
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# spectral-library-matching

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Identifies unknown molecules by matching observed EI mass spectra against an augmented reference library (NIST main library with NEIMS-predicted spectra substituted) using weighted cosine similarity plus a 5 Da molecular-mass filter, scoring recall@10.

## When to use

Use when query spectra (e.g., NIST 2017 replicates) must be identified against a reference library whose coverage is extended with model-predicted spectra, and ranked-match accuracy (recall@10) is the target metric.

## When NOT to use

- The sample molecule's molecular mass is unknown with reasonable accuracy, so the 5 Da mass filter cannot be applied to prune candidates
- Spectra come from a non-electron-ionization method, since the trained NEIMS model was fit to EI fragmentation patterns (retraining would be required)
- Isotope intensity peaks in ion fragments are critical to the match, as NEIMS currently does not model isotope peaks

## Inputs

- NIST 2017 replicates library query spectra (noisier spectra of molecules present in the NIST main library)
- NIST 2017 Mass Spectral Main Library ground-truth spectra (240,942 training spectra; augmented reference library source)
- NEIMS-predicted spectra per query molecule from the trained TensorFlow MLP (bidirectional prediction)
- Molecular mass of each query molecule (for the 5 Da mass filter)

## Outputs

- Per-query rank of the correct candidate spectrum in the augmented library
- recall@10 statistic (paper reports 91.8% for bidirectional NEIMS)
- Results tables (e.g., results_table.csv, mass_filter_results.csv) of per-query ranks and recall statistics

## How to apply

Build the augmented reference library by removing spectra of query-set molecules from the NIST main library and replacing them with NEIMS predictions (MLP over 4096-length radius-2 additive ECFPs, TensorFlow). For each query spectrum, compute the weighted cosine similarity (eq 1) between it and every candidate spectrum in the augmented library. Apply the mass filter with a tolerance of 5 Da of the query molecule's mass, which reduces the library to a median of 6696 candidate spectra per query. Record the rank of the correct candidate spectrum for every query and compute recall@10, the percentage of queries whose correct spectrum ranks <= 10; the paper reports 91.8% for the full bidirectional NEIMS model. Optionally compare forward-only, reverse-only, and bidirectional prediction modes to isolate the contribution of the bidirectional architecture.

## Related tools

- **NEIMS** (Supplies the predicted spectra used to augment the reference library before matching) — github.com/brain-research/deep-molecular-massspec
- **Tensorflow** (Constructs and runs the trained MLP model that generates predicted spectra for library augmentation) — github.com/brain-research/deep-molecular-massspec
- **RDKit Cheminformatics package** (Generates the 4096-length radius-2 additive ECFP molecular fingerprints that feed spectral prediction and supply query molecular mass for filtering)

## Examples

```
python make_spectra_prediction.py --input_file=examples/pentachlorobenzene.sdf --output_file=/tmp/annotated.sdf --weights_dir=$MODEL_WEIGHTS_DIR/massspec_weights
```

## Evaluation signals

- Recall@10 reaches ~91.8% for the full bidirectional NEIMS on the NIST 2017 replicates query set
- Applying the 5 Da mass filter improves recall@10 relative to the unfiltered augmented library
- Median candidate-library size per query after mass filtering is ~6696 spectra
- Bidirectional prediction mode exceeds both forward-only and reverse-only modes (bidirectional improves recall@10 by 30% over forward-only for linear-regression and MLP models)
- Each query's correct spectrum ranks <= 10 in the augmented library for the reported fraction of queries

## Limitations

- NEIMS does not model intensity peaks corresponding to isotopes in ion fragments, which can shift matches near molecular-mass offsets
- The mass filter assumes the molecular mass of the sample is known with some accuracy (within the 5 Da tolerance)
- For roughly half of molecules the predicted-to-overall similarity ratio exceeds 0.9, indicating room for further model improvement
- Query-set spectra from the NIST 2017 replicates library are noisier than the main-library spectra they are matched against

## Evidence
