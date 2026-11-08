---
name: metabolomics-method-benchmarking
description: Use when a new feature-correspondence method must be compared against
  a published alternative (e.g. metabCombiner) by running both on the same reference/target
  feature-table pairs and tabulating match counts and quality.
when_to_use_negative:
- The goal is to reproduce the paper's main M2S examples with annotated-feature validation
  rather than to compare M2S against metabCombiner on additional dataset pairs
- The datasets have not been peak-picked and QC-filtered (e.g. XCMS peak-picking,
  QC dilution-series R2 < 0.7 filtering, RT trimming), so benchmark inputs are not
  comparable to the paper's setup
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: M2S (Match 2 Sets)
  role: Method under benchmarking; run its matching workflow on the S4–S6 dataset
    pairs
  repo: https://github.com/rjdossan/M2S
- name: metabCombiner
  role: Published alternative method run on the same three dataset pairs for comparison
- name: MATLAB
  role: Environment in which the M2S MATLAB toolbox is executed for the benchmark
    runs
  repo: https://github.com/rjdossan/M2S
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:81233e954a987c92e6a2a867bedee091ec8119d19677c0f9f820028bf38bd0d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rjdossan/M2S
  tools:
  - M2S
  - M2S (Match 2 Sets)
  - MATLAB
  - metabCombiner
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# metabolomics-method-benchmarking

## Summary

Benchmark the M2S feature-matching method against metabCombiner on the same LC–MS metabolomics dataset pairs, assembling side-by-side tables of match counts and quality metrics to demonstrate comparative advantages.

## When to use

Use when a new feature-correspondence method must be compared against a published alternative (e.g. metabCombiner) by running both on the same reference/target feature-table pairs and tabulating match counts and quality.

## When NOT to use

- The goal is to reproduce the paper's main M2S examples with annotated-feature validation rather than to compare M2S against metabCombiner on additional dataset pairs
- The datasets have not been peak-picked and QC-filtered (e.g. XCMS peak-picking, QC dilution-series R2 < 0.7 filtering, RT trimming), so benchmark inputs are not comparable to the paper's setup

## Inputs

- Paired feature tables for Datasets 4, 5, and 6 (Examples S4–S6): reference and target, samples × features matrices with RT, m/z, and FI medians per feature
- Published metabCombiner algorithm/software for the same three dataset pairs

## Outputs

- Per-pair match counts and match-quality metrics for M2S and metabCombiner (uniquely matched features, agreement with available annotations)
- Side-by-side comparison table with rows for all three pairs (S4, S5, S6) × both methods, populated with match counts and quality metrics

## How to apply

Load paired feature tables (reference and target, samples × features with RT, m/z, and FI medians per feature) for the dataset pairs used in Examples S4–S6 (Datasets 4, 5, and 6). Run the M2S workflow on each pair: initial threshold matching with large RT (~1 min) and m/z (~0.025 Da) windows, neighbor-based shift estimation with residuals, and weighted penalty-score selection. Run metabCombiner on the same pairs per its published algorithm for combining feature lists. For each pair, record match counts and match quality from both methods — number of uniquely matched features and agreement of matches with available annotations — then assemble a side-by-side comparison table per dataset pair that reproduces the qualitative advantage of M2S reported in the paper.

## Related tools

- **M2S (Match 2 Sets)** (Method under benchmarking; run its matching workflow on the S4–S6 dataset pairs) — https://github.com/rjdossan/M2S
- **metabCombiner** (Published alternative method run on the same three dataset pairs for comparison)
- **MATLAB** (Environment in which the M2S MATLAB toolbox is executed for the benchmark runs) — https://github.com/rjdossan/M2S

## Examples

```
matlab -r "addpath(genpath('M2S')); run_examples_S4_S6.m"
```

## Evaluation signals

- The comparison table contains rows for all three dataset pairs (S4, S5, S6) and both methods (M2S and metabCombiner)
- Every row has populated match counts and match-quality metrics, not empty placeholders
- The assembled table reproduces the qualitative advantage of M2S over metabCombiner reported in the paper
- Both methods were run on identical input feature tables per pair (same RT, m/z, FI medians), ensuring a fair comparison
- Match quality is gauged consistently across methods, e.g. by agreement of matches with available feature annotations

## Limitations

- Validation by annotated features is limited by the number of annotations in the datasets
- Threshold selection is subjective, guided by visual inspection of the distance plots resulting from the initial matching
- The decision on whether to use step 3 (poor-match deletion) rests with the analyst, and this optional step may inadvertently delete some correct matches
- No software version is stated for M2S, complicating exact reproducibility of the benchmark

## Evidence

- [abstract] Comparison design and conclusion: "the results of matching three other dataset pairs using our method are compared with a published alternative method, metabCombiner, showing the advantages of our approach"
- [methods] Role of metabCombiner in the comparison: "These are used to compare the results of M2S and an existing method, metabCombiner."
- [abstract] Tool availability in MATLAB environment: "The method can be applied using M2S (Match 2 Sets), a free, open-source MATLAB toolbox"
- [intro] metabCombiner modeling of RT shift: "The software “metabCombiner”9 robustly models the retention time shift between two feature lists"
- [methods] Feature representation used as benchmark input: "the medians of RT, m/z, and FI across all samples were used."
