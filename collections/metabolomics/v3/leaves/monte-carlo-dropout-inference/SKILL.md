---
name: monte-carlo-dropout-inference
description: Use when MS2DeepScore Tanimoto predictions must carry per-pair uncertainty,
  or when uncertain predictions need to be filtered out to lower RMSE at the cost
  of retrieval rate.
when_to_use_negative:
- When a single deterministic similarity score is needed with no uncertainty estimate
  — plain MS2DeepScore inference without dropout is cheaper and sufficient
- When the model version is not the one trained on ~100,000 GNPS spectra (Zenodo record
  4699300); scores are not deterministic across retrained models and IQR behavior
  may differ
- When the goal is exact compound identification rather than structural similarity
  — high Tanimoto scores (0.8–0.9) cannot be discriminated from near-complete chemical
  match
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MS2DeepScore
  role: Siamese neural network whose base embedding network is run stochastically
    with dropout rate 0.2 at inference to produce n embeddings and n×n Tanimoto predictions
    per compound pair
  repo: https://github.com/matchms/ms2deepscore
- name: matchms
  role: Python library for MS/MS spectrum import, filtering, and handling before and
    around similarity computations
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:2e7e3b341780f598a3ac3df0ec9d6341ab0cee9d220a3fd86f145f8c11dc69f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore
  - matchms
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# monte-carlo-dropout-inference

## Summary

Quantifies prediction uncertainty in MS2DeepScore by running the Siamese base network multiple times with dropout (rate 0.2) active at inference, yielding n embeddings per spectrum and n×n Tanimoto predictions per compound pair whose spread (IQR) estimates reliability.

## When to use

Use when MS2DeepScore Tanimoto predictions must carry per-pair uncertainty, or when uncertain predictions need to be filtered out to lower RMSE at the cost of retrieval rate.

## When NOT to use

- When a single deterministic similarity score is needed with no uncertainty estimate — plain MS2DeepScore inference without dropout is cheaper and sufficient
- When the model version is not the one trained on ~100,000 GNPS spectra (Zenodo record 4699300); scores are not deterministic across retrained models and IQR behavior may differ
- When the goal is exact compound identification rather than structural similarity — high Tanimoto scores (0.8–0.9) cannot be discriminated from near-complete chemical match

## Inputs

- Trained MS2DeepScore Siamese model (200-dimensional embedding space)
- Test set of processed MS/MS spectra (GNPS-derived; 3601 spectra of 500 unseen compounds, binned 10–1000 m/z into 10,000 bins, square-root transformed intensities)
- Ground-truth Tanimoto scores (RDKit Morgan/Daylight fingerprints) for the same compound pairs

## Outputs

- n×n matrix of stochastic Tanimoto predictions per compound pair (n=10)
- Median Tanimoto score per compound pair (point estimate)
- IQR per compound pair (uncertainty estimate)
- Filtered prediction subset with corresponding RMSE and retention rate at each IQR threshold

## How to apply

For each test spectrum (e.g. the 3601 GNPS-derived test spectra from 500 unseen compounds), compute n=10 stochastic 200-dimensional embeddings from the base Siamese network with dropout rate 0.2 applied at inference. Pair all embeddings across a compound pair to obtain 10×10 = 100 Tanimoto score predictions; take the median as the point estimate and the interquartile range (IQR) as the uncertainty measure. Aggregate over different spectral pairs of the same compound pair, since median aggregation of per-pair scores improves prediction reliability. Optionally filter scores by increasingly stringent IQR thresholds (e.g. IQR < 0.025 discards ~75% of scores) to trade retrieval rate for lower prediction error (RMSE from ~0.17 down to ~0.11).

## Related tools

- **MS2DeepScore** (Siamese neural network whose base embedding network is run stochastically with dropout rate 0.2 at inference to produce n embeddings and n×n Tanimoto predictions per compound pair) — https://github.com/matchms/ms2deepscore
- **matchms** (Python library for MS/MS spectrum import, filtering, and handling before and around similarity computations) — https://github.com/matchms/matchms

## Examples

```
from ms2deepscore import MS2DeepScore; ms2ds_model = MS2DeepScore(model); ms2ds_embeddings = ms2ds_model.get_embedding_array(cleaned_spectra)
```

## Evaluation signals

- Dropout rate at inference is 0.2 with n=10 embeddings per spectrum, giving 10×10=100 predictions per compound pair
- Median of the 100 predictions is used as the point estimate; IQR as the uncertainty
- At IQR < 0.025 threshold, roughly 75% of scores are discarded and RMSE drops from ~0.17 to ~0.11
- Retained prediction subset should reach RMSE of about 0.1, consistent with the reported uncertainty-based selection
- Overall unfiltered prediction error should fall in the ~0.13–0.2 RMSE range for Tanimoto scores between 0.1 and 0.9

## Limitations

- A score itself is not deterministic but might change when a new model is trained (different parameters or training data)
- Uncertainty filtering reduces error only by discarding scores — it comes at the cost of a lower retrieval rate
- Spectral libraries are inherently biased toward certain compound classes; spectra come from different instrumentation types with varying quality and unbalanced compound classes

## Evidence

- [results] For a given spectrum, we compute n different embeddings, each from a slightly different version of the base neural network where 20% of its nodes are silenced (dropout rate = 0.2).: "For a given spectrum, we compute n different embeddings, each from a slightly different version of the base neural network where 20% of its nodes are silenced (dropout rate = 0.2)."
- [abstract] the prediction uncertainty estimate can be used to select a subset of predictions with a root mean squared error of about 0.1.: "the prediction uncertainty estimate can be used to select a subset of predictions with a root mean squared error of about 0.1."
- [results] We then filtered out scores, according to increasingly stringent interquartile range (IQR) thresholds: "We then filtered out scores, according to increasingly stringent interquartile range (IQR) thresholds"
- [readme] In addition to the prediction of a structural similarity, MS2DeepScore can also make use of an embedding evaluator predict the models accuracy for each spectrum.: [evidence span withheld — anchor status: external_doc]
