---
name: chemical-prioritization-overlap-analysis
description: Use when two chemical prioritization rankings (e.g., RI vs PrHQ) cover
  overlapping chemical sets and you need to quantify agreement between their orderings
  via rank correlation and overlap extraction.
when_to_use_negative:
- Chemicals lacking a CASRN — the HTS method is based on the U.S. EPA Chemistry Dashboard,
  so only chemicals with CASRN can retrieve the relevant domain data
- Chemicals absent from the SEEM database, which was unable to cover all compounds
  in the study
- Cases requiring internal-dose-based prioritization — the risk score was calculated
  using external exposure, and the IVIVE model was deferred to future studies
tools:
- name: ToxPI
  role: Provides the toxicity-side ToxPi score (calculated from ToxCast data) that
    is multiplied by the normalized exposure term to form the Risk Index being ranked
- name: U.S. EPA Chemistry Dashboard
  role: Source of the HTS-retrieved chemical domain data underlying the ranking; only
    chemicals with CASRN could be retrieved
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:9a38a2d1d125a979c3a4183cba52b22c7eb36f8db4f68a4282fcfab2c3b33c7d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ToxPI
  - U.S. EPA Chemistry Dashboard
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# chemical-prioritization-overlap-analysis

## Summary

Compares two risk-based chemical prioritization schemes — a Risk Index (RI = rescaled log-normalized SEEM exposure × ToxPi score from ToxCast) and a probabilistic hazard quotient (PrHQ) — by extracting each overlapping chemical's rank in both rankings and computing the Pearson correlation between the rank sets (r = 0.1, p < 0.001 for 7,770 RI-ranked chemicals).

## When to use

Use when two chemical prioritization rankings (e.g., RI vs PrHQ) cover overlapping chemical sets and you need to quantify agreement between their orderings via rank correlation and overlap extraction.

## When NOT to use

- Chemicals lacking a CASRN — the HTS method is based on the U.S. EPA Chemistry Dashboard, so only chemicals with CASRN can retrieve the relevant domain data
- Chemicals absent from the SEEM database, which was unable to cover all compounds in the study
- Cases requiring internal-dose-based prioritization — the risk score was calculated using external exposure, and the IVIVE model was deferred to future studies

## Inputs

- SEEM (Systematic Empirical Evaluation of Models) exposure estimates for HExpMetDB chemicals
- ToxCast-derived ToxPi toxicity scores per chemical
- Prior PrHQ (probabilistic hazard quotient) ranking covering 13,441 chemicals
- Chemical identifiers (CASRN) per chemical

## Outputs

- risk_index_ranking.csv with chemical identifiers, RI values, RI ranks, PrHQ ranks where available, and the Pearson correlation statistic
- RI ranking of 7,770 chemicals with values in [0, 0.30]
- Pearson rank-correlation coefficient between RI ranks and PrHQ ranks (r = 0.1, p < 0.001)

## How to apply

Load the SEEM exposure estimates and ToxCast-derived ToxPi toxicity scores for chemicals in the human exposome database (HExpMetDB) using pandas. Log-transform the SEEM exposure values and normalize/rescale them to the [0,1] range so the exposure term is bounded. Compute the Risk Index for each chemical as the product of the rescaled log-transformed normalized SEEM exposure and the ToxPi score, then rank all chemicals by RI in descending order (expected: 7,770 chemicals, RI range 0–0.30, methyl linoleate highest at RI = 0.30, BPA ranked 8/7,770). Identify chemicals overlapping with the prior PrHQ ranking of 13,441 chemicals, extract each chemical's rank under both schemes, and calculate the Pearson correlation coefficient with scipy.stats.pearsonr, verifying r = 0.1 with p < 0.001. Write the ranked RI table (chemical identifiers, RI values, RI ranks, PrHQ ranks where available, correlation statistic) to risk_index_ranking.csv.

## Related tools

- **ToxPI** (Provides the toxicity-side ToxPi score (calculated from ToxCast data) that is multiplied by the normalized exposure term to form the Risk Index being ranked)
- **U.S. EPA Chemistry Dashboard** (Source of the HTS-retrieved chemical domain data underlying the ranking; only chemicals with CASRN could be retrieved)

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(df['RI_rank'], df['PrHQ_rank'])
```

## Evaluation signals

- Output risk_index_ranking.csv contains exactly 7,770 chemicals with RI values
- All RI values fall within the [0, 0.30] range
- Methyl linoleate is the top-ranked chemical with RI = 0.30
- BPA has RI rank 8 of 7,770
- Pearson correlation between RI ranks and PrHQ ranks equals 0.1 with p < 0.001 via scipy.stats.pearsonr

## Limitations

- RI ranks correlate only weakly with PrHQ ranks (r = 0.1), so the two prioritization schemes order chemicals quite differently despite significance
- The risk score is based on external exposure; internal-dose conversion via IVIVE was not incorporated
- The LD50-based toxicity estimate may underestimate chronic and subchronic toxicity risk
- Toxicity due to chemical interactions (e.g., synergistic effects) in mixtures cannot be excluded

## Evidence

- [results] RI ranks were significantly correlated with PrHQ ranks with a small Pearson correlation coefficient (r=0.1, p<0.001); 7,770 chemicals received RIs ranging 0–0.30, with methyl linoleate highest (RI=0.30) and BPA ranked 8/7,770.: [evidence span withheld — anchor status: not_found]
- [abstract] prioritized 13,441 chemicals based on probabilistic hazard quotient and 7,770 chemicals based on risk index: "prioritized 13,441 chemicals based on probabilistic hazard quotient and 7,770 chemicals based on risk index"
- [discussion] we further calculated the ToxPi score in order to more comprehensively estimate the toxicity ranking of chemicals: "we further calculated the ToxPi score in order to more comprehensively estimate the toxicity ranking of chemicals"
