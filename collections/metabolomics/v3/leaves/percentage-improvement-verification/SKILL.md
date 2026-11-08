---
name: percentage-improvement-verification
description: Use when an article claims a relative percentage improvement of one model
  over baselines (e.g. ESP vs MLP/GNN average rank) and the claim must be independently
  verified from the reported metric values.
when_to_use_negative:
- When the claimed improvement is stated on a different evaluation setting than the
  metric values available (e.g. the README's NPLIB1 full-candidate demo rather than
  the NIST-20/PubChem candidate evaluation) — the recomputation will not reproduce
  the claim
- When the article reports only Rank@K values without average-rank numbers, since
  the claim is about average rank and top-k rates (MLP is noted as a strong contender
  at top five ranks) cannot be converted into this percentage
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: numpy
  role: Computes the relative improvement percentages (rank_baseline - rank_ESP) /
    rank_baseline * 100 and the comparison against claimed values
- name: scikit-learn
  role: Listed in the article's environment (python=3.8, scikit-learn) supporting
    the numeric metric computations for the verification workflow
- name: Ensemble Spectral Prediction (ESP)
  role: Source of the average-rank metric values being verified; models and evaluation
    scripts (train.py, ens_train_canopus.py) from HassounLab/ESP
  repo: https://github.com/HassounLab/ESP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae490
    title: An Ensemble Spectral Prediction (ESP) model for metabolite annotation
  source_content_hash: sha256:ff05d01069b8dde233ac86e5abb5e3e937bc0b127bad17d4b64353298a8f16a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/ESP
  tools:
  - Ensemble Spectral Prediction (ESP)
  - Python
  - numpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# percentage-improvement-verification

## Summary

Recompute a paper's claimed relative-improvement percentages from its underlying rank metrics — here, ESP's average-rank gains of 23.7% (over MLP) and 37.2% (over GNN) on NIST-20 training with PubChem candidate sets — and check the recomputed values against the published claims.

## When to use

Use when an article claims a relative percentage improvement of one model over baselines (e.g. ESP vs MLP/GNN average rank) and the claim must be independently verified from the reported metric values.

## When NOT to use

- When the claimed improvement is stated on a different evaluation setting than the metric values available (e.g. the README's NPLIB1 full-candidate demo rather than the NIST-20/PubChem candidate evaluation) — the recomputation will not reproduce the claim
- When the article reports only Rank@K values without average-rank numbers, since the claim is about average rank and top-k rates (MLP is noted as a strong contender at top five ranks) cannot be converted into this percentage

## Inputs

- Reported average-rank metric values for ESP, MLP, and GNN under the claimed evaluation condition (NIST-20 training, PubChem candidate sets)
- Claimed improvement percentages from the article (23.7% and 37.2%)

## Outputs

- Computed relative improvement of ESP over the MLP baseline as a percentage
- Computed relative improvement of ESP over the GNN baseline as a percentage
- Verification verdict stating whether each computed value matches the claim within rounding tolerance

## How to apply

Extract the reported average-rank metric values for the ESP, MLP, and GNN models under the exact evaluation condition stated in the claim (NIST-20-trained models evaluated on relevant PubChem candidate sets). Using numpy, compute the relative improvement of ESP over each baseline as (rank_baseline - rank_ESP) / rank_baseline * 100. Compare each computed percentage against the abstract's claimed values of 23.7% (MLP baseline) and 37.2% (GNN baseline), accepting agreement within rounding tolerance. Do not substitute metric values from a different evaluation setting — the README demo on NPLIB1 reports different ranks (e.g. 'Average rank 279.557' for ESP) and is not the condition underlying the abstract's percentages.

## Related tools

- **numpy** (Computes the relative improvement percentages (rank_baseline - rank_ESP) / rank_baseline * 100 and the comparison against claimed values)
- **scikit-learn** (Listed in the article's environment (python=3.8, scikit-learn) supporting the numeric metric computations for the verification workflow)
- **Ensemble Spectral Prediction (ESP)** (Source of the average-rank metric values being verified; models and evaluation scripts (train.py, ens_train_canopus.py) from HassounLab/ESP) — https://github.com/HassounLab/ESP

## Examples

```
python -c "import numpy as np; rank_mlp, rank_gnn, rank_esp = 339.350, 241.753, 279.557; print(round((rank_mlp-rank_esp)/rank_mlp*100,1), round((rank_gnn-rank_esp)/rank_gnn*100,1))"
```

## Evaluation signals

- Recomputed improvement over the MLP baseline equals 23.7% (within rounding tolerance)
- Recomputed improvement over the GNN baseline equals 37.2% (within rounding tolerance)
- The metric values used are average ranks from the NIST-20-trained / PubChem candidate-set condition, not the README NPLIB1 demo values (ESP 279.557, GNN 241.753, MLP 339.350)
- The two percentages are computed with the same formula form (rank_baseline - rank_ESP) / rank_baseline * 100 so the comparison is symmetric
- The verification explicitly reports a match/no-match verdict per baseline

## Limitations

- NIST-20 training data cannot be published due to NIST license regulations, so the exact metric values underlying the abstract's percentages may not be directly reproducible from released artifacts
- Published pretrained models and datasets are for NPLIB1 only; demo outputs there show different average ranks (ESP 279.557, MLP 339.350, GNN 241.753) and do not reproduce the abstract's claimed improvements
- The abstract claims percentages that are relative to the baselines under one specific candidate-set condition; annotation performance also depends on candidate-set size and candidate similarity, so gains do not transfer to other conditions

## Evidence

- [abstract] ESP improves average rank by 23.7% and 37.2% over the MLP and GNN baselines, respectively, demonstrating performance gain over state-of-the-art neural network approaches: "ESP improves average rank by 23.7% and 37.2% over the MLP and GNN baselines, respectively, demonstrating performance gain over state-of-the-art neural network approaches"
- [readme] We have shown improvements with ESP over the MLP model ... in terms of a 23.7% increase in average rank performance on the full NIST candidate set.: "We have shown improvements with ESP over the MLP model (implementation of NEIMS model (Wei et al., 2019)"
- [readme] We also show 37.2% improvement in average rank over the baseline GNN model, initially presented by our group (Hao et al., 2020): "We also show 37.2% improvement in average rank over the baseline GNN model, initially presented by our group (Hao et al., 2020)"
