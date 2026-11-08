---
name: retrometabolic-reaction-presence-quantification
description: Use when you have a reference set of characterized biochemical reaction
  IDs and want to quantify how many are anticipated by RetroRules reaction-rule predictions
  in MEANtools, with a statistical significance test.
when_to_use_negative:
- Your goal is de novo biosynthetic pathway reconstruction from paired transcriptomic-metabolomic
  data (e.g. mutual-rank clustering of mass features), not benchmarking rule-database
  coverage.
- You want to evaluate the loose or medium rule sets, which include ECDomainMiner
  predictions and spurious reaction–Pfam links rather than only experimentally validated
  entries.
- Your reactions lack identifiers mappable to the RetroRules/Rhea/KEGG cross-referenced
  rule-enzyme associations, so found/missing tagging cannot be assigned.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_2250
tools:
- name: MEANtools
  role: Loads the strict RetroRules reaction-rule dataset and checks which of the
    characterized reactions its prediction step anticipates, tagging each as found
    or missing
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:d83245de5b0219c16744dd659de75b9f2fc8831bbe802430cd23641ecb5d60d4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IntegrativeBioinformaticsLab/meantools
  tools:
  - MEANtools
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_2250
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# retrometabolic-reaction-presence-quantification

## Summary

Benchmark the coverage of a curated set of experimentally characterized biosynthetic reactions against MEANtools' RetroRules reaction-rule predictions, quantifying reaction presence as a percentage and testing its significance with a chi-square test on found/missing counts.

## When to use

Use when you have a reference set of characterized biochemical reaction IDs and want to quantify how many are anticipated by RetroRules reaction-rule predictions in MEANtools, with a statistical significance test.

## When NOT to use

- Your goal is de novo biosynthetic pathway reconstruction from paired transcriptomic-metabolomic data (e.g. mutual-rank clustering of mass features), not benchmarking rule-database coverage.
- You want to evaluate the loose or medium rule sets, which include ECDomainMiner predictions and spurious reaction–Pfam links rather than only experimentally validated entries.
- Your reactions lack identifiers mappable to the RetroRules/Rhea/KEGG cross-referenced rule-enzyme associations, so found/missing tagging cannot be assigned.

## Inputs

- S2 Data file listing 187 experimentally characterized biochemical reactions (reaction IDs)
- Combined_small_middle_big_datasets.csv from Zenodo record 15697913 (loose, medium, strict RetroRules reaction-rule datasets)
- RetroRules reaction rules available as SQLite database, usable directly with MEANtools

## Outputs

- reaction_presence_validation.csv with found and missing reaction IDs
- Coverage table: 134 found / 53 missing of 187 reactions (72% reaction presence)
- Chi-square result: statistic 35.10, DF = 1, p < 0.001

## How to apply

Download the S2 Data file listing the 187 experimentally characterized biochemical reactions and the Combined_small_middle_big_datasets.csv (loose, medium, strict rule datasets) from Zenodo record 15697913. Load the strict dataset — the only one restricted to experimentally validated entries cross-referenced with Rhea and KEGG-orthology — into MEANtools. For each of the 187 characterized reactions, check whether the MEANtools reaction-rule prediction anticipates it, tagging each reaction ID as found or missing. Tabulate the counts (expected: 134 found, 53 missing, i.e. 72% reaction presence), write the found/missing IDs to reaction_presence_validation.csv, and compute a chi-square test with DF = 1, expecting statistic 35.10 and p < 0.001.

## Related tools

- **MEANtools** (Loads the strict RetroRules reaction-rule dataset and checks which of the characterized reactions its prediction step anticipates, tagging each as found or missing) — https://github.com/IntegrativeBioinformaticsLab/meantools

## Examples

```
python -c "import pandas as pd; rr = pd.read_csv('Combined_small_middle_big_datasets.csv'); print(rr.head())"
```

## Evaluation signals

- Recovered presence count must equal 134 of 187 experimentally characterized reactions (72%).
- Chi-square result must match 35.10 with DF = 1 and p < 0.001.
- Missing reactions must number 53, and all found/missing IDs must be written to reaction_presence_validation.csv.
- Only the strict dataset (experimentally validated entries cross-referenced with Rhea and KEGG-orthology) should be used for the coverage call.
- Source files must trace to Zenodo record 15697913 (S2 Data and Combined_small_middle_big_datasets.csv).

## Limitations

- The strict dataset is the conservative choice: the loose dataset retains spurious reaction–Pfam links (e.g. AminoTran_1_2, Glyco_transf_20) after applying the Pfam cutoff of 6.
- RetroRules-based rules are designed for single-substrate reactions only, so multi-substrate conversions are not anticipated.
- MEANtools filters rules involving stereomeric structures to avoid complexity, so stereochemistry-dependent reactions may be reported as missing.

## Evidence

- [supplementary] The strict dataset contains only experimentally validated entries: "The strict dataset contains only experimentally validated entries"
- [discussion] The three reaction rules datasets resulting from RetroRules, loose, medium, and strict are available in a single CSV file in the https://zenodo.org/records/15697913.: "The three reaction rules datasets resulting from RetroRules, loose, medium, and strict are available in a single CSV file in the https://zenodo.org/records/15697913."
- [discussion] The RetroRules database is publicly available as SQLite database and can be used directly with MEANtools.: "The RetroRules database is publicly available as SQLite database and can be used directly with MEANtools."
