---
name: spectral-similarity-scoring-cosine-jaccard
description: Use when a GC/EI-MS query spectrum (e.g., from SWGDRUG or NIST) has no
  matching reference entry and must be ranked against a set of predicted candidate
  spectra to produce a top-k identification list.
when_to_use_negative:
- When the analyte spectrum already matches a known reference in NIST, Wiley, or SWGDRUG
  libraries — conventional screening suffices
- When no predicted candidate spectra or fingerprints exist for the candidate pool
  (this skill only scores; it does not enumerate or predict)
edam_operation: http://edamontology.org/operation_3553
edam_topics:
- http://edamontology.org/topic_3360
- http://edamontology.org/topic_0121
tools:
- name: NEIMS
  role: Predicts EI mass spectra for enumerated candidate compounds so each candidate
    has a spectrum to compare against the query
  repo: https://github.com/jhhung/PS2MS
- name: DeepEI
  role: Converts the analyte's mass spectrum into its corresponding molecular fingerprint
    for fingerprint-based similarity
  repo: https://github.com/jhhung/PS2MS
- name: PS2MS
  role: Computes the integrated SMSF similarity metric and yields the ranked top-100
    list of potential NPS identities
  repo: https://github.com/jhhung/PS2MS
- name: RASSP
  role: Alternative model that predicts EI spectra for the candidate subset, which
    are then scored with the same cosine+Jaccard+SMSF pipeline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:abc065331a6fa09d5a3923d5915d0fab698063603d5f1ef5bcc55de15924d8cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jhhung/PS2MS
  tools:
  - DeepEI
  - NEIMS
  - PS2MS
  - RASSP
  - python
  edam_operation: http://edamontology.org/operation_3553
  edam_topics:
  - http://edamontology.org/topic_3360
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# spectral-similarity-scoring-cosine-jaccard

## Summary

Ranks candidate compounds against an unknown EI mass spectrum by combining cosine similarity over spectra, Jaccard similarity over top peaks (with ±1 Da tolerance for isotopic variance), and a fingerprint-based weighted-sum metric (SMSF) between a query spectrum and predicted spectra from NEIMS or RASSP.

## When to use

Use when a GC/EI-MS query spectrum (e.g., from SWGDRUG or NIST) has no matching reference entry and must be ranked against a set of predicted candidate spectra to produce a top-k identification list.

## When NOT to use

- When the analyte spectrum already matches a known reference in NIST, Wiley, or SWGDRUG libraries — conventional screening suffices
- When no predicted candidate spectra or fingerprints exist for the candidate pool (this skill only scores; it does not enumerate or predict)

## Inputs

- EI mass spectrum of an unknown analyte (query, e.g., from GC/EI-MS at 70 eV)
- Predicted EI mass spectra for candidate compounds in MSP format (e.g., NEIMS or RASSP predictions)
- Molecular fingerprints for candidates and analyte (generated via DeepEI)

## Outputs

- Ranked list of candidate identities scored by SMSF (integrated similarity)
- Top-1 and top-20 hit-rate metrics for a labeled validation set
- Comparison table of top-k rates across predictors (e.g., rassp_vs_neims_topk_comparison.csv)

## How to apply

For each candidate, compute cosine similarity between the query and predicted EI mass spectrum, plus Jaccard similarity over the top peaks using a ±1 Da tolerance to absorb isotopic variance; adding Jaccard raised top-1 accuracy from below 40% to more than 50%. Convert the query spectrum to a molecular fingerprint with DeepEI and combine spectrum and fingerprint similarities via the SMSF weighted-sum metric, yielding over 90% (NIST) and 85% (SWGDRUG) top-20 hit rates. Optionally, a mass-weight filter using the M+ peak within a tolerance range can narrow the candidate set before scoring. Validate by reporting top-1 and top-20 hit rates against a held-out library mix.

## Related tools

- **NEIMS** (Predicts EI mass spectra for enumerated candidate compounds so each candidate has a spectrum to compare against the query) — https://github.com/jhhung/PS2MS
- **DeepEI** (Converts the analyte's mass spectrum into its corresponding molecular fingerprint for fingerprint-based similarity) — https://github.com/jhhung/PS2MS
- **PS2MS** (Computes the integrated SMSF similarity metric and yields the ranked top-100 list of potential NPS identities) — https://github.com/jhhung/PS2MS
- **RASSP** (Alternative model that predicts EI spectra for the candidate subset, which are then scored with the same cosine+Jaccard+SMSF pipeline)

## Examples

```
python drug-detection/run.py --query query.msp --db synthetic.msp --smsf --topk 20
```

## Evaluation signals

- Top-1 hit rate should exceed 50% on the validation set after Jaccard is incorporated (vs below 40% without it)
- Top-20 hit rate should reach over 90% on NIST and over 85% on SWGDRUG when using SMSF
- Real-world seized Eutylone samples (40 total) should rank within the top 3 in 37/40 cases
- Scores for known positives (cathinone-like) should form a distinct Gaussian distribution from noncathinone-like negatives in SWGDRUG data

## Limitations

- NEIMS-predicted spectra depend on training library (originally NIST-only), limiting generalization to SWGDRUG-style queries
- Isotopic variance is only handled with a coarse ±1 Da tolerance, not an isotopic-aware model
- Success depends on the target compound being reachable within the enumerated derivative space from the template drug core structure

## Evidence

- [methods] Adding Jaccard similarity raises top-1 accuracy: "The proportion of correct prediction appearing at top 1 increased from below 40% to more than 50% after incorporating Jaccard similarity."
- [methods] SMSF top-20 hit rates: "over 90% and 85% of the test substances, respectively, ranked within the top 20 predictions"
- [other] Isotopic tolerance in Jaccard: [evidence span withheld — anchor status: not_found]
- [methods] SMSF definition: [evidence span withheld — anchor status: not_found]
- [abstract] Real-world validation result: "Experimental results affirm the effectiveness of PS2MS in identifying cathinone derivatives within real evidence specimens"
- [discussion] Mass filtering via M+ peak: "utilizes the M+ peak as a reference for the mass weight filtering of enumerated compounds within a certain tolerance range"
- [readme] Workflow context in README: "PS2MS employs DeepEI to predict the fingerprint of the unknown analyte."
