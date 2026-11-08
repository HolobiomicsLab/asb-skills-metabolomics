---
name: randomization-output-tabular-aggregation
description: Use when you have a randomized sample injection sequence with categorical
  dimensions (e.g. treatment, gender, age group) and need to quantify consecutive
  same-category runs to validate randomization behavior against published expectations.
when_to_use_negative:
- The sequence was produced by block randomization with a small block size — the paper
  shows block size 2 yields no three consecutive samples of the same group, so run-tallying
  trivially finds no aggregations and the diagnostic is uninformative
- The sequence is unordered (e.g. a plate position layout with no injection order),
  since run detection depends on consecutive position in the sequence
- Samples carry no categorical dimensions to aggregate over — complete randomization
  does not stratify, but run tallying still needs at least one labeled dimension
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3570
tools:
- name: InjectionDesign
  role: Provides the complete randomization algorithm whose output sequence is scanned
    for aggregated feature occurrences; the algorithm does not stratify by any dimension
    and shuffles the entire sample set at random
  repo: https://github.com/CSi-Studio/InjectionDesign
- name: Excel
  role: Used to import the sample table and to compile/inspect the tallied aggregation
    counts CSV
- name: Java (JDK 17)
  role: Runtime for running InjectionDesign locally to generate the randomized sequence
    prior to aggregation tallying
  repo: https://github.com/CSi-Studio/InjectionDesign
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2023.02.26.530140
    title: 'InjectionDesign: Plate Design with Optimized Stratified Block Randomization
      for Modern LC/GC-MS-Based Sample Preparation'
  source_content_hash: sha256:6399dbea8baf5ac6a7016580c2c4daa56ed4632a8ebff26d3c3c3a4a1167329c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/InjectionDesign
  tools:
  - Excel
  - InjectionDesign
  - Java
  - Java (JDK 17)
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3570
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# randomization-output-tabular-aggregation

## Summary

Scans a randomized injection sequence for aggregated feature occurrences — runs of consecutive samples sharing the same category in a stratification dimension — and tallies these events per category per dimension into a CSV table, to quantify whether a randomization algorithm (e.g. complete randomization) produces clustered regions.

## When to use

Use when you have a randomized sample injection sequence with categorical dimensions (e.g. treatment, gender, age group) and need to quantify consecutive same-category runs to validate randomization behavior against published expectations.

## When NOT to use

- The sequence was produced by block randomization with a small block size — the paper shows block size 2 yields no three consecutive samples of the same group, so run-tallying trivially finds no aggregations and the diagnostic is uninformative
- The sequence is unordered (e.g. a plate position layout with no injection order), since run detection depends on consecutive position in the sequence
- Samples carry no categorical dimensions to aggregate over — complete randomization does not stratify, but run tallying still needs at least one labeled dimension

## Inputs

- Randomized sample injection sequence (ordered list of samples with categorical annotations, e.g. 1300-sample cohort table with treatment/gender/age-group labels)
- Definition of stratification dimensions and their category levels (e.g. two-category treatment, three-category gender, four-category age group)

## Outputs

- CSV table of aggregated feature-occurrence counts per category per dimension (e.g. complete_randomization_feature_aggregation.csv)

## How to apply

Take the randomized sequence produced by an algorithm such as InjectionDesign's complete randomization, which does not stratify by any dimension and shuffles the entire sample set at random. For each stratification dimension (in the paper's case study: two-category treatment, three-category gender, four-category age group over a 1300-sample cohort), scan the ordered sequence for runs of consecutive samples sharing the same category. Tally each aggregation event per category per dimension — e.g. how many runs of 3+ consecutive Female samples appear in the gender dimension. Write the tallied counts to a CSV such as complete_randomization_feature_aggregation.csv. The rationale is diagnostic: the paper reports that complete randomization generates a large number of aggregated features with high probability (the clustered regions highlighted in Figure 5b), in contrast to the no-three-consecutive-same-group control achievable by block randomization with block size 2, so the tally table reveals whether the algorithm averages out or preserves natural clustering in the sample feature distribution.

## Related tools

- **InjectionDesign** (Provides the complete randomization algorithm whose output sequence is scanned for aggregated feature occurrences; the algorithm does not stratify by any dimension and shuffles the entire sample set at random) — https://github.com/CSi-Studio/InjectionDesign
- **Excel** (Used to import the sample table and to compile/inspect the tallied aggregation counts CSV)
- **Java (JDK 17)** (Runtime for running InjectionDesign locally to generate the randomized sequence prior to aggregation tallying) — https://github.com/CSi-Studio/InjectionDesign

## Examples

```
java -jar /YOUR_PATH/InjectionDesignDeploy.jar  # generate complete-randomization sequence, then scan it and write the per-category run tallies to complete_randomization_feature_aggregation.csv
```

## Evaluation signals

- The aggregation table should exhibit the behavior the paper reports for complete randomization: a large number of aggregated features with high probability, corresponding to the red-boxed clustered regions highlighted in Figure 5b
- As a negative control, the same tally on a block-randomized sequence with block size 2 should show no three consecutive samples of the same group in any dimension
- Every category level in each dimension must appear in the tally (e.g. all 2 treatment, 3 gender, 4 age-group categories), with counts consistent with the input cohort composition (e.g. 615 Male / 434 Female / 251 Unknown)
- Output CSV must parse cleanly and contain one row per category-per-dimension with integer run counts
- Re-running the tally on the same sequence must reproduce identical counts (deterministic post-processing of a fixed input)

## Limitations

- Complete randomization is not a perfect solution for scenarios where block randomization loses the natural numerical characteristics of the sample feature distribution — the aggregation table will show many clustered runs, but this reflects the algorithm's known inadequacy rather than a processing error
- The tally is sensitive to the chosen run-length threshold; the paper's contrast is anchored on runs of three consecutive same-group samples
- With too many groups in one dimension, even optimized block randomization can produce local aggregation under special numerical distributions, so a non-zero tally does not by itself indicate complete randomization was used

## Evidence

- [results] the complete randomization algorithm generates a large number of aggregated features with high probability: [evidence span withheld — anchor status: not_found]
- [discussion] complete randomization not a perfect solution: "Although the complete randomization algorithm provided by InjectionDesign can be used to some extent for such scenarios, it is not a perfect solution."
- [discussion] block randomization loses natural characteristics: "the block randomization algorithm completely averages out the distribution of the sample features, thus losing some of the numerical characteristics that are naturally present in the sample"
- [methods] no three consecutive same group with block size 2: "since the block size is 2, theoretically there were no three consecutive samples of the same group in the results."
- [discussion] local aggregation under special numerical distributions: "When there are too many groups in one dimension. Due to the averaging step calculation method, which can lead to local aggregation under some special numerical distributions."
