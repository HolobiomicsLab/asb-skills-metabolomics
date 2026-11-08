---
name: exposure-toxicity-integration-ranking
description: Use when you hold paired exposure estimates (e.g., SEEM) and in vitro
  toxicity scores (ToxCast/ToxPi) for a chemical set and need a single risk-based
  ranking that integrates both dimensions.
when_to_use_negative:
- Chemicals lack ToxCast/ToxPi coverage or SEEM exposure estimates — the SEEM database
  was unable to cover all compounds in the study, so RI cannot be computed for uncovered
  chemicals
- You need internal-dose or IVIVE-based risk scoring — the article explicitly notes
  the risk score was calculated using external exposure and the IVIVE model should
  be considered in future studies
- HTS data retrieval is needed for chemicals without CASRN — the U.S. EPA Chemistry
  Dashboard-based method only succeeded for chemicals with CASRN
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: ToxPI
  role: Supplies the toxicity ranking component (ToxPi score from ToxCast data) that
    is multiplied by normalized exposure to form the risk index
- name: U.S. EPA Chemistry Dashboard
  role: Source of the high-throughput screening data retrieval; only chemicals with
    CASRN could retrieve domain data, constraining which chemicals receive an RI
- name: TEST
  role: Predicted oral LD50 values for rats feed the toxicity side of the prioritization
    that underlies the ToxPi score
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:2f41902502a39cfb07f9024a4220a8a44b9ab39d2e453299346bb73c16f1a5c2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - TEST
  - ToxPI
  - U.S. EPA Chemistry Dashboard
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# exposure-toxicity-integration-ranking

## Summary

Computes a risk index (RI) for chemicals in the HExpMetDB human exposome database by multiplying rescaled, log-transformed SEEM exposure estimates by ToxCast-derived ToxPi toxicity scores, then ranks chemicals by RI and cross-validates the ranking against probabilistic hazard quotient (PrHQ) ranks.

## When to use

Use when you hold paired exposure estimates (e.g., SEEM) and in vitro toxicity scores (ToxCast/ToxPi) for a chemical set and need a single risk-based ranking that integrates both dimensions.

## When NOT to use

- Chemicals lack ToxCast/ToxPi coverage or SEEM exposure estimates — the SEEM database was unable to cover all compounds in the study, so RI cannot be computed for uncovered chemicals
- You need internal-dose or IVIVE-based risk scoring — the article explicitly notes the risk score was calculated using external exposure and the IVIVE model should be considered in future studies
- HTS data retrieval is needed for chemicals without CASRN — the U.S. EPA Chemistry Dashboard-based method only succeeded for chemicals with CASRN

## Inputs

- SEEM (Systematic Empirical Evaluation of Models) exposure estimates for human exposome chemicals from HExpMetDB resource files
- ToxPi toxicity scores computed from the ToxCast database
- Probabilistic hazard quotient (PrHQ) ranks for the 13,441-chemical comparison set

## Outputs

- risk_index_ranking.csv with chemical identifiers, RI values, RI ranks, and PrHQ ranks where available
- Pearson rank-correlation statistic between RI and PrHQ ranks (r = 0.1, p < 0.001)

## How to apply

Load SEEM exposure estimates and ToxCast-derived ToxPi scores for chemicals from the HExpMetDB resource files with pandas; log-transform SEEM values and rescale to [0,1] so the exposure term is bounded. Compute RI = rescaled log-transformed normalized SEEM exposure × ToxPi score for each chemical, then sort in descending order and assign RI ranks. As a validation, join with the prior PrHQ ranking of 13,441 chemicals, extract each overlapping chemical's rank in both schemes, and compute the Pearson correlation with scipy.stats.pearsonr (expected r = 0.1, p < 0.001). Write the ranked table (identifiers, RI values, RI ranks, PrHQ ranks, correlation statistic) to risk_index_ranking.csv. Note the article computed the risk score from external exposure only, so treat RI as an external-exposure-based prioritization metric.

## Related tools

- **ToxPI** (Supplies the toxicity ranking component (ToxPi score from ToxCast data) that is multiplied by normalized exposure to form the risk index)
- **U.S. EPA Chemistry Dashboard** (Source of the high-throughput screening data retrieval; only chemicals with CASRN could retrieve domain data, constraining which chemicals receive an RI)
- **TEST** (Predicted oral LD50 values for rats feed the toxicity side of the prioritization that underlies the ToxPi score)

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(ri_ranks, prhq_ranks)  # expect r=0.1, p<0.001
```

## Evaluation signals

- Output table contains exactly 7,770 chemicals with RI values within [0, 0.30]
- Methyl linoleate is the top-ranked chemical with RI = 0.30
- BPA is ranked 8 of 7,770 in the RI ordering
- Pearson correlation between RI ranks and PrHQ ranks equals 0.1 with p < 0.001 (scipy.stats.pearsonr)
- All exposure terms are log-transformed and rescaled to [0,1] before multiplication with the ToxPi score

## Limitations

- The risk score is based on external exposure; an IVIVE model should be considered in future studies
- SEEM was unable to cover all compounds, so not every HExpMetDB chemical receives an RI
- Only chemicals with CASRN could retrieve domain data from the U.S. EPA Chemistry Dashboard
- RI ranking is correlated with PrHQ ranks only weakly (r = 0.1), so the two prioritizations should not be treated as interchangeable

## Evidence

- [abstract] Prioritized 13,441 chemicals based on probabilistic hazard quotient and 7,770 chemicals based on risk index: "prioritized 13,441 chemicals based on probabilistic hazard quotient and 7,770 chemicals based on risk index"
- [discussion] ToxPi score rationale: "we further calculated the ToxPi score in order to more comprehensively estimate the toxicity ranking of chemicals"
- [discussion] External exposure basis of risk score: "the risk score was calculated using the external exposure, and the IVIVE model should be considered in future studies"
- [abstract] Integration of exposure and toxicity: "chemicals were prioritized by integrating exposure level obtained from the Systematic Empirical Evaluation of Models with toxicity data predicted by the Toxicity Estimation Software Tool"
