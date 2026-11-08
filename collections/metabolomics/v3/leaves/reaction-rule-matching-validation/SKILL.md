---
name: reaction-rule-matching-validation
description: Use when validating that MEANtools' reaction-rule prediction (strict
  RetroRules dataset) anticipates known experimentally characterized biochemical reactions,
  or when reproducing reported coverage percentages and chi-square significance for
  reaction-rule databases.
when_to_use_negative:
- The goal is predicting novel pathway steps in a user's own metabolomics/transcriptomics
  data rather than benchmarking rule-database coverage.
- Only the loose or medium RetroRules datasets are available — the loose set retains
  spurious reaction-Pfam links and the medium set includes ECDomainMiner predictions,
  so coverage against the strict validated set cannot be reproduced.
- The reference reactions rely on stereomeric or multi-substrate conversions that
  RetroRules-based rules in MEANtools cannot represent (single-substrate rules only,
  stereomeric rules filtered).
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MEANtools
  role: Reaction-rule prediction engine used to test whether each characterized reaction
    is anticipated by the strict RetroRules dataset
  repo: github.com/IntegrativeBioinformaticsLab/meantools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:55d89f6ae87519630a960f5b373fa49923a0f362da53f3061f8f4ca15d8be91f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MEANtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# reaction-rule-matching-validation

## Summary

Benchmarks the coverage of RetroRules-derived reaction rules in MEANtools against a reference set of experimentally characterized biosynthetic reactions, tabulating found/missing reaction IDs and testing significance with a chi-square statistic.

## When to use

Use when validating that MEANtools' reaction-rule prediction (strict RetroRules dataset) anticipates known experimentally characterized biochemical reactions, or when reproducing reported coverage percentages and chi-square significance for reaction-rule databases.

## When NOT to use

- The goal is predicting novel pathway steps in a user's own metabolomics/transcriptomics data rather than benchmarking rule-database coverage.
- Only the loose or medium RetroRules datasets are available — the loose set retains spurious reaction-Pfam links and the medium set includes ECDomainMiner predictions, so coverage against the strict validated set cannot be reproduced.
- The reference reactions rely on stereomeric or multi-substrate conversions that RetroRules-based rules in MEANtools cannot represent (single-substrate rules only, stereomeric rules filtered).

## Inputs

- S2 Data file listing 187 experimentally characterized biochemical reactions (reaction IDs)
- Combined_small_middle_big_datasets.csv from Zenodo record 15697913 (loose, medium, and strict RetroRules reaction-rule datasets)
- Strict RetroRules dataset: experimentally validated reaction-enzyme entries cross-referenced with Rhea and KEGG-orthology databases

## Outputs

- reaction_presence_validation.csv containing found and missing reaction IDs
- Coverage table: 134 found / 53 missing of 187 reactions (72% presence)
- Chi-square result: statistic 35.10, DF = 1, p < 0.001

## How to apply

Download the S2 Data file listing the 187 experimentally characterized biochemical reactions together with Combined_small_middle_big_datasets.csv (loose, medium, strict RetroRules reaction-rule datasets) from Zenodo record 15697913. Load the strict dataset — which contains only experimentally validated reaction-enzyme entries cross-referenced with Rhea and KEGG-orthology — into MEANtools, rather than the loose set that retains spurious reaction-Pfam links. For each of the 187 characterized reactions, check whether MEANtools reaction-rule prediction anticipates the reaction and tag each reaction ID as found or missing. Tabulate results (expected: 134 found, 53 missing, i.e. 72% presence), write reaction_presence_validation.csv, and compute a chi-square test with DF = 1, expecting statistic 35.10 and p < 0.001. Rationale: benchmarking against the strict, experimentally validated ruleset ensures the coverage estimate reflects validated reaction-enzyme associations, not ECDomainMiner predictions or loose-set noise.

## Related tools

- **MEANtools** (Reaction-rule prediction engine used to test whether each characterized reaction is anticipated by the strict RetroRules dataset) — github.com/IntegrativeBioinformaticsLab/meantools

## Evaluation signals

- Recovered reaction-presence count must equal 134 of 187 (72%)
- Chi-square statistic must match 35.10 with DF = 1 and p < 0.001
- Found + missing reaction IDs must total exactly 187 with no duplicates
- reaction_presence_validation.csv must be produced with per-reaction found/missing tags
- Only the strict dataset (experimentally validated entries) may be used — loose/medium-set runs should not be reported as the validated coverage figure

## Limitations

- RetroRules-based rules in MEANtools are designed for single-substrate reactions only, so two-substrate conversions (e.g. tyrosine plus p-coumaroyl-CoA to p-coumaroyltyramine) are missed.
- MEANtools filters RetroRules rules involving stereomeric structures to avoid complexity, so stereoisomer-dependent reactions are not anticipated.
- The loose ruleset associates predictions with unrelated Pfams, producing spurious links — strict-set results must be used for validated claims.

## Evidence

- [discussion] The three reaction rules datasets resulting from RetroRules, loose, medium, and strict are available in a single CSV file in the https://zenodo.org/records/15697913.: "The three reaction rules datasets resulting from RetroRules, loose, medium, and strict are available in a single CSV file in the https://zenodo.org/records/15697913."
- [supplementary] Strict dataset contains only experimentally validated entries: "The strict dataset contains only experimentally validated entries"
- [supplementary] reaction rules-enzyme associations from the RetroRules database, cross-referenced with the Rhea and KEGG-orthology database: "reaction rules-enzyme associations from the RetroRules database, cross-referenced with the Rhea and KEGG-orthology database"
- [discussion] spurious links highlight importance of strict rules dataset: "These spurious links, coming from RetroRules, have been kept in the loose dataset after applying the Pfam cutoff of 6, which highlights the importance of using the strict rules dataset when"
- [abstract] MEANtools correctly anticipated five out of seven steps of the characterized pathway: "MEANtools correctly anticipated five out of seven steps of the characterized pathway"
