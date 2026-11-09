---
name: rank-correlation-statistics
description: Use when two chemical prioritization/ranking schemes (e.g., risk index
  vs. PrHQ) cover overlapping substances and you need to quantify whether their rank
  orders agree via a Pearson correlation coefficient.
when_to_use_negative:
- The two rankings do not share overlapping chemicals, so paired rank statistics cannot
  be computed
- You need concordance of ordinal/tied ranks rather than linear correlation — a rank-based
  coefficient (e.g., Spearman/Kendall) should be considered instead of assuming Pearson
  on ranks is adequate
- The goal is toxicity-only prioritization without an exposure term; the PrHQ or ToxPi-only
  pathway applies, not the RI-vs-PrHQ comparison
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: ToxPI
  role: Supplies the ToxPi toxicity component whose scores are combined with exposure
    to form the RI ranking compared against PrHQ ranks
- name: U.S. EPA Chemistry Dashboard
  role: Source of ToxCast-derived data used to build the ToxPi scores entering the
    RI ranking
- name: scipy.stats.pearsonr
  role: Function used to compute the Pearson correlation coefficient and p-value between
    paired RI and PrHQ rank sets
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:7e1da8f3d1b1168f1f9f4960450d94322540cc8ea63b162b9c7aa24df9bff8e8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ToxPI
  - U.S. EPA Chemistry Dashboard
  - scipy.stats.pearsonr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# rank-correlation-statistics

## Summary

Computes and interprets a Pearson correlation between two chemical prioritization schemes — here the risk index (RI = rescaled log-normalized SEEM exposure × ToxPi score) ranks and the probabilistic hazard quotient (PrHQ) ranks — to quantify agreement between risk- and hazard-driven rankings over overlapping chemicals.

## When to use

Use when two chemical prioritization/ranking schemes (e.g., risk index vs. PrHQ) cover overlapping substances and you need to quantify whether their rank orders agree via a Pearson correlation coefficient.

## When NOT to use

- The two rankings do not share overlapping chemicals, so paired rank statistics cannot be computed
- You need concordance of ordinal/tied ranks rather than linear correlation — a rank-based coefficient (e.g., Spearman/Kendall) should be considered instead of assuming Pearson on ranks is adequate
- The goal is toxicity-only prioritization without an exposure term; the PrHQ or ToxPi-only pathway applies, not the RI-vs-PrHQ comparison

## Inputs

- Risk index (RI) table: chemical identifiers, rescaled log-normalized SEEM exposure × ToxPi score, descending rank (7,770 chemicals)
- Probabilistic hazard quotient (PrHQ) ranking of 13,441 chemicals from the HExpMetDB prioritization
- Chemical identifier mapping (e.g., CASRN) to match substances across the two schemes

## Outputs

- Pearson correlation coefficient (r = 0.1) and p-value (p < 0.001) between RI and PrHQ ranks
- Ranked RI table with chemical identifiers, RI values, RI ranks, PrHQ ranks where available, and the correlation statistic (risk_index_ranking.csv)

## How to apply

First compute the primary ranking: log-transform SEEM exposure estimates, rescale to [0,1], and multiply by the ToxCast-derived ToxPi score to obtain the risk index, ranking 7,770 chemicals in descending order (methyl linoleate highest at RI = 0.30). Next, extract the subset of chemicals overlapping with the prior PrHQ ranking of 13,441 chemicals and record each chemical's rank under both schemes. Compute the Pearson correlation coefficient between the paired rank sets with scipy.stats.pearsonr; in this study RI and PrHQ ranks correlate at r = 0.1 with p < 0.001 — a significant but weak correlation, indicating the exposure-weighted risk index diverges substantially from pure hazard-based prioritization. Report r, p, the overlap size, and side-by-side ranks (e.g., BPA ranked 8/7,770 by RI) in the output table.

## Related tools

- **ToxPI** (Supplies the ToxPi toxicity component whose scores are combined with exposure to form the RI ranking compared against PrHQ ranks)
- **U.S. EPA Chemistry Dashboard** (Source of ToxCast-derived data used to build the ToxPi scores entering the RI ranking)
- **scipy.stats.pearsonr** (Function used to compute the Pearson correlation coefficient and p-value between paired RI and PrHQ rank sets)

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(ri_ranks, prhq_ranks)
```

## Evaluation signals

- Output table contains exactly 7,770 chemicals with RI values within [0, 0.30]
- Methyl linoleate has the highest RI = 0.30 and BPA is ranked 8 of 7,770
- Pearson correlation between RI and PrHQ ranks equals r = 0.1 with p < 0.001
- Every paired chemical has a defined rank in both the RI and PrHQ schemes (overlap subset only)
- Correlation computed via scipy.stats.pearsonr returns both coefficient and p-value, not just r

## Limitations

- The correlation is significant but weak (r = 0.1), so agreement between RI and PrHQ schemes should not be overstated
- RI ranks only 7,770 of the 13,441 PrHQ-prioritized chemicals because SEEM was unable to cover all compounds
- The risk score uses external exposure; IVIVE-based internal exposure would be needed to strengthen the comparison
- ToxPi/TEST-predicted LD50-based toxicity may underestimate chronic and subchronic toxicity risk, affecting rank agreement

## Evidence

- [results] RI ranks were significantly correlated with PrHQ ranks with a small Pearson correlation coefficient (r=0.1, p<0.001): [evidence span withheld — anchor status: not_found]
- [results] 7,770 chemicals received RIs ranging 0–0.30, with methyl linoleate highest (RI=0.30) and BPA ranked 8/7,770: [evidence span withheld — anchor status: not_found]
- [abstract] prioritized 13,441 chemicals based on probabilistic hazard quotient and 7,770 chemicals based on risk index: "prioritized 13,441 chemicals based on probabilistic hazard quotient and 7,770 chemicals based on risk index"
- [discussion] we further calculated the ToxPi score in order to more comprehensively estimate the toxicity ranking of chemicals: "we further calculated the ToxPi score in order to more comprehensively estimate the toxicity ranking of chemicals"
