---
name: consecutive-run-violation-detection
description: Use when an intra-batch randomized injection-order sequence (e.g., treatment
  vs placebo) has been produced by block randomization with block size 2 and you must
  verify the no-three-consecutive-same-group guarantee before export.
when_to_use_negative:
- The sequence was produced by the complete randomization algorithm rather than block
  randomization — runs of consecutive same-group samples are not precluded there,
  so a zero-violation criterion does not apply.
- The goal is inter-batch balancing of sample characteristics (age, gender, etc.)
  across batches, not checking within-batch injection-order adjacency.
- The randomization dimension is inherently unbalanced and was handled by optimized
  block randomization with many groups in one dimension, where the paper warns local
  aggregation can occur under some special numerical distributions.
tools:
- name: InjectionDesign
  role: Web/plate-design service whose intra-batch block randomization output (randomized
    injection sequence) is the object scanned for three-consecutive-same-group violations
  repo: https://github.com/CSi-Studio/InjectionDesign
- name: Java (JDK 17)
  role: Runtime for deploying InjectionDesign locally (java -jar InjectionDesignDeploy.jar)
    to generate and inspect the randomized sequences
  repo: https://github.com/CSi-Studio/InjectionDesign
- name: Excel
  role: Format used to import the sample list and receive the exported worksheet containing
    the final randomized sequence for validation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2023.02.26.530140
    title: 'InjectionDesign: Plate Design with Optimized Stratified Block Randomization
      for Modern LC/GC-MS-Based Sample Preparation'
  source_content_hash: sha256:c5f1d92b97a1be8b7d794c0dfc11bba44b7bac21c799d43eb0e4132d53a111d2
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
yes_rate: 0.8333333333333334
n_claims: 6
---

# consecutive-run-violation-detection

## Summary

Validates an intra-batch randomized injection sequence by scanning for positions where three consecutive samples share the same treatment/placebo (or other randomization-dimension) group. For block size 2 block randomization, the expected result is zero such runs, while the sample distribution stays even and local randomness is retained.

## When to use

Use when an intra-batch randomized injection-order sequence (e.g., treatment vs placebo) has been produced by block randomization with block size 2 and you must verify the no-three-consecutive-same-group guarantee before export.

## When NOT to use

- The sequence was produced by the complete randomization algorithm rather than block randomization — runs of consecutive same-group samples are not precluded there, so a zero-violation criterion does not apply.
- The goal is inter-batch balancing of sample characteristics (age, gender, etc.) across batches, not checking within-batch injection-order adjacency.
- The randomization dimension is inherently unbalanced and was handled by optimized block randomization with many groups in one dimension, where the paper warns local aggregation can occur under some special numerical distributions.

## Inputs

- Intra-batch randomized injection-order sequence per batch (ordered sample list with treatment/placebo group labels)
- Balanced cohort assignment table from inter-batch balancing (imported via Excel into InjectionDesign)
- Block randomization parameters (block size = 2; block permutations AB and BA)

## Outputs

- Count and list of positions where three consecutive samples share the same group (expected: zero violations)
- Per-batch validation report confirming even group distribution and retained local randomness in the final injection sequence

## How to apply

Take the final intra-batch randomized injection-order sequence assembled after inter-batch balancing (task_001 output) and after applying the Intra-Batch Randomization Algorithm with block size 2 — i.e., enumerating the 2-sample block permutations (AB, BA) and using a random number sequence to select block order. Scan the sequence linearly over all positions in each batch, recording any index i where sample[i], sample[i+1], and sample[i+2] all belong to the same group in the treatment/placebo dimension. For unbalanced remaining dimensions handled by the optimized block randomization (ascending sort, max-dimension vertical grouping, block randomization of each group, uniform-step interleaving of adjacent groups), repeat the scan per dimension. The validation criterion from the paper: because the block size is 2, there should be zero positions with three consecutive samples of the same group, while the group distribution remains even. If violations appear, re-examine the block permutation selection or interleaving step rather than post-hoc shuffling.

## Related tools

- **InjectionDesign** (Web/plate-design service whose intra-batch block randomization output (randomized injection sequence) is the object scanned for three-consecutive-same-group violations) — https://github.com/CSi-Studio/InjectionDesign
- **Java (JDK 17)** (Runtime for deploying InjectionDesign locally (java -jar InjectionDesignDeploy.jar) to generate and inspect the randomized sequences) — https://github.com/CSi-Studio/InjectionDesign
- **Excel** (Format used to import the sample list and receive the exported worksheet containing the final randomized sequence for validation)

## Examples

```
viol = [i for i in range(len(seq)-2) if seq[i]==seq[i+1]==seq[i+2]]; assert not viol  # block size 2 => 0 runs of 3 same-group
```

## Evaluation signals

- Zero positions i where seq[i] == seq[i+1] == seq[i+2] in the treatment/placebo group dimension, for every batch.
- Group sample counts in the randomized sequence remain even (balanced) after randomization.
- Sequence still exhibits local randomness (block order AB/BA chosen by random number sequence), not a deterministic alternation.
- Re-running the block permutation selection and the scan reproduces the zero-violation result; any violation indicates an error in block assembly or interleaving.

## Limitations

- The zero-violation guarantee holds only for block size 2; larger blocks or unbalanced dimensions randomized with the optimized algorithm are not covered by the same theoretical guarantee.
- Block randomization completely averages out the distribution of sample features, losing natural numerical characteristics present in the sample distribution.
- The optimized block randomization can lead to local aggregation when there are too many groups in one dimension under some special numerical distributions; complete randomization is 'not a perfect solution' for such scenarios either.

## Evidence

- [methods] no-three-consecutive guarantee at block size 2: "since the block size is 2, theoretically there were no three consecutive samples of the same group in the results."
- [intro] randomization as QC against drift/batch effect: "Randomizing the pretreated samples according to appropriate dimensions is an important method in the QC process for reducing machine drift and batch effect"
- [methods] optimized block randomization scope: "This work presents an optimized block randomization approach for handling dispersed samples that are out of balance."
- [readme] sequence visualization during randomization (README): [evidence span withheld — anchor status: external_doc]
