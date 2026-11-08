---
name: qualitative-vote-counting-meta-analysis
description: Use when input studies disclose only compound identifiers and trend labels
  (up-/down-regulated, no change) with no p-values, fold-changes or standard deviations,
  and a per-compound qualitative summary across studies is needed.
when_to_use_negative:
- Studies disclose full statistical parameters (p-value, fold-change, n) — use amanida's
  quantitative weighted meta-analysis (mode = "quan" with compute_amanida) instead,
  which combines p-values via weighted Fisher's method and fold-changes via study-size-weighted
  averaging
- 'Input data has not been curated: identifiers are inconsistent chemical names/InChI/InChIKey/SMILES
  and check_names harmonization has not been run, so votes for the same compound would
  be split across duplicate IDs'
- The goal is a weighted or significance-based effect-size estimate rather than a
  direction-of-change tally — vote-counting ignores study size and magnitudes entirely
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: amanida
  role: Provides amanida_read (mode = "qual"), amanida_vote, vote_plot and explore_plot
    for the qualitative vote-counting meta-analysis
  repo: https://github.com/mariallr/amanida
- name: R
  role: Execution environment for the amanida package functions
- name: webchem
  role: Used via check_names to search PubChem and harmonize compound identifiers
    (name, InChI, InChIKey, SMILES) to a common nomenclature before vote-counting
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btab591
    title: 'Amanida: an R package for meta-analysis of metabolomics non-integral data'
  source_content_hash: sha256:ba82e164d5424340a4fc2e419c5170c6afb482d6e47f0a61e15ad5ade5aa0498
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mariallr/amanida
  repo_url: https://github.com/mariallr/amanida
  tools:
  - R
  - amanida
  - webchem
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# qualitative-vote-counting-meta-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Performs a qualitative meta-analysis of metabolomics compounds via vote-counting when studies report only the trend (up-/down-regulation) of a compound rather than p-values or fold-changes; each compound-study observation contributes +1, -1 or 0 and votes are summed per compound.

## When to use

Use when input studies disclose only compound identifiers and trend labels (up-/down-regulated, no change) with no p-values, fold-changes or standard deviations, and a per-compound qualitative summary across studies is needed.

## When NOT to use

- Studies disclose full statistical parameters (p-value, fold-change, n) — use amanida's quantitative weighted meta-analysis (mode = "quan" with compute_amanida) instead, which combines p-values via weighted Fisher's method and fold-changes via study-size-weighted averaging
- Input data has not been curated: identifiers are inconsistent chemical names/InChI/InChIKey/SMILES and check_names harmonization has not been run, so votes for the same compound would be split across duplicate IDs
- The goal is a weighted or significance-based effect-size estimate rather than a direction-of-change tally — vote-counting ignores study size and magnitudes entirely

## Inputs

- CSV, xls/xlsx or txt file (e.g. amanida's bundled sample_data from the colorectal cancer urine volatilome systematic review) with columns: compound identifier, trend/behaviour label (up-regulated, down-regulated, or no change), and bibliographic reference
- coln character vector specifying those three columns in order: c("Compound Name", "Behaviour", "References")

## Outputs

- S4 object from amanida_vote with a vote-counting table accessed via vote_result@vote, listing compound identifier, summed votes and number of contributing articles
- Optional vote_plot / explore_plot visualizations (bar plots of votes and reports divided by trend, including compounds with discrepant up/down reports)

## How to apply

Load the data with amanida_read using mode = "qual" and coln = c("Compound Name", "Behaviour", "References"), where Behaviour holds trend labels (up-regulated or down-regulated). Optionally run check_names first to harmonize identifiers to PubChem IDs via webchem, following the Metabolites Merging Strategy (MMS), so duplicate compounds from different naming conventions merge before voting. Then call amanida_vote on the resulting data object; the function scores each compound-study observation +1 for up-regulation, -1 for down-regulation and 0 for no trend, sums votes per compound, and records the number of contributing articles. Results are returned as an S4 object whose vote-counting table is accessed via vote_result@vote, containing compound identifier, total votes and number of reports; optionally visualize with vote_plot or explore_plot (e.g. type = "mix", counts = 1 to reveal compounds with discrepant trends).

## Related tools

- **amanida** (Provides amanida_read (mode = "qual"), amanida_vote, vote_plot and explore_plot for the qualitative vote-counting meta-analysis) — https://github.com/mariallr/amanida
- **R** (Execution environment for the amanida package functions)
- **webchem** (Used via check_names to search PubChem and harmonize compound identifiers (name, InChI, InChIKey, SMILES) to a common nomenclature before vote-counting)

## Examples

```
vote_result <- amanida_vote(data_votes)
```

## Evaluation signals

- The vote table has exactly the columns expected for qualitative mode: compound identifier, total vote and number of reports/articles
- Every per-observation vote is one of +1 (up-regulated), -1 (down-regulated) or 0 (no trend); no other values appear in the tally
- The number of articles per compound equals the count of distinct contributing references after identifier harmonization; duplicated compound names (different spellings of the same metabolite) have been merged into a single row when check_names/comp.inf was used
- vote_result@vote is populated on the returned S4 object and sums of votes match a manual tally of trend labels in the input file
- vote_plot/explore_plot render without error and explore_plot with type = "mix" flags compounds having both up- and down-regulated reports (discrepancies)

## Limitations

- Vote-counting is purely qualitative: it discards study size and effect magnitude, so a single large study and a small study contribute equally
- Requires the curation/harmonization of identifiers beforehand — as the article states, 'Before using `Amanida` is important the curation of the identifiers'; otherwise duplicates split votes
- Input studies must disclose at least a trend label; metabolomics studies 'rarely disclose all the statistical information needed to perform a meta-analysis', which is precisely the situation this qualitative mode addresses

## Evidence

- [readme] Vote-counting scoring rule: [evidence span withheld — anchor status: external_doc]
