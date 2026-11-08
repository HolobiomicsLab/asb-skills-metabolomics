---
name: recall-at-k-metric-computation
description: Use when evaluating spectral library matching where the correct reference
  spectrum's rank must be summarized as recall@k (e.g., recall@10) over a query set,
  such as comparing forward vs bidirectional NEIMS prediction modes.
when_to_use_negative:
- You need per-peak spectral prediction accuracy (e.g., mean-squared error on intensity
  vectors) rather than retrieval rank quality in a library-matching task
- The reference library has not been augmented with predicted spectra (no coverage
  gap being evaluated), so ranking metrics against the original library answer a different
  question
- Query spectra have no known ground-truth matches in the library, making 'rank of
  the correct candidate' undefined
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: Tensorflow
  role: Builds and runs the NEIMS model variants whose predicted spectra are inserted
    into the augmented library before recall@10 is computed
  repo: github.com/brain-research/deep-molecular-massspec
- name: RDKit Cheminformatics package
  role: Generates the additive ECFP fingerprints (length 4096, radius 2) that are
    the model input producing the predicted spectra being ranked
  repo: github.com/brain-research/deep-molecular-massspec
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acscentsci.9b00085
    title: Rapid Prediction of Electron–Ionization Mass Spectrometry Using Neural
      Networks
  source_content_hash: sha256:faeba14270affca60d4a11ff24295be830d80719ff18e849ad8e9521f73c4730
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Adam optimizer
  - NEIMS
  - RDKit Cheminformatics package
  - Tensorflow
  - Vizier
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# recall-at-k-metric-computation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Computes recall@k (here recall@10) — the fraction of query molecules whose correct reference spectrum ranks at or below rank k under a similarity-ordered library match — as the primary accuracy metric for evaluating predicted EI mass spectra against an augmented spectral library.

## When to use

Use when evaluating spectral library matching where the correct reference spectrum's rank must be summarized as recall@k (e.g., recall@10) over a query set, such as comparing forward vs bidirectional NEIMS prediction modes.

## When NOT to use

- You need per-peak spectral prediction accuracy (e.g., mean-squared error on intensity vectors) rather than retrieval rank quality in a library-matching task
- The reference library has not been augmented with predicted spectra (no coverage gap being evaluated), so ranking metrics against the original library answer a different question
- Query spectra have no known ground-truth matches in the library, making 'rank of the correct candidate' undefined

## Inputs

- Per-query ranked candidate lists with the rank of the correct spectrum (output of weighted-cosine library matching on the augmented NIST main library)
- Weighted cosine similarity scores between each query spectrum (NIST 2017 replicates library) and every filtered reference-library spectrum
- Mass-filtered reference library (5 Da tolerance; median 6696 spectra per query)

## Outputs

- recall@10 value per prediction mode (forward, reverse, bidirectional) as a fraction in [0,1]
- results_table.csv tabulating recall@10 across model variants

## How to apply

For each query molecule (from the held-out half of the NIST 2017 replicates library), rank every candidate in the augmented reference library — where query-molecule spectra in the NIST main library have been replaced by model predictions — using the weighted cosine similarity metric (eq 1). Apply the mass filter with a tolerance of 5 Da of the query molecule's mass first; this reduces the library to a median of 6696 spectra per query, so ranks are computed over the filtered candidate set. Record the rank of the correct candidate spectrum for every query, then compute recall@10 as the fraction of queries whose correct spectrum ranks at or below rank 10. Tabulate recall@10 per prediction mode (forward, reverse, bidirectional) and validate against the reference result: the full bidirectional NEIMS achieves 91.8% recall@10.

## Related tools

- **Tensorflow** (Builds and runs the NEIMS model variants whose predicted spectra are inserted into the augmented library before recall@10 is computed) — github.com/brain-research/deep-molecular-massspec
- **RDKit Cheminformatics package** (Generates the additive ECFP fingerprints (length 4096, radius 2) that are the model input producing the predicted spectra being ranked) — github.com/brain-research/deep-molecular-massspec

## Evaluation signals

- Bidirectional NEIMS recall@10 should be approximately 91.8% on the NIST 2017 replicates query set; large deviation indicates a protocol error
- Recall@10 must be a fraction in [0,1] computed over the mass-filtered library (median library size 6696 spectra per query at 5 Da tolerance)
- Bidirectional recall@10 should exceed both forward-only and reverse-only baselines (the paper reports a ~30% improvement from bidirectional prediction)
- Rank of the correct candidate must be recorded for every query, so the number of rank records equals the query-set size
- Recall@10 must be monotonically >= recall@1 for the same ranked lists, as a sanity check on ranking correctness

## Limitations

- NEIMS does not model isotope intensity peaks in ion fragments, which can shift ranks for isotope-containing molecules
- Recall@10 values are tied to this specific evaluation protocol (augmented NIST main library, weighted cosine similarity, 5 Da mass filter) and are not comparable across protocols without matching those settings
- The metric is only meaningful when the correct spectrum is present in the augmented reference library

## Evidence

- [readme] readme: [evidence span withheld — anchor status: external_doc]
