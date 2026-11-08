---
name: benchmark-reaction-set-curation
description: Use when validating a reaction-rules prediction engine against a curated
  set of experimentally characterized biosynthetic reactions, to report coverage proportions
  and their statistical significance.
when_to_use_negative:
- The analysis goal is de novo pathway reconstruction from paired transcriptomic-metabolomic
  data rather than benchmarking rule-set coverage against known reactions
- The rule set of interest is the loose RetroRules dataset, whose spurious reaction-Pfam
  links make it unsuitable for strict validation benchmarks
- The candidate reactions involve stereoisomeric conversions or multi-substrate steps,
  which MEANtools filters or cannot anticipate (mass spectrometric data does not capture
  stereoisomers; RetroRules rules are single-substrate)
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0605
tools:
- name: MEANtools
  role: Reaction-rule prediction engine used to check whether each characterized benchmark
    reaction is anticipated; run with the strict RetroRules dataset
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
- name: RetroRules
  role: Source of reaction rules (loose/medium/strict datasets) benchmarked for coverage
    of experimentally characterized biosynthetic reactions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:f86970edc3b76520e92a762102421a9e1ef75d544b7cd272411ece3e2e751015
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IntegrativeBioinformaticsLab/meantools
  tools:
  - MEANtools
  - RetroRules
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# benchmark-reaction-set-curation

## Summary

Curates and benchmarks a reference set of experimentally characterized biochemical reactions against MEANtools reaction-rule predictions derived from RetroRules, quantifying coverage and testing its statistical significance with a chi-square test.

## When to use

Use when validating a reaction-rules prediction engine against a curated set of experimentally characterized biosynthetic reactions, to report coverage proportions and their statistical significance.

## When NOT to use

- The analysis goal is de novo pathway reconstruction from paired transcriptomic-metabolomic data rather than benchmarking rule-set coverage against known reactions
- The rule set of interest is the loose RetroRules dataset, whose spurious reaction-Pfam links make it unsuitable for strict validation benchmarks
- The candidate reactions involve stereoisomeric conversions or multi-substrate steps, which MEANtools filters or cannot anticipate (mass spectrometric data does not capture stereoisomers; RetroRules rules are single-substrate)

## Inputs

- S2 Data file listing 187 experimentally characterized biochemical reactions (Zenodo record 15697913)
- Combined_small_middle_big_datasets.csv containing loose, medium, and strict RetroRules reaction-rule datasets (Zenodo record 15697913)
- Strict RetroRules reaction rules-enzyme association dataset cross-referenced with Rhea and KEGG-orthology databases

## Outputs

- reaction_presence_validation.csv tagging each of the 187 reaction IDs as found or missing (134 found / 53 missing, 72% presence)
- Chi-square test result on presence counts (chi-square = 35.10, DF = 1, p < 0.001) recorded alongside the contingency table

## How to apply

Download the S2 Data file listing the 187 experimentally characterized biochemical reactions and the Combined_small_middle_big_datasets.csv (loose, medium, strict RetroRules rule sets) from Zenodo record 15697913. Load the strict dataset — which contains only experimentally validated reaction-enzyme entries cross-referenced with Rhea and KEGG-orthology — into MEANtools, since the loose set retains spurious reaction-Pfam links. For each of the 187 characterized reactions, check whether MEANtools reaction-rule prediction anticipates the reaction, tagging each reaction ID as found or missing; expect 134 found and 53 missing (72% reaction presence) and write reaction_presence_validation.csv. Compute a chi-square test on the presence counts with DF = 1, recording the statistic 35.10 and p < 0.001 alongside the table.

## Related tools

- **MEANtools** (Reaction-rule prediction engine used to check whether each characterized benchmark reaction is anticipated; run with the strict RetroRules dataset) — https://github.com/IntegrativeBioinformaticsLab/meantools
- **RetroRules** (Source of reaction rules (loose/medium/strict datasets) benchmarked for coverage of experimentally characterized biosynthetic reactions)

## Examples

```
python run_benchmark.py --reactions S2_data_187_reactions.csv --rules Combined_small_middle_big_datasets.csv --ruleset strict --output reaction_presence_validation.csv --chisq
```

## Evaluation signals

- Recovered reaction-presence count equals 134 of 187 experimentally characterized reactions (72% coverage)
- Chi-square statistic matches 35.10 with DF = 1 and p < 0.001
- Found and missing reaction IDs are written to reaction_presence_validation.csv with no untagged entries
- Benchmark was run against the strict dataset (experimentally validated entries only), not the loose or medium rule sets
- Tabulated found + missing counts sum to exactly 187 reactions

## Limitations

- Coverage results depend on using the strict RetroRules dataset; the loose set retains spurious reaction-Pfam links after applying the Pfam cutoff of 6
- The benchmark cannot cover reactions that MEANtools structurally cannot anticipate, e.g., stereoisomeric conversions filtered to avoid complexity and two-substrate reactions like tyrosine + p-coumaroyl-CoA to p-coumaroyltyramine
- The 187-reaction reference set and chi-square significance reflect the paper's specific curation; other benchmark sets may yield different coverage

## Evidence

- [results] Of 187 experimentally characterized biochemical reactions tested, 134 were found in RetroRules (72% coverage), and this presence was significantly higher than expected (χ2 = 35.10; DF = 1; p < 0.001).: [evidence span withheld — anchor status: not_found]
- [supplementary] The strict dataset contains only experimentally validated entries: "The strict dataset contains only experimentally validated entries"
- [discussion] These spurious links, coming from RetroRules, have been kept in the loose dataset after applying the Pfam cutoff of 6, which highlights the importance of using the strict rules dataset when: "These spurious links, coming from RetroRules, have been kept in the loose dataset after applying the Pfam cutoff of 6, which highlights the importance of using the strict rules dataset when"
- [supplementary] reaction rules-enzyme associations from the RetroRules database, cross-referenced with the Rhea and KEGG-orthology database: "reaction rules-enzyme associations from the RetroRules database, cross-referenced with the Rhea and KEGG-orthology database"
