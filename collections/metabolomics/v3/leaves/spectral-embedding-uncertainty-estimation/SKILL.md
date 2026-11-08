---
name: spectral-embedding-uncertainty-estimation
description: Use when MS2DeepScore Tanimoto predictions must be accompanied by a reliability
  measure, or when only the most confident subset of spectrum-pair scores is acceptable
  and a precision/retention trade-off is needed.
when_to_use_negative:
- You only need a single deterministic similarity matrix without per-prediction confidence
  (plain MS2DeepScore scoring suffices)
- You want to train a new Siamese model rather than estimate uncertainty of an existing
  trained model
- Predictions must be deterministic and reproducible across re-trained models — the
  paper notes scores 'might change when a new model is trained'
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MS2DeepScore
  role: Siamese neural network whose base network is run in Monte-Carlo dropout mode
    (dropout rate 0.2) to produce n stochastic embeddings per spectrum
  repo: https://github.com/matchms/ms2deepscore
- name: matchms
  role: Python library for MS/MS spectrum import, cleaning, handling and comparisons
    feeding the embeddings
- name: RDKit
  role: Computes ground-truth Morgan/Daylight molecular fingerprints for Tanimoto-score
    error evaluation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:b8343a67e5a174398921edf7550d42d75b0ec64cd6e54a396891672c3b97c1b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore
  - RDKit
  - matchms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# spectral-embedding-uncertainty-estimation

## Summary

Quantify per-prediction uncertainty of MS2DeepScore Tanimoto-score predictions via Monte-Carlo dropout, then filter predictions by interquartile range (IQR) to trade retrieval rate for lower RMSE.

## When to use

Use when MS2DeepScore Tanimoto predictions must be accompanied by a reliability measure, or when only the most confident subset of spectrum-pair scores is acceptable and a precision/retention trade-off is needed.

## When NOT to use

- You only need a single deterministic similarity matrix without per-prediction confidence (plain MS2DeepScore scoring suffices)
- You want to train a new Siamese model rather than estimate uncertainty of an existing trained model
- Predictions must be deterministic and reproducible across re-trained models — the paper notes scores 'might change when a new model is trained'

## Inputs

- Test set of cleaned MS/MS spectra (e.g. 3601 spectra from 500 unseen compounds, GNPS-derived, Zenodo record 4699300)
- Trained MS2DeepScore model (e.g. ms2deepscore_model.pt)
- Ground-truth Tanimoto scores computed from molecular fingerprints (e.g. Morgan/Daylight in RDKit) for error evaluation

## Outputs

- Median Tanimoto-score prediction and IQR uncertainty value per compound pair
- Filtered score subset with associated RMSE and retention rate at each IQR threshold
- Optionally 200-dimensional embeddings for downstream clustering/visualization

## How to apply

Run stochastic inference with the trained MS2DeepScore Siamese model by keeping dropout active (dropout rate 0.2) and computing n=10 embeddings per spectrum, yielding a 10x10 grid of score predictions per compound pair. Aggregate the 100 predictions per compound pair by taking the median as the point estimate and the IQR as the uncertainty measure. Optionally also median over spectral pairs of the same compound pair, which the paper shows improves reliability versus individual predictions. Then apply increasingly stringent IQR thresholds to discard high-uncertainty scores, recording RMSE (against true Tanimoto scores from RDKit fingerprints) and retained fraction at each threshold; expect ~93% retention at the reported threshold and RMSE dropping from ~0.17 to ~0.1 (RMSE ~0.1 on the retained subset). Newer ms2deepscore 2.0 releases replace manual MC dropout with a trained embedding evaluator that predicts model accuracy per spectrum.

## Related tools

- **MS2DeepScore** (Siamese neural network whose base network is run in Monte-Carlo dropout mode (dropout rate 0.2) to produce n stochastic embeddings per spectrum) — https://github.com/matchms/ms2deepscore
- **matchms** (Python library for MS/MS spectrum import, cleaning, handling and comparisons feeding the embeddings)
- **RDKit** (Computes ground-truth Morgan/Daylight molecular fingerprints for Tanimoto-score error evaluation)

## Examples

```
from ms2deepscore.models import load_model; from ms2deepscore import MS2DeepScore; model = load_model("ms2deepscore_model.pt"); ms2ds_model = MS2DeepScore(model); emb = ms2ds_model.get_embedding_array(cleaned_spectra)
```

## Evaluation signals

- With dropout rate 0.2 and n=10 embeddings, each compound pair yields exactly 100 score predictions (10x10)
- Unfiltered test-set RMSE should be in the ~0.13-0.2 range; filtering at the reported IQR threshold (~0.025) should reduce average RMSE to about 0.1
- Retention curve is monotonic: stricter IQR thresholds retain fewer scores but lower RMSE; report both at each threshold
- Median over spectral pairs of the same compound pair should not degrade (and per the paper should improve) reliability versus individual predictions
- Filtered predictions reach RMSE of about 0.1 on the retained subset at roughly 93% retention, consistent with the reported uncertainty-based selection

## Limitations

- Filtering comes at a cost in retrieval rate — about 75% of scores are discarded at the reported threshold, so downstream analyses lose most of the score matrix
- Predicted scores are not deterministic and may change when a new model is trained (different parameters or training data)
- Hard to discriminate high Tanimoto scores (0.8-0.9) from near-complete chemical match; spectral libraries are biased toward certain compound classes, affecting generalization to experimental data

## Evidence

- [results] MC dropout mechanism: "For a given spectrum, we compute n different embeddings, each from a slightly different version of the base neural network where 20% of its nodes are silenced (dropout rate = 0.2)."
- [abstract] uncertainty-based selection outcome: "the prediction uncertainty estimate can be used to select a subset of predictions with a root mean squared error of about 0.1."
- [results] filtering procedure: "We then filtered out scores, according to increasingly stringent interquartile range (IQR) thresholds"
- [readme] modern alternative via embedding evaluator: [evidence span withheld — anchor status: external_doc]
