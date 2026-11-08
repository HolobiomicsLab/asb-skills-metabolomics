---
name: batch-effect-qc-sequence-design
description: Use when block randomization averages out natural numerical characteristics
  of sample feature distributions (e.g. gender or age-group counts) and you need to
  quantify the resulting aggregation/clustering behavior in the injection sequence
  as a baseline comparison.
when_to_use_negative:
- When the goal is to minimize confounding-variable interference in the injection
  sequence — stratified/optimized block randomization is the intended algorithm; complete
  randomization is only a baseline for scenarios where block randomization averages
  out natural feature distribution
- When samples are unequally distributed and need balanced batches — the inter-batch
  balancing algorithm, not complete randomization, addresses that
tools:
- name: InjectionDesign
  role: Provides the complete randomization algorithm for intra-batch sample sequence
    shuffling without stratification; free web service or local Java deployment
  repo: https://github.com/CSi-Studio/InjectionDesign
- name: Excel
  role: Import sample basic information via the downloadable template; alternatively
    used to process/export aggregation tallies
- name: Java (JDK 17)
  role: Runtime for local deployment of the InjectionDesign service used to run the
    randomization
  repo: https://github.com/CSi-Studio/InjectionDesign
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2023.02.26.530140
    title: 'InjectionDesign: Plate Design with Optimized Stratified Block Randomization
      for Modern LC/GC-MS-Based Sample Preparation'
  source_content_hash: sha256:af4e0863182b32a71299376d0d054453fd711c15b31b782087aa836de90a3f33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/InjectionDesign
  tools:
  - Excel
  - InjectionDesign
  - Java
  - Java (JDK 17)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# batch-effect-qc-sequence-design

## Summary

Establishes a complete-randomization baseline for LC/GC-MS injection-sequence QC: run InjectionDesign's complete randomization algorithm (no stratification, whole-sample shuffle) across all stratification dimensions and tally aggregated feature occurrences to quantify what block randomization averages out.

## When to use

Use when block randomization averages out natural numerical characteristics of sample feature distributions (e.g. gender or age-group counts) and you need to quantify the resulting aggregation/clustering behavior in the injection sequence as a baseline comparison.

## When NOT to use

- When the goal is to minimize confounding-variable interference in the injection sequence — stratified/optimized block randomization is the intended algorithm; complete randomization is only a baseline for scenarios where block randomization averages out natural feature distribution
- When samples are unequally distributed and need balanced batches — the inter-batch balancing algorithm, not complete randomization, addresses that

## Inputs

- Cohort/sample table (e.g. chained cohort_table CSV) with categorical metadata per sample: treatment arm, gender, age group
- InjectionDesign web service (http://injection.design, admin/admin123 demo) or local deployment with samples imported via Excel template

## Outputs

- Randomized injection sequence per batch lacking any stratification constraint
- Aggregated-feature-occurrence tally per category per dimension (e.g. complete_randomization_feature_aggregation.csv)

## How to apply

Load the cohort table (e.g. a 1300-sample cohort with treatment/placebo, gender categories such as 615 Male / 434 Female / 251 Unknown, and age groups such as 169 Kids / 351 Teenager / 481 Middle-Aged / 299 Older) and run InjectionDesign's complete randomization algorithm on the intra-batch samples — this algorithm does not stratify by any dimension but shuffles the entire sample set at random, and is offered for unidentified interferences or simulating the natural sample distribution. Apply the randomization simultaneously across all dimensions of interest (e.g. two-category treatment, three-category gender, four-category age group). Then scan each randomized sequence for aggregated feature occurrences — runs of consecutive samples sharing the same category in a dimension — and tally these aggregation events per category per dimension. Export the tallied counts (e.g. to complete_randomization_feature_aggregation.csv) and compare against the block-randomization control, where block size 2 theoretically yields no three consecutive samples of the same group. The expected validation pattern is that complete randomization generates a large number of aggregated features with high probability, corresponding to the clustered regions highlighted in Figure 5b of the paper.

## Related tools

- **InjectionDesign** (Provides the complete randomization algorithm for intra-batch sample sequence shuffling without stratification; free web service or local Java deployment) — https://github.com/CSi-Studio/InjectionDesign
- **Excel** (Import sample basic information via the downloadable template; alternatively used to process/export aggregation tallies)
- **Java (JDK 17)** (Runtime for local deployment of the InjectionDesign service used to run the randomization) — https://github.com/CSi-Studio/InjectionDesign

## Examples

```
java -jar /YOUR_PAHT/InjectionDesignDeploy.jar  # then at http://localhost:8080 run complete randomization on the intra-batch samples and export the sequence worksheet
```

## Evaluation signals

- Aggregation table shows a large number of aggregated features with high probability across all three dimensions, matching the clustered regions highlighted in Figure 5b of the paper
- Category-level tallies are consistent with the input cohort marginals (e.g. gender totals 615/434/251 and age-group totals 169/351/481/299 are preserved after shuffling)
- Contrast check: the block-randomization control with block size 2 produces no three consecutive samples of the same group in the treatment/placebo dimension, whereas the complete-randomization sequence does contain such runs
- Randomization is truly unstratified — no dimension was used as a balancing key when generating the sequence
- Output CSV schema contains one aggregation count per category per dimension

## Limitations

- The complete randomization algorithm is not a perfect solution for scenarios where block randomization loses the natural numerical characteristics of the sample distribution
- Block randomization itself completely averages out the distribution of sample features, so neither algorithm alone satisfies both goals simultaneously
- The optimized block randomization can lead to local aggregation when there are too many groups in one dimension under some special numerical distributions

## Evidence

- [discussion] complete randomization as baseline: "Although the complete randomization algorithm provided by InjectionDesign can be used to some extent for such scenarios, it is not a perfect solution."
- [discussion] block randomization averaging: "the block randomization algorithm completely averages out the distribution of the sample features, thus losing some of the numerical characteristics that are naturally present in the sample"
- [methods] block size 2 control: "since the block size is 2, theoretically there were no three consecutive samples of the same group in the results."
- [other] complete randomization purpose: [evidence span withheld — anchor status: not_found]
- [readme] workflow step in README: [evidence span withheld — anchor status: external_doc]
