---
name: hlb-distribution-statistics
description: Use when a table of IFS-predicted biotransformation half-lives exists
  for an exposome chemical inventory and you need population-level distribution statistics
  (count predicted, median HLB, fraction below 12 h) and spot-check validation.
when_to_use_negative:
- The chemical list still contains compounds with a metal atom or molecular weight
  > 1,000 — these must be filtered out before IFS-based HLB statistics are computed
- The input is the predicted biotransformation metabolite database (>95,000 metabolites
  from BioTransformer) rather than parent-chemical HLB values
- Half-lives come from a different method than Iterative Fragment Selection (e.g.,
  measured HLBs or BioTransformer-predicted metabolite rates), so the reported IFS
  median and <12 h fraction do not apply
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: BioTransformer
  role: Listed in the source task card's tool set for this HLB task; note the article
    applies BioTransformer to predict biotransformation products, while HLB prediction
    itself uses the Iterative Fragment Selection approach
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:63e51dcb197d64ce61292de2300580a10b14944abe57bac22bfabdd2f64782ba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BioTransformer
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# hlb-distribution-statistics

## Summary

Summarizes the distribution of predicted human biotransformation half-lives (HLBs) across a compiled chemical list, reporting how many chemicals were successfully predicted, the median HLB (e.g., 4.96 h), and the fraction with HLB < 12 h, plus spot-check verification against known chemicals.

## When to use

Use when a table of IFS-predicted biotransformation half-lives exists for an exposome chemical inventory and you need population-level distribution statistics (count predicted, median HLB, fraction below 12 h) and spot-check validation.

## When NOT to use

- The chemical list still contains compounds with a metal atom or molecular weight > 1,000 — these must be filtered out before IFS-based HLB statistics are computed
- The input is the predicted biotransformation metabolite database (>95,000 metabolites from BioTransformer) rather than parent-chemical HLB values
- Half-lives come from a different method than Iterative Fragment Selection (e.g., measured HLBs or BioTransformer-predicted metabolite rates), so the reported IFS median and <12 h fraction do not apply

## Inputs

- Compiled chemical list of the human exposome database with identifiers (e.g., CASRN) and successfully predicted IFS HLB values in hours
- Log Kow and log Koa values from UFZ-LSER database v3.2.1 used as IFS descriptors

## Outputs

- Count of chemicals with successfully predicted HLBs (reference: 19,406)
- Median predicted biotransformation half-life (reference: 4.96 h)
- Fraction and count of chemicals with HLB < 12 h (reference: 13,733/19,406, ~70%)
- Spot-check table of half-life values for representative chemicals (DEHP, BPA, PFOA, PCB-105)

## How to apply

Start from the assembled table of chemical identifiers and IFS-predicted HLB values produced by applying the Iterative Fragment Selection QSAR (Arnot et al. 2014) to chemicals with retrieved log Kow and log Koa descriptors from the UFZ-LSER database v3.2.1. Compute (1) the count of chemicals with successfully predicted HLBs, (2) the median HLB, and (3) the fraction of substances with half-lives below 12 h, which in the reference application was ~70% (13,733/19,406). Perform spot-check verification on representative chemicals to confirm the prediction table is correctly assembled: DEHP 4.5 h, BPA 7.1 h, PFOA 7,786 h, and PCB-105 21,809 h. Chemicals with a metal atom or molecular weight > 1,000 must have been filtered out beforehand, since the IFS approach cannot handle them; their absence defines the maximum achievable prediction count. Validation is complete when the prediction count, median, and <12 h fraction all reproduce, and spot-check values match exactly.

## Related tools

- **BioTransformer** (Listed in the source task card's tool set for this HLB task; note the article applies BioTransformer to predict biotransformation products, while HLB prediction itself uses the Iterative Fragment Selection approach)

## Examples

```
import pandas as pd; df = pd.read_csv('hexpmetdb_hlb.csv'); print(len(df), df['hlb_h'].median(), (df['hlb_h'] < 12).mean())
```

## Evaluation signals

- Predicted-chemical count equals 19,406 after applying the metal-atom / MW > 1,000 filter
- Median predicted HLB equals 4.96 h
- Count of chemicals with HLB < 12 h equals 13,733 of 19,406 (~70%)
- Spot-check chemicals match reported half-lives exactly: DEHP 4.5 h, BPA 7.1 h, PFOA 7,786 h, PCB-105 21,809 h
- Every row in the HLB table carries a chemical identifier and a half-life value, and no filtered (metal or MW > 1,000) chemical appears

## Limitations

- The IFS approach cannot predict HLBs for chemicals with a metal atom or molecular weight > 1,000, capping the number of predictable chemicals
- Prediction failures and filtering reduce coverage of the >20,000-chemical human exposome database, so distribution statistics describe only the successfully predicted subset

## Evidence

- [abstract] The biotransformation half-lives (HLBs) of all the chemicals were assessed using the Iterative Fragment Selection approach: "The biotransformation half-lives (HLBs) of all the chemicals were assessed using the Iterative Fragment Selection approach"
- [discussion] Filter criterion for IFS HLB prediction: "we cannot predict the HLBs of chemicals with a metal atom or molecular weight of >1,000 using the IFS approach"
- [results] HLB count and median finding: [evidence span withheld — anchor status: not_found]
- [results] Fraction below 12 h finding: [evidence span withheld — anchor status: not_found]
- [methods] Spot-check chemicals: [evidence span withheld — anchor status: not_found]
- [methods] Descriptor retrieval and IFS application workflow: [evidence span withheld — anchor status: not_found]
