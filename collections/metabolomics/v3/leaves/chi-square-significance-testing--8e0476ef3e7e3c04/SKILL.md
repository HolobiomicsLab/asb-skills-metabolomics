---
name: chi-square-significance-testing
description: Use when tabulating binary found/missing outcomes (e.g., 134 of 187 experimentally
  characterized reactions present in RetroRules via MEANtools) and you must test if
  the presence rate exceeds chance (χ2, DF = 1).
when_to_use_negative:
- Input is not a binary found/missing categorization — e.g., continuous correlation
  or edge-score data from MEANtools FCs, which require MR/exponential-decay weighting
  instead
- Expected cell counts are too small for a valid chi-square approximation; use exact
  tests (e.g., Fisher's exact) instead
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: MEANtools
  role: Generates the found/missing reaction-presence calls that feed the chi-square
    test, using the strict RetroRules reaction-rule dataset
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:6610328e698471f9581a17b8d6c7b04d87ec21ee724d611303f9d121713eed18
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IntegrativeBioinformaticsLab/meantools
  tools:
  - MEANtools
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# chi-square-significance-testing

## Summary

Applies a chi-square goodness-of-fit test to assess whether an observed coverage count (e.g., biosynthetic reactions recovered from MEANtools using the strict RetroRules dataset) deviates significantly from chance expectation.

## When to use

Use when tabulating binary found/missing outcomes (e.g., 134 of 187 experimentally characterized reactions present in RetroRules via MEANtools) and you must test if the presence rate exceeds chance (χ2, DF = 1).

## When NOT to use

- Input is not a binary found/missing categorization — e.g., continuous correlation or edge-score data from MEANtools FCs, which require MR/exponential-decay weighting instead
- Expected cell counts are too small for a valid chi-square approximation; use exact tests (e.g., Fisher's exact) instead

## Inputs

- S2 Data file listing the 187 experimentally characterized biochemical reactions (Zenodo record 15697913)
- reaction_presence_validation.csv with found/missing reaction ID tags from MEANtools prediction against the strict RetroRules dataset

## Outputs

- Presence/absence contingency table (found = 134, missing = 53, 72% coverage)
- Chi-square test result: χ2 = 35.10, DF = 1, p < 0.001, recorded alongside the table

## How to apply

First produce the binary presence table: run MEANtools reaction-rule prediction against the strict RetroRules dataset (experimentally validated entries cross-referenced with Rhea and KEGG-orthology, from Combined_small_middle_big_datasets.csv on Zenodo record 15697913), tagging each of the 187 characterized reaction IDs as found or missing. Tabulate the counts (expected: 134 found, 53 missing, 72% presence). Then compute a chi-square test on the presence counts with DF = 1, comparing observed presence against the chance-expected split. Record the statistic, degrees of freedom, and p-value alongside the table in reaction_presence_validation.csv.

## Related tools

- **MEANtools** (Generates the found/missing reaction-presence calls that feed the chi-square test, using the strict RetroRules reaction-rule dataset) — https://github.com/IntegrativeBioinformaticsLab/meantools

## Examples

```
from scipy.stats import chisquare; chi2, p = chisquare([134, 53])  # DF=1, expect chi2≈35.10, p<0.001
```

## Evaluation signals

- Recovered reaction-presence count must equal 134 of 187 (72% coverage)
- Chi-square statistic must match 35.10 with DF = 1 and p < 0.001 as reported in the paper
- Found + missing counts must sum to 187 (the size of the S2 Data reference set)
- The strict dataset used must be the experimentally validated entries cross-referenced with Rhea and KEGG-orthology, not the loose or medium rule sets

## Limitations

- Coverage assessment depends on the strict RetroRules dataset; the loose ruleset retains spurious reaction-Pfam links after the Pfam cutoff of 6, which would inflate apparent coverage
- MEANtools filters stereomeric conversion rules and its rules are designed for single-substrate reactions only, so some of the 53 missing reactions may reflect these preprocessing constraints rather than true database gaps

## Evidence

- [results] Of 187 experimentally characterized biochemical reactions tested, 134 were found in RetroRules (72% coverage), and this presence was significantly higher than expected (χ2 = 35.10; DF = 1; p < 0.001).: [evidence span withheld — anchor status: not_found]
- [methods] Load the strict dataset, which contains only experimentally validated reaction-enzyme entries cross-referenced with the Rhea and KEGG-orthology databases, into MEANtools.: [evidence span withheld — anchor status: not_found]
- [discussion] The three reaction rules datasets resulting from RetroRules, loose, medium, and strict are available in a single CSV file in the https://zenodo.org/records/15697913.: "The three reaction rules datasets resulting from RetroRules, loose, medium, and strict are available in a single CSV file in the https://zenodo.org/records/15697913"
