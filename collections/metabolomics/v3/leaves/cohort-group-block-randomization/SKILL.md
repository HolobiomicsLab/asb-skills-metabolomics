---
name: cohort-group-block-randomization
description: Use when assembling intra-batch injection-order sequences for a batch
  whose cohort assignment is already balanced across groups, and the randomized sequence
  must avoid three consecutive samples from the same group while retaining local randomness.
when_to_use_negative:
- Block randomization completely averages out the sample feature distribution, so
  it is the wrong choice when the natural numerical characteristics of the sample
  distribution must be preserved; the article notes complete randomization is also
  'not a perfect solution' for such scenarios.
- Avoid when a dimension has too many groups under special numerical distributions,
  since the optimized algorithm's averaging step can produce local aggregation.
- Do not use for inter-batch balancing of covariates (age, gender) across batches
  — that is a separate pre-step, not intra-batch randomization.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: InjectionDesign
  role: Free web service implementing the intra-batch block randomization and optimized
    block randomization algorithms and visualizing the final distribution sequence
    in real time
  repo: https://github.com/CSi-Studio/InjectionDesign
- name: Java (JDK 17)
  role: Runtime for the locally deployed InjectionDesign service (java -jar InjectionDesignDeploy.jar)
    that executes the randomization algorithms
  repo: https://github.com/CSi-Studio/InjectionDesign
- name: Excel
  role: Template-based import of the sample list used as randomization input and export
    of the final randomized worksheet
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2023.02.26.530140
    title: 'InjectionDesign: Plate Design with Optimized Stratified Block Randomization
      for Modern LC/GC-MS-Based Sample Preparation'
  source_content_hash: sha256:e99360f993d6a9e4fb161ff3e1f49295a2a0c7ff045a5c414434c4c3a0e7804f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/InjectionDesign
  tools:
  - Excel
  - InjectionDesign
  - Java (JDK 17)
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cohort-group-block-randomization

## Summary

Intra-batch randomization of sample injection order using block randomization (block size 2) for balanced treatment/placebo dimensions and an optimized block randomization (ascending sort, max-dimension vertical grouping, block randomization per group, uniform-step interleaving) for unbalanced dimensions, preventing runs of three consecutive same-group samples in LC/GC-MS sequences.

## When to use

Use when assembling intra-batch injection-order sequences for a batch whose cohort assignment is already balanced across groups, and the randomized sequence must avoid three consecutive samples from the same group while retaining local randomness.

## When NOT to use

- Block randomization completely averages out the sample feature distribution, so it is the wrong choice when the natural numerical characteristics of the sample distribution must be preserved; the article notes complete randomization is also 'not a perfect solution' for such scenarios.
- Avoid when a dimension has too many groups under special numerical distributions, since the optimized algorithm's averaging step can produce local aggregation.
- Do not use for inter-batch balancing of covariates (age, gender) across batches — that is a separate pre-step, not intra-batch randomization.

## Inputs

- Balanced cohort assignment table from inter-batch balancing (task_001 output), e.g. Excel-imported sample list with treatment/placebo and other classification dimensions
- Randomization dimension definitions per batch

## Outputs

- Final intra-batch randomized injection-order sequence for each batch
- Scan record of positions where three consecutive samples share the same treatment/placebo group (expected: zero violations)

## How to apply

Load the balanced cohort assignment table produced by inter-batch balancing. For the treatment/placebo dimension, apply block randomization with block size 2: enumerate the 2-sample block permutations (AB and BA) and use a random number sequence to select block order per the Intra-Batch Randomization Algorithm for Uniformly Distributed Samples. For any remaining unbalanced dimensions, apply the optimized block randomization algorithm: ascending sort of samples, grouping along the max dimension, block randomization within each group, then uniform-step interleaving of adjacent groups. Assemble the final intra-batch randomized injection-order sequence per batch, then scan the sequence for positions where three consecutive samples share the same group; with block size 2 the article reports zero such positions, and even sample distribution with retained local randomness.

## Related tools

- **InjectionDesign** (Free web service implementing the intra-batch block randomization and optimized block randomization algorithms and visualizing the final distribution sequence in real time) — https://github.com/CSi-Studio/InjectionDesign
- **Java (JDK 17)** (Runtime for the locally deployed InjectionDesign service (java -jar InjectionDesignDeploy.jar) that executes the randomization algorithms) — https://github.com/CSi-Studio/InjectionDesign
- **Excel** (Template-based import of the sample list used as randomization input and export of the final randomized worksheet)

## Examples

```
java -jar /YOUR_PATH/InjectionDesignDeploy.jar  # then at http://localhost:8080: upload sample Excel, define randomization dimension (treatment/placebo), run block randomization, Export All worksheets
```

## Evaluation signals

- Zero positions in the randomized sequence where three consecutive samples belong to the same treatment/placebo group (theoretical guarantee since block size is 2).
- Sample group distribution across the sequence remains even (balanced) after randomization.
- Sequence retains local randomness rather than a deterministic pattern.
- Each batch's sequence is reproducible/traceable and visualized in the tool before worksheet download.

## Limitations

- Block randomization completely averages out the distribution of sample features, losing natural numerical characteristics present in the sample.
- With too many groups in one dimension, the averaging step of the optimized algorithm can cause local aggregation under some special numerical distributions.
- Complete randomization is not a perfect substitute for scenarios where block randomization loses numerical characteristics.

## Evidence

- [methods] block randomization with block size 2 avoids three consecutive same-group samples: "since the block size is 2, theoretically there were no three consecutive samples of the same group in the results."
- [intro] optimized block randomization for dispersed/unequally distributed samples: "This work presents an optimized block randomization approach for handling dispersed samples that are out of balance."
- [intro] randomization as QC step to reduce drift and batch effect: "Randomizing the pretreated samples according to appropriate dimensions is an important method in the QC process for reducing machine drift and batch effect"
- [readme] README: define balance and randomization dimensions separately with real-time visualized sequence: [evidence span withheld — anchor status: external_doc]
