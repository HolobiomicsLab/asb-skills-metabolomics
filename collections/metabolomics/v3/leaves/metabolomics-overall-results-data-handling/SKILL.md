---
name: metabolomics-overall-results-data-handling
description: Use when input is a compound-level table from metabolomics studies where
  only the compound identifier, trend/behaviour label (up-regulated, down-regulated,
  or no trend) and bibliographic reference are disclosed, and a per-compound qualitative
  meta-analysis vote count is needed.
when_to_use_negative:
- The dataset contains full statistical parameters (p-value, fold-change, N) — use
  amanida's quantitative mode (mode = "quan" with compute_amanida, Fisher's-method
  p-value combination and weighted fold-change averaging) instead.
- Studies report effect sizes with standard deviations, for which a standard weighted
  quantitative meta-analysis is appropriate rather than trend-only vote-counting.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: amanida
  role: Provides amanida_read (mode = "qual") and amanida_vote for qualitative vote-counting,
    plus vote_plot/explore_plot visualization
  repo: https://github.com/mariallr/amanida
- name: R
  role: Runtime environment in which the amanida package functions are executed
- name: webchem
  role: Used by check_names to search compound names/InChI/InChIKey/SMILES in PubChem
    and harmonize identifiers before vote-counting
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btab591
    title: 'Amanida: an R package for meta-analysis of metabolomics non-integral data'
  source_content_hash: sha256:f9d7ff0d86b9b2ec2fb0a3af8cae137c7c1c07e80fa64b4bcca3248a3324331d
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# metabolomics-overall-results-data-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Performs a qualitative vote-counting meta-analysis for metabolomics compounds when studies report only trend labels (up/down/no change) instead of full statistics, using the amanida_vote function from the amanida R package on identifier-and-trend-label data such as the bundled colorectal cancer urine volatilome sample_data.

## When to use

Use when input is a compound-level table from metabolomics studies where only the compound identifier, trend/behaviour label (up-regulated, down-regulated, or no trend) and bibliographic reference are disclosed, and a per-compound qualitative meta-analysis vote count is needed.

## When NOT to use

- The dataset contains full statistical parameters (p-value, fold-change, N) — use amanida's quantitative mode (mode = "quan" with compute_amanida, Fisher's-method p-value combination and weighted fold-change averaging) instead.
- Studies report effect sizes with standard deviations, for which a standard weighted quantitative meta-analysis is appropriate rather than trend-only vote-counting.

## Inputs

- CSV/XLS/XLSX/TXT file with columns in order: compound Id, Trend/Behaviour (up-regulated or down-regulated), bibliographic Reference
- amanida sample_data (overall results from the colorectal cancer urine volatilome/metabolome systematic review, Mallafré et al. Cancers 2021, 13(11), 2534)

## Outputs

- S4 object with a vote-counting table accessible via @vote (compound identifier, total votes, number of contributing reports)
- vote_counts_table.csv (compound identifier, summed votes, number of articles)
- Optional visualizations: vote_plot bar plot of vote-counting and explore_plot bar plot of reports divided by trend

## How to apply

Read the identifier/trend/reference table with amanida_read in mode = "qual", passing coln = c("Compound Name", "Behaviour", "References") in that exact order. Optionally run check_names(datafile) to harmonize identifiers (chemical name, InChI, InChIKey, SMILES) to a common PubChem ID via webchem before counting. Then call amanida_vote(data_votes), which scores each compound-study observation as +1 for up-regulation, -1 for down-regulation and 0 for no trend, and aggregates votes per compound across reports. Retrieve the vote-counting table from the S4 result object via the @vote slot; it reports total votes together with the number of articles contributing to each compound. Rationale: metabolomics studies rarely disclose the statistical information (p-value, fold-change, SD) needed for quantitative meta-analysis, so trend-only vote-counting enables combining overall results.

## Related tools

- **amanida** (Provides amanida_read (mode = "qual") and amanida_vote for qualitative vote-counting, plus vote_plot/explore_plot visualization) — https://github.com/mariallr/amanida
- **R** (Runtime environment in which the amanida package functions are executed)
- **webchem** (Used by check_names to search compound names/InChI/InChIKey/SMILES in PubChem and harmonize identifiers before vote-counting)

## Examples

```
coln = c("Compound Name", "Behaviour", "References"); data_votes <- amanida_read(input_file, mode = "qual", coln, separator = ";"); vote_result <- amanida_vote(data_votes)
```

## Evaluation signals

- Every compound-study observation maps to a vote of +1 (up-regulation), -1 (down-regulation) or 0 (no trend); no other vote values appear in the output table.
- The result object is an S4 object with the vote-counting table retrievable via the @vote slot.
- Each compound row carries the number of contributing reports; the number of articles for a compound equals the number of studies reporting that compound's trend.
- Running with check_names beforehand yields one harmonized PubChem ID per compound with duplicates checked, so votes are not split across duplicate identifier spellings.
- Plotting with vote_plot/explore_plot shows bar plots whose counts match the vote table for the same dataset.

## Limitations

- Vote-counting uses only trend labels and discards magnitude/significance information; it cannot weight studies by sample size (unlike amanida's quantitative mode).
- Identifiers must be curated/harmonized beforehand — the article states 'Before using `Amanida` is important the curation of the identifiers' and check_names requires name/InChI/InChIKey/SMILES formats resolvable via PubChem.
- Mixed results for a compound (some studies up-regulated, some down-regulated) can cancel out in summed votes; explore_plot(type = "mix") exists specifically to surface such discrepancies.

## Evidence

- [readme] Vote-counting scoring rule: [evidence span withheld — anchor status: external_doc]
- [readme] Input reading for qualitative analysis: [evidence span withheld — anchor status: not_found]
