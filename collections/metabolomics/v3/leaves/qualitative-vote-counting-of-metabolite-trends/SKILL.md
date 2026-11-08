---
name: qualitative-vote-counting-of-metabolite-trends
description: Use when the input is a set of per-study aggregate statistics (p-values
  and fold-changes) for metabolites and the goal is to identify which compounds recur
  consistently across studies with a consistent up/down trend direction.
when_to_use_negative:
- When individual-level participant data are available and a quantitative meta-analysis
  of effect sizes can be performed instead of vote-counting
- When compound names have not been harmonized — synonyms across studies will split
  votes and undercount recurrent metabolites
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: amanida
  role: Implements the qualitative vote-counting (+1 up / -1 down) and aggregation
    of votes across studies
  repo: https://github.com/mariallr/easy-amanida
- name: Easy-Amanida
  role: Shiny web app wrapping amanida that provides the workflow to optimize naming
    combination and harmonize compound names before voting
  repo: https://github.com/mariallr/easy-amanida
- name: webchem
  role: Supports compound naming harmonization within the Easy-Amanida workflow
- name: Shiny
  role: R add-on for interactive web apps in which Easy-Amanida is implemented
- name: R
  role: Environment in which amanida vote-counting is executed and vote_counting_results.csv
    is written
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/jrsm.1713
    title: 'Easy‐Amanida: An R Shiny application for the meta‐analysis of aggregate
      results in clinical metabolomics using Amanida and Webchem'
  source_content_hash: sha256:9cbeacbb0de14cbea62915221e5e02add401c8668cf593c05923de082e03779f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mariallr/easy-amanida
  repo_url: https://github.com/mariallr/easy-amanida
  tools:
  - Easy-Amanida
  - R
  - Shiny
  - amanida
  - webchem
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# qualitative-vote-counting-of-metabolite-trends

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies the amanida qualitative vote-counting method to aggregate metabolomics statistics (p-values and fold-changes per study), assigning +1 to up-regulated and -1 to down-regulated compounds per study, then aggregating votes to find consistently recurrent metabolites — e.g. glutamine (7 studies) and histidine (5 studies) down-regulated in PDAC.

## When to use

Use when the input is a set of per-study aggregate statistics (p-values and fold-changes) for metabolites and the goal is to identify which compounds recur consistently across studies with a consistent up/down trend direction.

## When NOT to use

- When individual-level participant data are available and a quantitative meta-analysis of effect sizes can be performed instead of vote-counting
- When compound names have not been harmonized — synonyms across studies will split votes and undercount recurrent metabolites

## Inputs

- Per-study aggregate statistics table (p-values and fold-changes per compound), e.g. loaded from the easy-amanida GitHub repository
- Metabolite/compound identifier list requiring naming harmonization (synonyms across studies)

## Outputs

- vote_counting_results.csv — summarized vote-counting table with recurrence counts and trend directions per compound
- Tally of consistent significant compounds split by direction (8 up-regulated / 12 down-regulated in the PDAC example)
- List of metabolites flagged as inconsistent-trend and excluded (15 in the PDAC example)

## How to apply

Load the dataset of aggregate statistics (p-values and fold-changes per study) from the easy-amanida GitHub repository into R and apply the amanida qualitative vote-counting method, assigning +1 to up-regulated and -1 to down-regulated compounds in each study. Aggregate votes across studies to compute the number of studies in which each metabolite is recurrently identified with a consistent trend. Flag metabolites with inconsistent regulation trends across studies (15 in the PDAC dataset) and exclude them from the significant-compound tally. Summarize consistent hits — glutamine (7 studies, down-regulated), histidine (5 studies, down-regulated), and the three recurrent up-regulated bile acids (tauroursodeoxycholic, taurocholic, glycocholic acid; 3 studies each) — yielding 8 up-regulated and 12 down-regulated significant compounds. Write the summarized vote-counting table to vote_counting_results.csv. The amanida package handles compound naming harmonization when combined via the easy-amanida workflow.

## Related tools

- **amanida** (Implements the qualitative vote-counting (+1 up / -1 down) and aggregation of votes across studies) — https://github.com/mariallr/easy-amanida
- **Easy-Amanida** (Shiny web app wrapping amanida that provides the workflow to optimize naming combination and harmonize compound names before voting) — https://github.com/mariallr/easy-amanida
- **webchem** (Supports compound naming harmonization within the Easy-Amanida workflow)
- **Shiny** (R add-on for interactive web apps in which Easy-Amanida is implemented)
- **R** (Environment in which amanida vote-counting is executed and vote_counting_results.csv is written)

## Examples

```
library(amanida); votes <- votevot(pdac_aggregate_stats); write.csv(votes, "vote_counting_results.csv")
```

## Evaluation signals

- Output reproduces glutamine identified in 7 studies, consistently down-regulated
- Output reproduces histidine identified in 5 studies, consistently down-regulated
- Each of the three bile acids (tauroursodeoxycholic, taurocholic, glycocholic acid) appears recurrently in 3 studies, up-regulated
- After removal of the 15 inconsistent-trend metabolites, the tally is exactly 8 up-regulated and 12 down-regulated significant compounds
- vote_counting_results.csv is produced and its per-compound vote sums equal the number of studies with consistent trend direction

## Limitations

- Metabolites with inconsistent regulation trends across studies (15 in the PDAC dataset) must be flagged and excluded from the significant-compound tally
- Metabolomics meta-analysis is hindered by the wide variety of compounds, differing experimental methods, lack of consensus on statistical reporting, and compound naming synonyms
- Only aggregate statistics (p-value and fold-change) can be used; individual-level data are not required but effect-size granularity is lost

## Evidence
