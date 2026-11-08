---
name: sample-sequence-distribution-verification
description: Use when an intra-batch randomized injection sequence has just been generated
  (block randomization or optimized block randomization) and must be checked for consecutive
  same-group runs and even group distribution before LC/GC-MS injection.
when_to_use_negative:
- The sequence was produced by a completely random algorithm rather than block randomization
  — the paper notes complete randomization 'is not a perfect solution' and triple-run
  counts are not theoretically bounded to zero, so the zero-violation criterion does
  not apply
- The samples are still unbalanced across batches (inter-batch balancing on dimensions
  like age and gender has not been done), since this skill assumes the balanced assignment
  table from task-level balancing output
- Optimized block randomization was applied to a dimension with too many groups, where
  the paper warns local aggregation can occur under some special numerical distributions
  and the zero-triple expectation may not hold
tools:
- name: InjectionDesign
  role: Generates the intra-batch randomized injection-order sequence (block size
    2 permutations AB/BA, optimized block randomization) that is then scanned for
    triple same-group runs
  repo: https://github.com/CSi-Studio/InjectionDesign
- name: Java (JDK 17)
  role: Runtime for the locally installed InjectionDesign jar whose output sequences
    are validated
  repo: https://github.com/CSi-Studio/InjectionDesign
- name: Excel
  role: Format used to import the sample/cohort table and to export the final worksheet
    whose injection sequence is verified
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2023.02.26.530140
    title: 'InjectionDesign: Plate Design with Optimized Stratified Block Randomization
      for Modern LC/GC-MS-Based Sample Preparation'
  source_content_hash: sha256:ffac5964128964da978ae47f4a569cc413eca8c2725aa9a7a63e5693a49b5c64
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/InjectionDesign
  tools:
  - Excel
  - InjectionDesign
  - Java (JDK 17)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# sample-sequence-distribution-verification

## Summary

Validates an intra-batch randomized injection-order sequence (e.g., from block randomization with block size 2 in InjectionDesign) by scanning for runs of three consecutive samples from the same treatment/placebo group and confirming the sample distribution remains even.

## When to use

Use when an intra-batch randomized injection sequence has just been generated (block randomization or optimized block randomization) and must be checked for consecutive same-group runs and even group distribution before LC/GC-MS injection.

## When NOT to use

- The sequence was produced by a completely random algorithm rather than block randomization — the paper notes complete randomization 'is not a perfect solution' and triple-run counts are not theoretically bounded to zero, so the zero-violation criterion does not apply
- The samples are still unbalanced across batches (inter-batch balancing on dimensions like age and gender has not been done), since this skill assumes the balanced assignment table from task-level balancing output
- Optimized block randomization was applied to a dimension with too many groups, where the paper warns local aggregation can occur under some special numerical distributions and the zero-triple expectation may not hold

## Inputs

- Balanced cohort assignment table from the inter-batch balancing output (importable via Excel)
- Treatment/placebo group labels for each sample (classification dimension used for randomization)
- Per-batch intra-batch randomized injection-order sequence to be validated

## Outputs

- Count and positions of occurrences where three consecutive samples share the same treatment/placebo group (expected: zero for block size 2)
- Per-batch validation verdict that the randomized sequence has no triple same-group runs while retaining even sample distribution and local randomness

## How to apply

Load the balanced cohort assignment table produced by the inter-batch balancing step (Excel import is supported by InjectionDesign). Apply the intra-batch block randomization algorithm with block size 2, enumerating the 2-sample block permutations (AB and BA) and using a random number sequence to select block order per the Intra-Batch Randomization Algorithm for Uniformly Distributed Samples; for unbalanced remaining dimensions use the optimized block randomization (ascending sort, max-dimension vertical grouping, block randomization per group, uniform-step interleaving of adjacent groups). Assemble the final injection-order sequence for each batch, then scan every position i where samples i, i-1, and i-2 share the same treatment/placebo group and record any such position. Because the block size is 2, the theoretical expectation is zero positions with three consecutive samples of the same group. Also confirm the group distribution remains even while local randomness is retained in the sequence.

## Related tools

- **InjectionDesign** (Generates the intra-batch randomized injection-order sequence (block size 2 permutations AB/BA, optimized block randomization) that is then scanned for triple same-group runs) — https://github.com/CSi-Studio/InjectionDesign
- **Java (JDK 17)** (Runtime for the locally installed InjectionDesign jar whose output sequences are validated) — https://github.com/CSi-Studio/InjectionDesign
- **Excel** (Format used to import the sample/cohort table and to export the final worksheet whose injection sequence is verified)

## Examples

```
python -c "s=['A','B','A','B']; bad=[i for i in range(2,len(s)) if s[i]==s[i-1]==s[i-2]]; print('violations:',len(bad))"
```

## Evaluation signals

- Zero positions i exist where sequence[i], sequence[i-1], and sequence[i-2] share the same treatment/placebo group (theoretical guarantee for block size 2)
- The group distribution in the randomized sequence remains even (balanced counts per group) after randomization
- The sequence retains local randomness (adjacent blocks AB/BA appear in non-deterministic order rather than a fixed repeating pattern)
- Every batch produced by the inter-batch balancing step has its final injection-order sequence scanned, not just a subset
- The downloaded/exported worksheet sequence matches the validated visualized distribution sequence shown by the tool

## Limitations

- The zero-triple-run guarantee holds only because the block size is 2; larger or unbalanced block structures do not carry the same theoretical guarantee
- Block randomization completely averages out the distribution of sample features, losing numerical characteristics naturally present in the sample distribution
- Optimized block randomization can lead to local aggregation when there are too many groups in one dimension under some special numerical distributions

## Evidence

- [methods] Zero triple-run guarantee for block size 2: "since the block size is 2, theoretically there were no three consecutive samples of the same group in the results."
- [methods] Optimized block randomization for unbalanced samples: "This work presents an optimized block randomization approach for handling dispersed samples that are out of balance."
- [intro] Randomization as a QC step against drift and batch effect: "Randomizing the pretreated samples according to appropriate dimensions is an important method in the QC process for reducing machine drift and batch effect"
- [readme] Separate balance and randomization dimensions in the tool: [evidence span withheld — anchor status: external_doc]
- [discussion] Block randomization loses natural numerical characteristics: "the block randomization algorithm completely averages out the distribution of the sample features, thus losing some of the numerical characteristics that are naturally present in the sample"
