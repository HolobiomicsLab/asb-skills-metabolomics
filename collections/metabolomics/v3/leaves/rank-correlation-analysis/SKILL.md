---
name: rank-correlation-analysis
description: Use when two chromatographic systems (or instruments) share measured
  molecules and you need to decide whether rank/order-based models can pool training
  data better than absolute retention-time models.
when_to_use_negative:
- You need absolute retention-time prediction on a single system (use SVR-based retention
  time prediction instead); SVR is limited in its capacity to use data from other
  chromatographic systems
- The system pair shares no molecules or retention times have not been de-duplicated/cleaned
  (lowest RT kept, >5% spread removed), so concordance and correlation scores would
  be unreliable
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3318
tools:
- name: scipy
  role: Computes Pearson and Spearman rank correlations of retention times across
    system pairs
- name: numpy
  role: Array math underlying concordance fractions and correlation computation
- name: pandas
  role: Loads and preprocesses PredRet retention tables (lowest RT per molecule, >5%
    spread filter)
- name: retention_order_prediction
  role: Reference implementation providing PredRet/v2/config.json and the evaluation
    scenarios over the five systems
  repo: https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/bty590
    title: Liquid-chromatography retention order prediction for metabolite identification
  source_content_hash: sha256:ca4e95868f1005748418102cf873cbd8d8e6fb5957154b34b9d451b7e4795e0d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - numpy
  - pandas
  - retention_order_prediction
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# rank-correlation-analysis

## Summary

Quantifies how well molecular elution order versus absolute retention times are conserved across heterogeneous LC chromatographic systems, using pairwise order-concordance fractions and Pearson/Spearman correlations on shared molecules from the PredRet dataset.

## When to use

Use when two chromatographic systems (or instruments) share measured molecules and you need to decide whether rank/order-based models can pool training data better than absolute retention-time models.

## When NOT to use

- You need absolute retention-time prediction on a single system (use SVR-based retention time prediction instead); SVR is limited in its capacity to use data from other chromatographic systems
- The system pair shares no molecules or retention times have not been de-duplicated/cleaned (lowest RT kept, >5% spread removed), so concordance and correlation scores would be unreliable

## Inputs

- PredRet retention time measurements for chromatographic systems (Eawag_XBridgeC18, FEM_long, RIKEN, UFZ_Phenomenex, LIFE_old) loaded via PredRet/v2/config.json
- Per-molecule retention time tables (pandas DataFrames) keyed by molecule identifier
- Measurements restricted to those added before July 2015

## Outputs

- conservation_stats.tsv with per-system-pair order-concordance fractions and Pearson/Spearman retention-time correlations
- Shared-molecule subsets per system pair used for concordance and correlation computation

## How to apply

Load PredRet measurements for the systems Eawag_XBridgeC18, FEM_long, RIKEN, UFZ_Phenomenex and LIFE_old (via PredRet/v2/config.json) with pandas. Preprocess each system by keeping only the lowest retention time per molecule, removing molecules whose lowest and largest retention times differ by more than 5%, and dropping molecules with very small retention times that do not interact with the chromatographic system. For each system pair, restrict to molecules measured in both systems and compute the order-concordance score: the fraction of molecule pairs (i, j) whose relative elution preference t_i < t_j agrees between the systems. For the same pairs, compute absolute retention-time conservation as Pearson and Spearman correlations of retention times with numpy/scipy. Assemble both scores per system pair into conservation_stats.tsv; the expected result is that order-concordance exceeds the retention-time correlation, showing order is much better conserved between instruments.

## Related tools

- **scipy** (Computes Pearson and Spearman rank correlations of retention times across system pairs)
- **numpy** (Array math underlying concordance fractions and correlation computation)
- **pandas** (Loads and preprocesses PredRet retention tables (lowest RT per molecule, >5% spread filter))
- **retention_order_prediction** (Reference implementation providing PredRet/v2/config.json and the evaluation scenarios over the five systems) — https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git

## Examples

```
from scipy.stats import pearsonr, spearmanr; pear, pear_p = pearsonr(rt_A, rt_B); rho, rho_p = spearmanr(rt_A, rt_B)
```

## Evaluation signals

- Order-concordance fraction must exceed the Pearson/Spearman retention-time correlation for each system pair, matching the reported finding that retention order is much better conserved between instruments
- Every molecule retained per system must have its lowest retention time kept and lowest-vs-largest RT difference ≤ 5%
- Molecules with very small retention times (non-interacting with the chromatographic system) must be absent from each system's table
- Each pairwise score must be computed only on molecules measured in both systems of the pair
- conservation_stats.tsv must contain one row per system pair with both an order-concordance value and a retention-time correlation value

## Limitations

- Molecules with very small retention times are removed because they may not interact with the chromatographic system, which restricts the shared-molecule set
- Only measurements added before July 2015 were considered in the analysis
- SVR-based absolute retention time prediction was limited in its capacity to use data from other chromatographic systems, unlike RankSVM order prediction

## Evidence

- [results] Removal of non-interacting molecules: [evidence span withheld — anchor status: not_found]
