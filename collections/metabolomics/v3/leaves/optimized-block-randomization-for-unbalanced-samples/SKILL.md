---
name: optimized-block-randomization-for-unbalanced-samples
description: Use when balancing has left one or more randomization dimensions with
  unequal group counts (e.g. treatment vs placebo) and the final intra-batch injection
  sequence must still avoid consecutive same-group runs.
when_to_use_negative:
- The randomization dimension is already perfectly balanced, so plain intra-batch
  block randomization (e.g. block size 2 with AB/BA permutations) suffices without
  the sorting/grouping/interleaving steps
- The natural numerical distribution of sample features must be preserved in the sequence
  — block randomization completely averages out the distribution of sample features
- Samples belong to multi-stage long-term projects where only persistent storage and
  traceability of injection information (not sequence randomization) is required
edam_topics:
- http://edamontology.org/topic_3570
tools:
- name: InjectionDesign
  role: Web service implementing the optimized block randomization for unequally distributed
    samples and visualizing the resulting intra-batch distribution sequence
  repo: https://github.com/CSi-Studio/InjectionDesign
- name: Java (JDK 17)
  role: Runtime for the local InjectionDesign deployment that executes the randomization
    algorithm
- name: Excel
  role: Import of sample lists into the project before balancing and randomization
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2023.02.26.530140
    title: 'InjectionDesign: Plate Design with Optimized Stratified Block Randomization
      for Modern LC/GC-MS-Based Sample Preparation'
  source_content_hash: sha256:08d808d69444867c5cbce55a0c11220612fa1df516f888edda96686cb2e037a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/InjectionDesign
  tools:
  - Excel
  - InjectionDesign
  - Java (JDK 17)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3570
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# optimized-block-randomization-for-unbalanced-samples

## Summary

Randomizes the intra-batch injection order of samples whose groups are unevenly distributed by ascending-sorting, grouping into max/min dimensions, block-randomizing each vertical group, and interleaving adjacent groups with uniform steps — an optimized block randomization from the InjectionDesign service for LC/GC-MS plate/sequence design.

## When to use

Use when balancing has left one or more randomization dimensions with unequal group counts (e.g. treatment vs placebo) and the final intra-batch injection sequence must still avoid consecutive same-group runs.

## When NOT to use

- The randomization dimension is already perfectly balanced, so plain intra-batch block randomization (e.g. block size 2 with AB/BA permutations) suffices without the sorting/grouping/interleaving steps
- The natural numerical distribution of sample features must be preserved in the sequence — block randomization completely averages out the distribution of sample features
- Samples belong to multi-stage long-term projects where only persistent storage and traceability of injection information (not sequence randomization) is required

## Inputs

- Balanced cohort/sample assignment table from inter-batch balancing (per-batch sample list with treatment/placebo group labels)
- Sample metadata with one or more randomization dimensions (e.g. treatment/placebo, age, gender)
- Excel-imported sample list with basic sample information (InjectionDesign template)

## Outputs

- Final intra-batch randomized injection-order sequence for each batch
- Visualized distribution sequence of group assignments along the injection order
- Run-length audit of consecutive same-group samples in the randomized sequence

## How to apply

Start from the balanced cohort table produced by inter-batch balancing. For each unbalanced dimension, ascending-sort the samples by the group identifier, count groups in the max dimension versus the min dimension, and group samples vertically by the max dimension so each vertical group is as even as possible. Apply block randomization independently to each vertical group (for block size 2, enumerate AB and BA permutations and draw a random number sequence to select block order). Finally, interleave the block-randomized adjacent groups with a uniform-step (averaging) calculation to assemble the intra-batch injection sequence. Validate by scanning the resulting sequence for runs of ≥3 consecutive same-group samples and confirming the group counts remain even while local randomness is preserved.

## Related tools

- **InjectionDesign** (Web service implementing the optimized block randomization for unequally distributed samples and visualizing the resulting intra-batch distribution sequence) — https://github.com/CSi-Studio/InjectionDesign
- **Java (JDK 17)** (Runtime for the local InjectionDesign deployment that executes the randomization algorithm)
- **Excel** (Import of sample lists into the project before balancing and randomization)

## Examples

```
java -jar /YOUR_PATH/InjectionDesignDeploy.jar   # then in the web UI: upload samples Excel, set randomization dimension, download worksheet
```

## Evaluation signals

- Scan the final intra-batch sequence: with block size 2, there should be zero positions where three consecutive samples share the same group (article criterion: 'theoretically there were no three consecutive samples of the same group in the results')
- Per-group counts in the randomized sequence should remain even/uneven as expected from the input (optimized algorithm is applied for unequally distributed samples, so no forced balancing is introduced)
- Visualized distribution sequence should show no long local aggregation of the same group beyond what the uniform-step interleaving produces
- All samples from the balanced cohort table must appear exactly once in the output sequence (no dropped or duplicated positions)
- Real-time visualization in InjectionDesign must match the final downloaded worksheet sequence

## Limitations

- Block randomization completely averages out the distribution of the sample features, losing some of the numerical characteristics that are naturally present in the sample
- When there are too many groups in one dimension, the averaging step calculation method can lead to local aggregation under some special numerical distributions
- The complete randomization algorithm provided by InjectionDesign is only a partial workaround for scenarios where block randomization loses numerical characteristics
- Randomization cannot compensate for drift arising from omitted critical experimental steps in the sample preparation pipeline

## Evidence

- [intro] Optimized block randomization algorithm for unequally distributed samples: "An optimized block randomization algorithm for unequally distributed samples."
- [methods] No three consecutive same-group samples with block size 2: "since the block size is 2, theoretically there were no three consecutive samples of the same group in the results."
- [discussion] Averaging step causes local aggregation when groups are numerous: "When there are too many groups in one dimension. Due to the averaging step calculation method, which can lead to local aggregation under some special numerical distributions."
- [discussion] Block randomization loses natural numerical characteristics: "the block randomization algorithm completely averages out the distribution of the sample features, thus losing some of the numerical characteristics that are naturally present in the sample"
- [readme] Balance and randomization dimensions defined separately with real-time visualization: [evidence span withheld — anchor status: external_doc]
