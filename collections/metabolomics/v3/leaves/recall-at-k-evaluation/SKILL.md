---
name: recall-at-k-evaluation
description: Use when a query set of replicate mass spectra must be identified against
  a reference library and you need a rank-based accuracy metric (e.g. recall@10) to
  judge library augmentation or matching quality.
when_to_use_negative:
- The query molecule is absent from both the reference library and the set of molecules
  with predicted spectra — correct identification is impossible and recall is meaningless
  for those queries
- You need per-spectrum prediction fidelity rather than rank-based identification
  accuracy — use direct spectrum-similarity analysis instead
- Comparing against libraries using molecular formula filtering (e.g. the CFM-EI molecular
  formula filter) rather than the 5 Da mass tolerance — results are not directly comparable
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: NEIMS
  role: Bidirectional MLP model generating the predicted spectra that populate the
    augmented library being evaluated
  repo: github.com/brain-research/deep-molecular-massspec
- name: Tensorflow
  role: Framework used to construct, train, and run the NEIMS model whose predictions
    are matched during evaluation
- name: RDKit Cheminformatics package
  role: Generates the additive ECFP fingerprints (length 4096, radius 2) fed to the
    model for query-set molecules
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_005
  - task_007
  source_papers:
  - doi: 10.1021/acscentsci.9b00085
    title: Rapid Prediction of Electron–Ionization Mass Spectrometry Using Neural
      Networks
  source_content_hash: sha256:54ddf4278e99a331a4163df8ab4702560ab92dfb3ecc7914c6caef2e3c23d763
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Adam optimizer
  - NEIMS
  - RDKit Cheminformatics package
  - Tensorflow
  - Vizier
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# recall-at-k-evaluation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Evaluates library-matching identification accuracy by computing recall@k — the percentage of query spectra whose correct molecule ranks within the top k hits — using weighted cosine similarity against an augmented spectral reference library.

## When to use

Use when a query set of replicate mass spectra must be identified against a reference library and you need a rank-based accuracy metric (e.g. recall@10) to judge library augmentation or matching quality.

## When NOT to use

- The query molecule is absent from both the reference library and the set of molecules with predicted spectra — correct identification is impossible and recall is meaningless for those queries
- You need per-spectrum prediction fidelity rather than rank-based identification accuracy — use direct spectrum-similarity analysis instead
- Comparing against libraries using molecular formula filtering (e.g. the CFM-EI molecular formula filter) rather than the 5 Da mass tolerance — results are not directly comparable

## Inputs

- NIST 2017 replicates library query spectra (held-out test half)
- Augmented reference library: NIST main library spectra plus NEIMS predictions substituted for query-set molecules
- Molecular masses of query molecules (for the 5 Da mass filter)

## Outputs

- Per-query rank of the correct spectrum (e.g. mass_filter_results.csv with ranks and recall statistics)
- Recall@10 score (percentage of queries with correct hit ranked ≤ 10)

## How to apply

Match each query spectrum from the NIST 2017 replicates library against the augmented reference library (NIST main library with query molecules' observed spectra replaced by NEIMS-predicted spectra) using the weighted cosine similarity of eq 1. Apply the mass filter with a 5 Da tolerance of the query molecule's mass, which reduces the candidate library to a median of 6696 spectra per query. Record the rank of the correct spectrum for every query, then compute recall@10 as the percentage of queries with rank ≤ 10. Rationale: recall@10 directly measures whether library matching would return the correct molecule within the top hits a practitioner would inspect, making it the optimization target for hyperparameter tuning (e.g. with Vizier) and the acceptance criterion for the augmented library.

## Related tools

- **NEIMS** (Bidirectional MLP model generating the predicted spectra that populate the augmented library being evaluated) — github.com/brain-research/deep-molecular-massspec
- **Tensorflow** (Framework used to construct, train, and run the NEIMS model whose predictions are matched during evaluation)
- **RDKit Cheminformatics package** (Generates the additive ECFP fingerprints (length 4096, radius 2) fed to the model for query-set molecules)

## Examples

```
python make_spectra_prediction.py --input_file=examples/pentachlorobenzene.sdf --output_file=/tmp/annotated.sdf --weights_dir=$MODEL_WEIGHTS_DIR/massspec_weights
```

## Evaluation signals

- Computed recall@10 matches the reported 91.7–91.8% on the NIST 2017 test split
- Applying the 5 Da mass filter improves recall@10 over the unfiltered augmented library (mass filtering contributes ~6 percentage points)
- Per-query rank file contains a valid integer rank for every query in the held-out test half
- Recall@10 is computed as the percentage of queries whose correct spectrum has rank ≤ 10, not rank < 10
- Median filtered library size is around 6696 spectra per query molecule when the 5 Da tolerance is applied

## Limitations

- NEIMS currently does not have a method to model intensity peaks corresponding to isotopes in ion fragments
- Recall@10 is a rank-based, indirect measure — roughly half of molecules still have a predicted-to-observed similarity ratio below 0.9, indicating room for model improvement not captured by recall alone
- Evaluation assumes query molecules are present in the NIST main library (their observed spectra are removed and replaced with predictions); queries outside library coverage cannot be correctly identified

## Evidence

- [methods] recall@10 definition and threshold: [evidence span withheld — anchor status: not_found]
- [readme] repository availability for reproduction: [evidence span withheld — anchor status: external_doc]
