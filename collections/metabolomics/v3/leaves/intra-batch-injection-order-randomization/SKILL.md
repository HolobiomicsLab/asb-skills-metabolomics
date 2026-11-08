---
name: intra-batch-injection-order-randomization
description: Use when a batch's samples must be ordered for LC/GC-MS injection such
  that treatment/placebo or other randomized dimensions never cluster (e.g., no three
  consecutive samples of the same group) after inter-batch balancing has been done.
when_to_use_negative:
- When preserving the natural numerical characteristics of a sample feature distribution
  matters more than uniform averaging — block randomization 'completely averages out
  the distribution of the sample features'
- When a dimension has too many groups under special numerical distributions, where
  the optimized block randomization averaging step can cause local aggregation
- When inter-batch balancing across batches (e.g., by age, gender) is the actual goal
  — use the inter-batch balancing skill instead
tools:
- name: InjectionDesign
  role: Web service/tool implementing the intra-batch block and optimized block randomization
    algorithms and visualizing the resulting injection sequence
  repo: https://github.com/CSi-Studio/InjectionDesign
- name: Java (JDK 17)
  role: Runtime for the locally deployed InjectionDesign jar when the web service
    cannot be used
- name: Excel
  role: Sample import template format for delivering sample metadata into InjectionDesign
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2023.02.26.530140
    title: 'InjectionDesign: Plate Design with Optimized Stratified Block Randomization
      for Modern LC/GC-MS-Based Sample Preparation'
  source_content_hash: sha256:4bb6ed8b6c74a19b23840c67472b192c22961adab39fe0ab1f5a21749b795a94
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
yes_rate: 0.9
n_claims: 10
---

# intra-batch-injection-order-randomization

## Summary

Randomizes the injection order of samples within each LC/GC-MS batch using block randomization (balanced dimensions, e.g. block size 2) or optimized block randomization (unequally distributed dimensions), reducing machine drift and batch effect while keeping group distribution even.

## When to use

Use when a batch's samples must be ordered for LC/GC-MS injection such that treatment/placebo or other randomized dimensions never cluster (e.g., no three consecutive samples of the same group) after inter-batch balancing has been done.

## When NOT to use

- When preserving the natural numerical characteristics of a sample feature distribution matters more than uniform averaging — block randomization 'completely averages out the distribution of the sample features'
- When a dimension has too many groups under special numerical distributions, where the optimized block randomization averaging step can cause local aggregation
- When inter-batch balancing across batches (e.g., by age, gender) is the actual goal — use the inter-batch balancing skill instead

## Inputs

- Balanced cohort assignment table produced by the inter-batch balancing step
- Sample list with classification dimensions (e.g., treatment/placebo, age, gender), imported via the Excel template into InjectionDesign
- Randomization dimension definition and block size choice (e.g., block size 2)

## Outputs

- Final intra-batch randomized injection-order sequence for each batch (downloadable worksheet)
- Visualized distribution sequence per randomization adjustment
- Count/positions of violations where three consecutive samples share the same group (expected: zero for block size 2)

## How to apply

First ensure the balanced cohort assignment from inter-batch balancing is loaded, since this skill handles only the within-batch ordering. For each balanced randomization dimension (e.g., treatment/placebo), apply block randomization: enumerate the n-sample block permutations (for block size 2: AB and BA) and use a generated random number sequence to select block order; because block size is 2, no position can have three consecutive samples of the same group. For any remaining unbalanced dimension, apply the optimized block randomization algorithm: ascending sort, max-dimension vertical grouping, block randomization of each group, then uniform-step interleaving of adjacent groups. Assemble the final intra-batch randomized injection-order sequence per batch (InjectionDesign supports separate definition of the balance dimension and the randomization dimension with real-time visualized sequence output). Finally scan the sequence and record any position where three consecutive samples share the same group to validate the constraint.

## Related tools

- **InjectionDesign** (Web service/tool implementing the intra-batch block and optimized block randomization algorithms and visualizing the resulting injection sequence) — https://github.com/CSi-Studio/InjectionDesign
- **Java (JDK 17)** (Runtime for the locally deployed InjectionDesign jar when the web service cannot be used)
- **Excel** (Sample import template format for delivering sample metadata into InjectionDesign)

## Examples

```
java -jar /YOUR_PATH/InjectionDesignDeploy.jar
```

## Evaluation signals

- Zero positions in the output sequence where three consecutive samples belong to the same group when block size is 2
- Sample distribution across the randomization dimension remains even within each batch after randomization
- Randomization dimension was defined separately from the balance dimension in the tool before generating the sequence
- Sequence retains local randomness (block order selected by random number sequence, not a fixed pattern)
- Final sequence is exportable/traceable per batch as a worksheet for multi-week or multi-month projects

## Limitations

- Block randomization completely averages out the distribution of sample features, losing natural numerical characteristics; complete randomization partially compensates but 'is not a perfect solution'
- Optimized block randomization can produce local aggregation when there are too many groups in one dimension under special numerical distributions
- Only the within-batch ordering is addressed; samples must already be evenly distributed across batches by the inter-batch balancing step

## Evidence

- [methods] since the block size is 2, theoretically there were no three consecutive samples of the same group in the results.: "since the block size is 2, theoretically there were no three consecutive samples of the same group in the results."
- [methods] This work presents an optimized block randomization approach for handling dispersed samples that are out of balance.: "This work presents an optimized block randomization approach for handling dispersed samples that are out of balance."
- [intro] Randomizing the pretreated samples according to appropriate dimensions is an important method in the QC process for reducing machine drift and batch effect: "Randomizing the pretreated samples according to appropriate dimensions is an important method in the QC process for reducing machine drift and batch effect"
- [readme] Users need to define the balance dimension and the randomization dimension separately. For each adjustment, the system displays the final visualized distribution sequence in real time.: [evidence span withheld — anchor status: external_doc]
- [discussion] the block randomization algorithm completely averages out the distribution of the sample features, thus losing some of the numerical characteristics that are naturally present in the sample: "the block randomization algorithm completely averages out the distribution of the sample features, thus losing some of the numerical characteristics that are naturally present in the sample"
