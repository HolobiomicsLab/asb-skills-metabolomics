---
name: urinary-metabolite-consistency-evaluation
description: Use when a urinary (or other) metabolomics dataset compiled from a systematic
  review contains compounds with more than one report plus statistical significance,
  and you need to check whether all reports agree on up- vs down-regulation (e.g.
  the Mallafré et al.
when_to_use_negative:
- Compounds have only a single report — the vote-counting division by number of reports
  and the consistency question require more than one report per compound.
- Raw data or standard deviations are available, allowing a full quantitative meta-analysis
  rather than amanida's P-value/fold-change-only approach.
- The goal is standard meta-analysis output (combined P-value, combined fold-change)
  rather than evaluating inter-study trend consistency via vote-counting and the explore
  plot.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: amanida
  role: Performs the weighted meta-analysis (weighted Fisher P-value combination,
    weighted log fold-change averaging), computes vote-counting per compound, and
    renders the mixed-type explore plot used to assess trend consistency.
  repo: github.com/mariallr/amanida
- name: R
  role: Runtime environment in which the amanida package functions (amanida_read,
    compute_amanida, explore_plot, amanida_vote) are executed.
- name: webchem
  role: Optionally used by amanida's check_names to harmonize compound identifiers
    to PubChem IDs before vote-counting, so reports of the same metabolite are merged.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btab591
    title: 'Amanida: an R package for meta-analysis of metabolomics non-integral data'
  source_content_hash: sha256:8a412bf18ba58a7577ec75a75a40144877df0975358934298fdffc74fe3b0dba
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mariallr/amanida
  tools:
  - R
  - amanida
  - webchem
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# urinary-metabolite-consistency-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluate whether urinary metabolomics compounds reported across multiple studies show consistent regulation trends, by combining amanida's vote-counting results with its mixed-type explore plot on the colorectal cancer (Mallafré et al.) sample dataset.

## When to use

Use when a urinary (or other) metabolomics dataset compiled from a systematic review contains compounds with more than one report plus statistical significance, and you need to check whether all reports agree on up- vs down-regulation (e.g. the Mallafré et al. CRC urine dataset in amanida's sample_data).

## When NOT to use

- Compounds have only a single report — the vote-counting division by number of reports and the consistency question require more than one report per compound.
- Raw data or standard deviations are available, allowing a full quantitative meta-analysis rather than amanida's P-value/fold-change-only approach.
- The goal is standard meta-analysis output (combined P-value, combined fold-change) rather than evaluating inter-study trend consistency via vote-counting and the explore plot.

## Inputs

- amanida sample_data (bundled urinary CRC volatilome/metabolomics dataset from a systematic review, Mallafré et al.) or a user CSV/XLS/TXT with columns Id, P-value, Fold-change, N total, References
- vote-counting results table (amanida_result@vote) from compute_amanida

## Outputs

- explore_plot.png — mixed-type explore plot showing reports per compound divided by trend, subset by the counts cut-off and highlighting discrepant compounds
- significant_compounds.csv — compounds with more than one report and statistical significance plus their consistency status (Hippuric acid and Phenol, both consistent)

## How to apply

Load the amanida package in R and read the bundled sample_data, the colorectal cancer urinary volatilome/metabolomics dataset from Mallafré et al. (Cancers 2021, 13(11), 2534). Run the amanida meta-analysis with compute_amanida, which combines P-values via weighted Fisher's method and fold-changes by weighted logarithmic averaging with study size (n) as the weight; the S4 object exposes quantitative results via @stat and vote-counting via @vote. Render the mixed-type explore plot with type = 'mix' and a counts cut-off (e.g. counts = 1) so that compounds with discrepancies (reports both up- and down-regulated) are visually distinguished. Derive the subset of compounds with more than one report and statistical significance from the vote-counting table, where votes are +1 for up-regulation, -1 for down-regulation and 0 for no trend, divided by the number of reports. A compound is judged consistent when every report shows the same trend; for this dataset the expected compounds are Hippuric acid and Phenol, both consistent.

## Related tools

- **amanida** (Performs the weighted meta-analysis (weighted Fisher P-value combination, weighted log fold-change averaging), computes vote-counting per compound, and renders the mixed-type explore plot used to assess trend consistency.) — github.com/mariallr/amanida
- **R** (Runtime environment in which the amanida package functions (amanida_read, compute_amanida, explore_plot, amanida_vote) are executed.)
- **webchem** (Optionally used by amanida's check_names to harmonize compound identifiers to PubChem IDs before vote-counting, so reports of the same metabolite are merged.)

## Examples

```
explore_plot(sample_data, type = "mix", counts = 1)
```

## Evaluation signals

- The vote-counting table (amanida_result@vote) contains Hippuric acid and Phenol as compounds with more than one report and statistical significance.
- The mixed explore plot (type = 'mix', counts = 1) shows all reports of Hippuric acid and Phenol with the same regulation trend — no compound flagged as discrepant (both up- and down-regulated).
- Vote-counting values are consistent with the scheme: +1 up-regulation, -1 down-regulation, 0 no trend, divided by the number of reports.
- compute_amanida returned an S4 object with both @stat (quantitative results) and @vote (vote-counting) slots populated.
- Input identifiers were curated/harmonized (check_names to PubChem ID) before the meta-analysis so duplicate reports of the same compound are merged.

## Limitations

- Metabolomics studies rarely disclose all the statistical information needed to perform a meta-analysis; amanida works only with P-value and fold-change when standard deviation is unavailable.
- Identifier curation is critical: before using Amanida, identifiers must be curated and harmonized (converted to PubChem ID with duplicates checked via webchem), otherwise reports of the same compound may not merge.
- Consistency here is a qualitative vote-counting/trend judgement (reports agreeing on direction), not a quantitative heterogeneity assessment; compounds with only one report cannot be evaluated for consistency.

## Evidence

- [readme] type = "mix": subset the data by a cut-off value indicated by the counts parameter and show compounds with discrepancies (reports up-regulated and down-regulated): [evidence span withheld — anchor status: external_doc]
- [readme] Compound vote-counting: votes are +1 for up-regulation, -1 for down-regulation and 0 if no trend. The total votes are divided by the number of reports.: [evidence span withheld — anchor status: external_doc]
- [readme] The dataset consist in a short list of compounds extracted from Comprehensive Volatilome and Metabolome Signatures of Colorectal Cancer in Urine: A Systematic Review and Meta-Analysis Mallafré et al.: [evidence span withheld — anchor status: external_doc]
