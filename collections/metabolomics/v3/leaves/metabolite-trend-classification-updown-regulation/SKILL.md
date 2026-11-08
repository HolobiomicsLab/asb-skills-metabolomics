---
name: metabolite-trend-classification-updown-regulation
description: Use when input data consist only of compound identifiers and trend/behaviour
  labels (up-regulated, down-regulated, or no trend) per study — i.e. studies that
  do not disclose p-values, fold-changes, or standard deviations — and a qualitative
  per-compound meta-analysis is needed.
when_to_use_negative:
- The dataset includes p-values and fold-changes (with study size n) — use amanida's
  quantitative weighted meta-analysis (compute_amanida, mode = 'quan') instead, which
  combines p-values via weighted Fisher's method and fold-changes via weighted log-average.
- The input already contains vote-counting results rather than raw per-study trend
  labels.
- Raw data with standard deviations/variances are available, permitting standard quantitative
  meta-analysis methods.
edam_operation: http://edamontology.org/operation_3454
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: amanida
  role: Provides amanida_read (mode = 'qual'), check_names (identifier harmonization
    to PubChem ID), and amanida_vote (vote-counting with +1/-1/0 scoring) plus vote_plot
    and explore_plot visualizations
  repo: https://github.com/mariallr/amanida
- name: R
  role: Runtime environment in which the amanida package functions are executed
- name: webchem
  role: Used by check_names (comp.inf = TRUE path) to search PubChem and convert chemical
    names, InChI, InChIKey, and SMILES to a common nomenclature before vote-counting
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btab591
    title: 'Amanida: an R package for meta-analysis of metabolomics non-integral data'
  source_content_hash: sha256:bf87efe445cc7e4425aef767ca936b7aadbad558a9f451d39dd00df881b3da9e
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
  edam_operation: http://edamontology.org/operation_3454
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# metabolite-trend-classification-updown-regulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Qualitative vote-counting meta-analysis for metabolomics compounds when studies report only the direction of change (trend) rather than p-values or fold-changes: each compound-study observation is scored +1 (up-regulated), -1 (down-regulated), or 0 (no trend), and votes are summed per compound using the amanida R package's amanida_vote function.

## When to use

Use when input data consist only of compound identifiers and trend/behaviour labels (up-regulated, down-regulated, or no trend) per study — i.e. studies that do not disclose p-values, fold-changes, or standard deviations — and a qualitative per-compound meta-analysis is needed.

## When NOT to use

- The dataset includes p-values and fold-changes (with study size n) — use amanida's quantitative weighted meta-analysis (compute_amanida, mode = 'quan') instead, which combines p-values via weighted Fisher's method and fold-changes via weighted log-average.
- The input already contains vote-counting results rather than raw per-study trend labels.
- Raw data with standard deviations/variances are available, permitting standard quantitative meta-analysis methods.

## Inputs

- csv, xls/xlsx, or txt file with columns: compound identifier, trend/behaviour label (up-regulated / down-regulated / no trend), and bibliographic reference
- amanida-bundled sample_data from the colorectal cancer urine volatilome/metabolome systematic review (Mallafré et al., Cancers 2021, 13(11), 2534)

## Outputs

- S4 object with a vote-counting table accessible via @vote containing compound identifier, summed votes, and number of reports/articles per compound
- vote_counts_table.csv (compound identifier, summed votes, number of articles)
- Optional visualizations: vote_plot (bar plot of qualitative results) and explore_plot (reports divided by trend including total vote-counting)

## How to apply

Load the data with amanida_read using mode = 'qual' and coln in the order Id, Trend (Behaviour), Reference (e.g. c('Compound Name', 'Behaviour', 'References')); supported file formats are csv, xls/xlsx, and txt with an explicit separator. Optionally run check_names to harmonize identifiers (chemical names, InChI, InChIKey, SMILES) to a common PubChem ID via the webchem package, following the Metabolites Merging Strategy, before calling amanida_vote(data_votes). The vote-counting mechanism scores each compound-study observation +1 for up-regulation, -1 for down-regulation, and 0 for no trend; the total votes are divided by the number of reports per compound. Access results via the S4 object slot vote_results@vote, which contains compound identifier, summed votes, and the number of contributing articles. Optionally visualize with vote_plot (bar plot of vote-counts, subsettable with a counts cut-off) or explore_plot (type = 'all'/'sub'/'mix', where 'mix' highlights compounds with discrepant up- and down-regulated reports).

## Related tools

- **amanida** (Provides amanida_read (mode = 'qual'), check_names (identifier harmonization to PubChem ID), and amanida_vote (vote-counting with +1/-1/0 scoring) plus vote_plot and explore_plot visualizations) — https://github.com/mariallr/amanida
- **R** (Runtime environment in which the amanida package functions are executed)
- **webchem** (Used by check_names (comp.inf = TRUE path) to search PubChem and convert chemical names, InChI, InChIKey, and SMILES to a common nomenclature before vote-counting)

## Examples

```
vote_result <- amanida_vote(amanida_read(system.file("extdata", "dataset2.csv", package = "amanida"), mode = "qual", c("Compound Name", "Behaviour", "References"), separator = ";"))
```

## Evaluation signals

- The output S4 object exposes a table at vote_results@vote; verify it exists and has compound identifier, vote, and report-count columns.
- Every per-study vote must be exactly +1, -1, or 0 corresponding to up-, down-, and no-trend labels; spot-check against the input Behaviour column.
- Each compound's total vote should equal the sum of its per-study votes (divided by the number of reports), and the report count should match the number of distinct references citing that compound.
- Compounds whose reports contain both up- and down-regulated entries should surface as discrepancies when explore_plot is called with type = 'mix'.
- Identifiers absent from public databases after check_names should be flagged rather than silently dropped; duplicate PubChem IDs should be merged.

## Limitations

- Vote-counting is purely qualitative: it discards magnitude information and only reflects direction of change, so it cannot estimate effect size or global significance.
- Studies rarely disclose all statistical information needed for quantitative meta-analysis, which is the motivating gap — but compounds with few reports (low votecount_lim) yield unstable votes.
- Identifier harmonization depends on PubChem lookup via webchem (comp.inf = TRUE); unresolvable or ambiguous IDs can prevent correct merging of the same metabolite across studies.

## Evidence

- [readme] vote-counting mechanism: [evidence span withheld — anchor status: external_doc]
