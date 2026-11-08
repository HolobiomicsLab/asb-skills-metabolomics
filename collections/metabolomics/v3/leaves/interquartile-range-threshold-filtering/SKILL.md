---
name: interquartile-range-threshold-filtering
description: Use when you have multiple stochastic predictions (e.g., MC-dropout embeddings)
  per compound pair and must decide which similarity scores are reliable enough to
  use downstream, or quantify the error-vs-retrieval tradeoff.
when_to_use_negative:
- Predictions come from a single deterministic score per pair with no ensemble/dropout
  distribution, so no IQR can be computed
- The analysis requires full coverage of all compound pairs — filtering at IQR < 0.025
  discards ~75% of scores, which is unacceptable for exhaustive retrieval
- Input spectra are from compound classes heavily underrepresented in the training
  library; the article notes spectral libraries are biased toward certain compound
  classes, so uncertainty filtering will not fix systematic bias
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3473
tools:
- name: MS2DeepScore
  role: Siamese neural network whose MC-dropout embeddings (dropout rate 0.2, n=10)
    produce the per-pair score distributions that are filtered by IQR
  repo: https://github.com/matchms/ms2deepscore
- name: matchms
  role: Python library for spectrum import/handling and the cleaning pipeline applied
    before computing MS2DeepScore similarity matrices
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:6bdcb81b1cc4b5b40ba27e0d4e78cab29665357c6d7d311cefcba0f5cc4ed210
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore
  - matchms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# interquartile-range-threshold-filtering

## Summary

Filters Monte-Carlo dropout ensemble Tanimoto score predictions (from MS2DeepScore Siamese-network embeddings) by an interquartile range (IQR) uncertainty threshold, retaining only confident scores. This trades retrieval rate for accuracy: filtering at IQR < 0.025 discards ~75% of scores but lowers RMSE from ~0.17 to ~0.11.

## When to use

Use when you have multiple stochastic predictions (e.g., MC-dropout embeddings) per compound pair and must decide which similarity scores are reliable enough to use downstream, or quantify the error-vs-retrieval tradeoff.

## When NOT to use

- Predictions come from a single deterministic score per pair with no ensemble/dropout distribution, so no IQR can be computed
- The analysis requires full coverage of all compound pairs — filtering at IQR < 0.025 discards ~75% of scores, which is unacceptable for exhaustive retrieval
- Input spectra are from compound classes heavily underrepresented in the training library; the article notes spectral libraries are biased toward certain compound classes, so uncertainty filtering will not fix systematic bias

## Inputs

- Trained MS2DeepScore Siamese model (e.g., pre-trained model from Zenodo record 4699300 dataset; v0.2.0 in the paper)
- Test/query MS/MS spectra (GNPS-derived; paper test set: 3601 spectra of 500 unseen compounds)
- Ground-truth Tanimoto similarity scores (e.g., from RDKit Morgan-2/Morgan-3 circular fingerprints) for RMSE evaluation

## Outputs

- Per compound pair: median Tanimoto score (point estimate) and IQR (uncertainty measure) over 10x10 MC-dropout predictions
- Filtered score subset at each IQR threshold, with per-threshold RMSE and retention fraction (error-vs-retrieval curve)

## How to apply

Run Monte-Carlo dropout inference with dropout rate 0.2, generating n=10 stochastic embeddings per spectrum so each compound pair yields a 10x10 set of Tanimoto score predictions. For each compound pair, take the median as the point estimate and the IQR of the 100 predictions as the uncertainty measure. Sweep increasingly stringent IQR thresholds (e.g., down to IQR < 0.025) and at each threshold record the remaining prediction error (RMSE against true Tanimoto scores) and the fraction of scores retained. The article shows this reduces average RMSE from ~0.17 to ~0.11 on the 3601-spectrum test set (500 unseen GNPS-derived compounds, Zenodo record 4699300), with stricter thresholds discarding progressively more scores. Select the threshold based on whether the application tolerates losing coverage (the strictest threshold discards ~75% of scores) or needs high retrieval rate.

## Related tools

- **MS2DeepScore** (Siamese neural network whose MC-dropout embeddings (dropout rate 0.2, n=10) produce the per-pair score distributions that are filtered by IQR) — https://github.com/matchms/ms2deepscore
- **matchms** (Python library for spectrum import/handling and the cleaning pipeline applied before computing MS2DeepScore similarity matrices) — https://github.com/matchms/matchms

## Examples

```
from ms2deepscore import MS2DeepScore; from matchms.Pipeline import Pipeline, create_workflow; from matchms.filtering.default_pipelines import DEFAULT_FILTERS; pipeline = Pipeline(create_workflow(query_filters=DEFAULT_FILTERS, score_computations=[[MS2DeepScore, {'model': model}]])); report = pipeline.run(spectrum_file_name); similarity_matrix = pipeline.scores.to_array()
```

## Evaluation signals

- Unfiltered median predictions show RMSE ~0.15-0.17 against true Tanimoto scores on the 3601-spectrum test set
- Filtered subset (IQR < 0.025) reaches RMSE of about 0.1 (~0.11), consistent with the paper's uncertainty-based selection result
- Retention fraction decreases monotonically as the IQR threshold tightens; at IQR < 0.025 roughly 75% of scores are discarded, so a strict threshold with high retention indicates a bug (e.g., IQR computed over the wrong axis)
- IQR must be computed over the 10x10 = 100 predictions per compound pair (n=10 dropout embeddings per spectrum), not over the embedding dimensions
- Point estimates are the per-pair medians of the stochastic predictions — medians of spectra pairs of the same compound pair were shown in the paper to improve reliability over individual predictions

## Limitations

- Scores are not deterministic and may change when a new model is trained (different parameters or training data), so absolute IQR thresholds may not transfer across model versions
- The paper could not significantly decrease the overall Tanimoto prediction error with deeper/wider networks or less restrictive binning; filtering improves error only on the retained subset, not globally
- It remains difficult to discriminate high Tanimoto scores (0.8-0.9) from a near-complete chemical match, so even low-uncertainty filtered predictions have limited resolution at the top of the score range
- The paper's pre-trained model results used MS2DeepScore 0.2.0 (Python 3.7/3.8); current library versions (2.0+, Python 3.11/3.12 per README) train an additional embedding evaluator, so the reported numbers may not reproduce out-of-the-box on the latest release

## Evidence

- [results] MC dropout per-spectrum embeddings with dropout rate 0.2: "For a given spectrum, we compute n different embeddings, each from a slightly different version of the base neural network where 20% of its nodes are silenced (dropout rate = 0.2)."
- [results] Filtering scores by increasingly stringent IQR thresholds reduces error at the cost of retrieval: "We then filtered out scores, according to increasingly stringent interquartile range (IQR) thresholds"
- [abstract] Uncertainty estimate selects a subset of predictions with RMSE about 0.1: "the prediction uncertainty estimate can be used to select a subset of predictions with a root mean squared error of about 0.1."
- [readme] MS2DeepScore embedding evaluator provides per-spectrum accuracy prediction: [evidence span withheld — anchor status: external_doc]
