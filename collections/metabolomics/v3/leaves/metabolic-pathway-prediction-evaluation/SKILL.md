---
name: metabolic-pathway-prediction-evaluation
description: Use when you need to quantify and statistically validate how much of
  a reference set of experimentally characterized biosynthetic reactions a computational
  reaction-rule predictor (e.g., MEANtools/RetroRules rules) can anticipate.
when_to_use_negative:
- The goal is de novo pathway discovery from multi-omics data rather than benchmarking
  predictor coverage — use the MEANtools discovery workflow (MR correlation, ClusterONE
  feature clusters) instead
- The reaction set includes multi-substrate conversions or stereoisomeric steps (e.g.,
  phenylalanine to p-coumaroyl-CoA), since RetroRules-based rules are designed for
  single-substrate reactions and MEANtools filters stereomeric rules
- Only the loose RetroRules ruleset is available and spurious reaction-Pfam associations
  cannot be excluded — benchmark results would be inflated
edam_operation: http://edamontology.org/operation_3660
tools:
- name: MEANtools
  role: Reaction-rule prediction engine used to check whether it anticipates each
    characterized reaction (strict RetroRules dataset loaded into MEANtools)
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
- name: RetroRules
  role: Source of general reaction rules (loose/medium/strict datasets) benchmarked
    for coverage of characterized biosynthetic reactions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:6b40fa9b2a9a8b877d6a4c810a38ac432d4e1f2e5336316198467f79175fb42f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IntegrativeBioinformaticsLab/meantools
  tools:
  - MEANtools
  - RetroRules
  edam_operation: http://edamontology.org/operation_3660
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# metabolic-pathway-prediction-evaluation

## Summary

Benchmark a rule-based reaction-prediction engine (MEANtools reaction rules built on RetroRules) against a curated set of experimentally characterized biosynthetic reactions, quantifying coverage (134/187, 72%) and testing significance with a chi-square goodness-of-fit test (χ2 = 35.10; DF = 1; p < 0.001).

## When to use

Use when you need to quantify and statistically validate how much of a reference set of experimentally characterized biosynthetic reactions a computational reaction-rule predictor (e.g., MEANtools/RetroRules rules) can anticipate.

## When NOT to use

- The goal is de novo pathway discovery from multi-omics data rather than benchmarking predictor coverage — use the MEANtools discovery workflow (MR correlation, ClusterONE feature clusters) instead
- The reaction set includes multi-substrate conversions or stereoisomeric steps (e.g., phenylalanine to p-coumaroyl-CoA), since RetroRules-based rules are designed for single-substrate reactions and MEANtools filters stereomeric rules
- Only the loose RetroRules ruleset is available and spurious reaction-Pfam associations cannot be excluded — benchmark results would be inflated

## Inputs

- S2 Data file listing 187 experimentally characterized biochemical reactions (reaction IDs)
- Combined_small_middle_big_datasets.csv (RetroRules loose, medium, and strict reaction-rule datasets) from Zenodo record 15697913
- MEANtools reaction-rule prediction engine with RetroRules SQLite database

## Outputs

- reaction_presence_validation.csv with found/missing reaction IDs
- Coverage table (134 found, 53 missing; 72% reaction presence)
- Chi-square result (χ2 = 35.10; DF = 1; p < 0.001)

## How to apply

Download the S2 Data file listing the 187 experimentally characterized biochemical reactions and the Combined_small_middle_big_datasets.csv (loose, medium, strict RetroRules reaction-rule datasets) from Zenodo record 15697913. Load the strict dataset — which contains only experimentally validated reaction-enzyme entries cross-referenced with the Rhea and KEGG-orthology databases — into MEANtools; prefer strict over loose because the loose ruleset retains spurious reaction-Pfam links even after the Pfam cutoff of 6. For each of the 187 characterized reactions, check whether MEANtools reaction-rule prediction anticipates the reaction, tagging each reaction ID as found or missing. Tabulate the results, expecting 134 found and 53 missing (72% reaction presence), write found/missing IDs to reaction_presence_validation.csv, and compute a chi-square test on the presence counts with DF = 1 to confirm enrichment above chance.

## Related tools

- **MEANtools** (Reaction-rule prediction engine used to check whether it anticipates each characterized reaction (strict RetroRules dataset loaded into MEANtools)) — https://github.com/IntegrativeBioinformaticsLab/meantools
- **RetroRules** (Source of general reaction rules (loose/medium/strict datasets) benchmarked for coverage of characterized biosynthetic reactions)

## Examples

```
import pandas as pd; rules = pd.read_csv('Combined_small_middle_big_datasets.csv'); strict = rules[rules['dataset']=='strict']
```

## Evaluation signals

- Recovered reaction-presence count equals 134 of 187 experimentally characterized reactions (72% coverage)
- Chi-square result matches the reported χ2 = 35.10 with DF = 1 and p < 0.001
- Found + missing reaction IDs sum to exactly 187, with no duplicate or unclassified IDs
- Results are derived from the strict dataset (experimentally validated entries only), not the loose or medium datasets
- reaction_presence_validation.csv is produced with found/missing reaction IDs and the presence table alongside the chi-square statistics

## Limitations

- Coverage reflects only the 187-reaction reference set; coverage of uncharacterized reactions cannot be assessed this way
- RetroRules-based rules are designed for single-substrate reactions only, so two-substrate conversions (e.g., tyrosine plus p-coumaroyl-CoA to p-coumaroyltyramine) will be missed
- MEANtools filters rules involving stereomeric structures, and mass spectrometric data does not capture stereoisomers, which can hinder reconstruction of steps like phenylalanine to p-coumaroyl-CoA
- The loose RetroRules ruleset keeps spurious reaction-Pfam links even after a Pfam cutoff of 6, so loose-set results are not comparable to strict-set results

## Evidence

- [other] Of 187 experimentally characterized biochemical reactions tested, 134 were found in RetroRules (72% coverage), and this presence was significantly higher than expected (χ2 = 35.10; DF = 1; p < 0.001).: [evidence span withheld — anchor status: not_found]
- [supplementary] The strict dataset contains only experimentally validated entries: "The strict dataset contains only experimentally validated entries"
- [discussion] The three reaction rules datasets resulting from RetroRules, loose, medium, and strict are available in a single CSV file in the https://zenodo.org/records/15697913.: "The three reaction rules datasets resulting from RetroRules, loose, medium, and strict are available in a single CSV file in the https://zenodo.org/records/15697913."
