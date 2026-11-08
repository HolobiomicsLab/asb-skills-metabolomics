---
name: compound-level-vote-aggregation-across-studies
description: Use when input metabolomics studies report only the compound identifier
  and a trend label (up-regulated/down-regulated/no trend) rather than p-values, fold-changes,
  or variance, and a cross-study qualitative meta-analysis per compound is needed.
when_to_use_negative:
- 'Studies disclose p-values and fold-changes (with N): then compute_amanida''s weighted
  meta-analysis (mode = "quan", Fisher''s method for p-values, weighted log fold-change
  averaging) is the appropriate quantitative route instead of vote-counting.'
- The input requires directional effect sizes or statistical significance measures
  rather than a rankable qualitative trend — vote-counting yields only summed votes
  and report counts, no p-value or confidence interval.
- Identifiers are uncurated compound names without running check_names first when
  comp.inf = T is desired — the package requires the prior use of check_names before
  PubChem-based duplicate checking.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: amanida
  role: Provides amanida_read (mode = "qual"), amanida_vote for vote-counting computation,
    check_names for ID harmonization, and vote_plot/explore_plot visualizations
  repo: https://github.com/mariallr/amanida
- name: R
  role: Runtime environment in which the amanida package functions are executed
- name: webchem
  role: Used by check_names to search chemical name, InChI, InChIKey, and SMILES identifiers
    in PubChem and transform them to a common nomenclature
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btab591
    title: 'Amanida: an R package for meta-analysis of metabolomics non-integral data'
  source_content_hash: sha256:c778f738ca38062916fe191bd5e89467ccae339b52f709a2b17fc75842593513
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
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# compound-level-vote-aggregation-across-studies

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Qualitative meta-analysis via vote-counting: each study's trend for a compound is scored (+1 up-regulation, -1 down-regulation, 0 no trend) and votes are summed per compound, with the number of contributing articles recorded. This lets compounds be ranked across studies even when only identifier and trend labels are disclosed.

## When to use

Use when input metabolomics studies report only the compound identifier and a trend label (up-regulated/down-regulated/no trend) rather than p-values, fold-changes, or variance, and a cross-study qualitative meta-analysis per compound is needed.

## When NOT to use

- Studies disclose p-values and fold-changes (with N): then compute_amanida's weighted meta-analysis (mode = "quan", Fisher's method for p-values, weighted log fold-change averaging) is the appropriate quantitative route instead of vote-counting.
- The input requires directional effect sizes or statistical significance measures rather than a rankable qualitative trend — vote-counting yields only summed votes and report counts, no p-value or confidence interval.
- Identifiers are uncurated compound names without running check_names first when comp.inf = T is desired — the package requires the prior use of check_names before PubChem-based duplicate checking.

## Inputs

- csv/xls/xlsx/txt file of compounds with columns: Id (compound name or unique identification), Trend (Behaviour: up-regulated or down-regulated), Reference (bibliographic reference)
- Package-bundled sample_data from 'Comprehensive Volatilome and Metabolome Signatures of Colorectal Cancer in Urine: A Systematic Review and Meta-Analysis' (Mallafré et al., Cancers 2021)

## Outputs

- S4 object with a vote-counting table (access via @vote) containing compound identifier, summed votes, and number of contributing articles
- CSV export of the vote-counting table (vote_counts_table.csv)
- Optional visualizations: vote_plot bar plot of qualitative results and explore_plot showing reports divided by trend with total vote-counting

## How to apply

Load the trend-only data with `amanida_read` in mode = "qual", specifying coln in the order Id, Trend (up-/down-regulated), and Reference; csv, xls/xlsx and txt are supported. Optionally run `check_names` to harmonize identifiers (chemical name, InChI, InChIKey, SMILES are converted to a common PubChem nomenclature via webchem, based on the Metabolites Merging Strategy) and check duplicates. Then run `amanida_vote` on the parsed data: each compound-study observation contributes +1 (up), -1 (down) or 0 (no trend), votes are summed per compound, and total votes are divided by the number of reports. Results are returned in an S4 object with a vote-counting table accessible via `@vote`; write that table (compound identifier, summed votes, number of articles) to CSV. Note the amanida paper's motivation: metabolomics studies rarely disclose all statistical information needed for a meta-analysis, so vote-counting works with identifier-and-trend-labels only.

## Related tools

- **amanida** (Provides amanida_read (mode = "qual"), amanida_vote for vote-counting computation, check_names for ID harmonization, and vote_plot/explore_plot visualizations) — https://github.com/mariallr/amanida
- **R** (Runtime environment in which the amanida package functions are executed)
- **webchem** (Used by check_names to search chemical name, InChI, InChIKey, and SMILES identifiers in PubChem and transform them to a common nomenclature)

## Examples

```
data_votes <- amanida_read(input_file, mode = "qual", coln, separator = ";"); vote_result <- amanida_vote(data_votes)
```

## Evaluation signals

- Every study observation maps to exactly one vote: +1 for up-regulation, -1 for down-regulation, 0 for no trend — check no compound-study pair is dropped or double-counted after ID harmonization.
- Per-compound total vote equals the sum of individual votes and equals the sum divided by the number of reports as documented; verify against the @vote slot table.
- The number of articles recorded per compound matches the number of distinct References contributing to that compound in the input file.
- After check_names, no duplicate PubChem IDs remain unmerged; compounds appearing under multiple names in the input are consolidated into a single vote row.
- Sanity check on the bundled sample_data (colorectal cancer urine volatilome dataset): summed votes per compound should reproduce the published vote-counting table for compounds with consistent trends.

## Limitations

- Vote-counting conveys direction and consistency of regulation only — it provides no statistical significance, effect size magnitude, or confidence measure per compound.
- Dividing total votes by the number of reports means compounds reported in only a few studies can show extreme normalized votes; subsetting via explore_plot (type = "mix" with a counts cutoff) helps surface discrepant compounds.
- Metabolomics studies rarely disclose all statistical information needed for a meta-analysis; when p-values and fold-changes ARE available, the weighted quantitative path (Fisher's method, weighted fold-change averaging by study size n) is more informative.
- ID harmonization depends on PubChem lookup via webchem; identifiers not resolvable to PubChem cannot be merged or checked for duplicates, and comp.inf = T requires prior check_names use.

## Evidence

- [readme] Compound vote-counting: votes are +1 for up-regulation, -1 for down-regulation and 0 if no trend. The total votes are divided by the number of reports.: [evidence span withheld — anchor status: external_doc]
- [intro] Amanida also computes qualitative meta-analysis performing a vote-counting for compounds, including the option of only using identifier and trend labels: "Amanida also computes qualitative meta-analysis performing a vote-counting for compounds, including the option of only using identifier and trend labels"
- [readme] The IDs in format chemical name, InChI, InChIKey, and SMILES are searched in PubChem to transform all into a common nomenclature using `webchem` package.: "The IDs in format chemical name, InChI, InChIKey, and SMILES are searched in PubChem to transform all into a common nomenclature using `webchem` package."
- [readme] In this step you will obtain an S4 object with one table:

* vote-counting access by `vote_results@vote`: [evidence span withheld — anchor status: external_doc]
- [abstract] metabolomics studies rarely disclose all the statistical information needed to perform a meta-analysis.: "metabolomics studies rarely disclose all the statistical information needed to perform a meta-analysis."
