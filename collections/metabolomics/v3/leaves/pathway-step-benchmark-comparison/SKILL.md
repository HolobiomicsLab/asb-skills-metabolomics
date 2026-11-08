---
name: pathway-step-benchmark-comparison
description: Use when a pathway-prediction run (e.g., MEANtools reaction anticipation
  on the Jeon 2020 tomato paired-omics dataset) must be scored against published pathway
  steps.
when_to_use_negative:
- The benchmark pathway contains multi-substrate conversions (e.g., p-coumaroyltyramine
  formation from tyrosine and p-coumaroyl-CoA), since RetroRules-based rules are designed
  for single-substrate reactions only
- The published steps hinge on stereoisomeric conversions (e.g., phenylalanine to
  p-coumaroyl-CoA), which are not captured by the mass spectrometric data and are
  filtered by MEANtools
- The run used the loose RetroRules dataset, where spurious reaction-Pfam links (e.g.,
  AminoTran_1_2, Glyco_transf_20) survive the Pfam cutoff of 6 and can inflate the
  match count
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: MEANtools
  role: Generates the reaction-anticipation predictions that are scored against the
    published pathway steps
  repo: github.com/IntegrativeBioinformaticsLab/meantools
- name: RetroRules
  role: Source of the strict (experimentally validated) reaction-rule dataset used
    for the benchmarked prediction run
  repo: https://zenodo.org/records/15697913/files/Combined_small_middle_big_datasets.csv
- name: LOTUS
  role: Metabolite structure database used during mass-feature annotation preceding
    the benchmark comparison
  repo: github.com/IntegrativeBioinformaticsLab/meantools
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:a092dfe0383b880e00a9494346adc75686354d829c478a63ef0d838b3f659da1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClusterONE
  - LOTUS
  - MEANtools
  - RetroRules
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# pathway-step-benchmark-comparison

## Summary

Benchmarks computationally anticipated biosynthetic reaction steps against a published, experimentally characterized pathway (e.g., the 7-step tomato falcarindiol pathway of Jeon and colleagues, 2020), tabulating which steps are predicted or missed.

## When to use

Use when a pathway-prediction run (e.g., MEANtools reaction anticipation on the Jeon 2020 tomato paired-omics dataset) must be scored against published pathway steps.

## When NOT to use

- The benchmark pathway contains multi-substrate conversions (e.g., p-coumaroyltyramine formation from tyrosine and p-coumaroyl-CoA), since RetroRules-based rules are designed for single-substrate reactions only
- The published steps hinge on stereoisomeric conversions (e.g., phenylalanine to p-coumaroyl-CoA), which are not captured by the mass spectrometric data and are filtered by MEANtools
- The run used the loose RetroRules dataset, where spurious reaction-Pfam links (e.g., AminoTran_1_2, Glyco_transf_20) survive the Pfam cutoff of 6 and can inflate the match count

## Inputs

- MEANtools reaction-anticipation output from the strict RetroRules ruleset (extracted from the combined RetroRules ruleset CSV on Zenodo)
- Published pathway step list (e.g., the 7 falcarindiol biosynthetic steps from Jeon and colleagues, 2020)
- Per-gene correlation statistics for predicted enzyme-metabolite steps (Pearson correlation, p-value)

## Outputs

- pathway_steps_comparison.csv tabulating each published step as predicted or not predicted
- Count of correctly anticipated steps (e.g., 5 of 7) with matched gene predictions and correlation statistics per step

## How to apply

Load the published pathway definition (the 7 falcarindiol biosynthetic steps reported by Jeon and colleagues, 2020) and the output of the MEANtools reaction-anticipation run performed with the strict RetroRules dataset (only experimentally validated entries, from the Combined_small_middle_big_datasets.csv on Zenodo). Classify each published step as predicted or not predicted, recording the predicted gene (e.g., Solyc12g100250.1 for the crepenynic acid to dehydrocrepenynic acid conversion, correlation 0.744, p-value 1.106E-10) and any supporting statistics. Compare both strict and loose ruleset runs where available, since both correctly predicted the enzyme for the octadecadiene-diynoic acid to metabolite 6 conversion, but the loose set can retain spurious reaction-Pfam links after the Pfam cutoff of 6. Tabulate the per-step comparison and report the fraction anticipated (five of seven in the reference run).

## Related tools

- **MEANtools** (Generates the reaction-anticipation predictions that are scored against the published pathway steps) — github.com/IntegrativeBioinformaticsLab/meantools
- **RetroRules** (Source of the strict (experimentally validated) reaction-rule dataset used for the benchmarked prediction run) — https://zenodo.org/records/15697913/files/Combined_small_middle_big_datasets.csv
- **LOTUS** (Metabolite structure database used during mass-feature annotation preceding the benchmark comparison) — github.com/IntegrativeBioinformaticsLab/meantools

## Evaluation signals

- Reference outcome: five of seven falcarindiol pathway steps anticipated under strict settings
- The crepenynic acid to dehydrocrepenynic acid conversion is matched to Solyc12g100250.1 with correlation 0.744 and p-value 1.106E-10, agreeing with the desaturase identified by Jeon and colleagues
- The enzyme for the octadecadiene-diynoic acid to metabolite 6 conversion is predicted by both strict and loose runs
- The comparison table contains exactly the 7 published steps, each labeled predicted or not predicted
- Spurious Pfam associations (e.g., AminoTran_1_2, Glyco_transf_20) are absent from the strict-ruleset match list

## Limitations

- Stereoisomeric conversions are filtered by MEANtools and invisible to mass spectrometric data, so affected steps will register as misses
- Multi-substrate reactions (e.g., tyrosine plus p-coumaroyl-CoA) fall outside the single-substrate RetroRules rule design
- The loose ruleset can keep spurious reaction-Pfam links after the Pfam cutoff of 6, inflating apparent matches

## Evidence

- [abstract] Five of seven steps matched: "MEANtools correctly anticipated five out of seven steps of the characterized pathway"
- [discussion] Both rulesets recover key enzyme: "Most importantly, both predictions correctly predict the enzyme associated to the conversion of octadecadiene- diynoic acid into metabolite 6, as reported by Jeon and colleagues (2020)"
- [discussion] Use strict ruleset for benchmarking: "These spurious links, coming from RetroRules, have been kept in the loose dataset after applying the Pfam cutoff of 6, which highlights the importance of using the strict rules dataset"
- [supplementary] Strict dataset definition: "The strict dataset contains only experimentally validated entries"
- [readme] Readme points to documentation: "General documentation for MEANtools. Read the documentation here: https://meantools.readthedocs.io/en/latest/"
