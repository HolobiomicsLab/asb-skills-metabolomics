---
name: risk-index-computation
description: Use when prioritizing chemicals that have both exposure estimates (e.g.,
  SEEM) and predicted toxicity data (e.g., ToxPi scores from ToxCast), and you need
  a combined risk-based ranking.
when_to_use_negative:
- When only one dimension (exposure alone or toxicity alone) is available for prioritization
  — the RI requires both a normalized exposure term and a ToxPi score.
- When the chemical lacks a CASRN and cannot retrieve domain data from the U.S. EPA
  Chemistry Dashboard, or lacks SEEM coverage ('the SEEM database was unable to cover
  all the compounds').
- When the goal is hazard-only prioritization — use the PrHQ approach for the 13,441-chemical
  hazard-based ranking instead.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3407
- http://edamontology.org/topic_2269
tools:
- name: ToxPI
  role: Supplies the ToxPi toxicity score used as the hazard term of the risk index
    product
- name: U.S. EPA Chemistry Dashboard
  role: Source of ToxCast HTS data used to compute the ToxPi scores entering the RI
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:0a653156261114e9be0028228fc9276ac5ba59e273b7bddc34d9e82d2fc4d097
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ToxPI
  - U.S. EPA Chemistry Dashboard
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3407
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# risk-index-computation

## Summary

Computes a risk index (RI) for each chemical as the product of normalized log-transformed SEEM exposure estimates and ToxCast-derived ToxPi toxicity scores, then ranks chemicals and compares RI ranks to probabilistic hazard quotient (PrHQ) ranks via Pearson correlation.

## When to use

Use when prioritizing chemicals that have both exposure estimates (e.g., SEEM) and predicted toxicity data (e.g., ToxPi scores from ToxCast), and you need a combined risk-based ranking.

## When NOT to use

- When only one dimension (exposure alone or toxicity alone) is available for prioritization — the RI requires both a normalized exposure term and a ToxPi score.
- When the chemical lacks a CASRN and cannot retrieve domain data from the U.S. EPA Chemistry Dashboard, or lacks SEEM coverage ('the SEEM database was unable to cover all the compounds').
- When the goal is hazard-only prioritization — use the PrHQ approach for the 13,441-chemical hazard-based ranking instead.

## Inputs

- SEEM exposure estimates for chemicals in the human exposome database (HExpMetDB resource files)
- ToxCast-derived ToxPi toxicity scores per chemical
- PrHQ rank list for the 13,441 prioritized chemicals (for rank-correlation comparison)

## Outputs

- risk_index_ranking.csv with chemical identifiers, RI values, RI ranks, PrHQ ranks where available, and the Pearson correlation statistic
- RI rank distribution (7,770 chemicals, RI range 0–0.30)

## How to apply

Load the SEEM exposure estimates and ToxCast-derived ToxPi toxicity scores for chemicals in the human exposome database (HExpMetDB) using pandas. Log-transform the SEEM exposure values and normalize (min-max rescale) them to the [0,1] range so the exposure term is bounded. Compute RI for each chemical as the product of the rescaled log-transformed normalized exposure and the ToxPi score, then rank all chemicals by RI in descending order. For validation against the article's results, 7,770 chemicals received RIs in the range 0–0.30, methyl linoleate was highest at RI = 0.30, and BPA ranked 8 of 7,770. To assess concordance with the hazard-only scheme, overlap the RI ranking with the PrHQ ranking of 13,441 chemicals, extract each overlapping chemical's rank in both schemes, and compute the Pearson correlation with scipy.stats.pearsonr (expected r = 0.1, p < 0.001). Write the final ranked table — chemical identifiers, RI values, RI ranks, PrHQ ranks where available, and the correlation statistic — to risk_index_ranking.csv.

## Related tools

- **ToxPI** (Supplies the ToxPi toxicity score used as the hazard term of the risk index product)
- **U.S. EPA Chemistry Dashboard** (Source of ToxCast HTS data used to compute the ToxPi scores entering the RI)

## Examples

```
import pandas as pd; from scipy.stats import pearsonr; ri = norm_log_expo * toxpi; r, p = pearsonr(ri_rank, prhq_rank)
```

## Evaluation signals

- Output contains exactly 7,770 chemicals with RI values within [0, 0.30].
- Methyl linoleate has the highest RI = 0.30.
- BPA is ranked 8 of 7,770 by RI.
- Pearson correlation between RI ranks and PrHQ ranks equals 0.1 with p < 0.001 (scipy.stats.pearsonr).
- Exposure term is bounded in [0,1] after log-transform and min-max rescaling.

## Limitations

- The risk score was calculated using external exposure; the IVIVE model should be considered in future studies.
- The SEEM database was unable to cover all compounds summarized in the study.
- The LD50-based approach may underestimate chronic and subchronic toxicity risk.
- RI and PrHQ ranks show only weak correlation (r = 0.1), so the two prioritizations are not interchangeable.

## Evidence

- [results] RI ranks were significantly correlated with PrHQ ranks with a small Pearson correlation coefficient (r=0.1, p<0.001); 7,770 chemicals received RIs ranging 0–0.30, with methyl linoleate highest (RI=0.30) and BPA ranked 8/7,770.: [evidence span withheld — anchor status: not_found]
- [abstract] Prioritized 13,441 chemicals based on probabilistic hazard quotient and 7,770 chemicals based on risk index: "prioritized 13,441 chemicals based on probabilistic hazard quotient and 7,770 chemicals based on risk index"
- [abstract] Prioritized chemicals by integrating exposure level (SEEM) with predicted toxicity data and ToxPi from ToxCast: "chemicals were prioritized by integrating exposure level obtained from the Systematic Empirical Evaluation of Models with toxicity data predicted by the Toxicity Estimation Software Tool"
- [discussion] Risk score based on external exposure; IVIVE model should be considered in future: "the risk score was calculated using the external exposure, and the IVIVE model should be considered in future studies"
- [discussion] SEEM database unable to cover all compounds: "the SEEM database was unable to cover all the compounds summarized in the present study"
