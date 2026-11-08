---
name: cohort-randomization-sequence-generation
description: Use when a cohort table carries three or fewer classification dimensions
  (e.g. two-category treatment, three-category gender, four-category age group) and
  the goal is an unstratified complete-randomization injection order whose natural
  feature clustering (e.
when_to_use_negative:
- The experiment requires suppression of batch/drift bias and confounder balancing
  — use stratified or optimized block randomization instead, since block randomization
  is what guarantees no long same-group runs (block size 2 gives no three consecutive
  samples of the same group).
- More than three classification dimensions must be visualized/managed — InjectionDesign
  supports visual presentation of up to three classification dimensions.
- The goal is a plate layout (position design) with QC placement rules (Blank/Solvent/Pooled/LTR
  QC areas) rather than an injection-order randomization outcome.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: InjectionDesign
  role: Web service (or locally deployed instance) that imports the cohort table and
    runs the complete randomization algorithm, displaying the randomized sequence
    in real time
  repo: https://github.com/CSi-Studio/InjectionDesign
- name: Java (JDK 17)
  role: Runtime for launching the locally deployed InjectionDesign jar when the security-sensitive
    lab avoids the public web service
  repo: https://github.com/CSi-Studio/InjectionDesign
- name: Excel
  role: 'Template format used to import the cohort sample list (dimensions: gender,
    treatment, age group) into the project'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2023.02.26.530140
    title: 'InjectionDesign: Plate Design with Optimized Stratified Block Randomization
      for Modern LC/GC-MS-Based Sample Preparation'
  source_content_hash: sha256:a77928a13b54b83d0f62b3ffd1095ffedddb438da032d887b87100a6622e5148
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/InjectionDesign
  tools:
  - Excel
  - InjectionDesign
  - Java
  - Java (JDK 17)
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cohort-randomization-sequence-generation

## Summary

Runs InjectionDesign's complete randomization algorithm on an intra-batch sample set without stratifying on any dimension, producing a shuffled injection sequence for LC/GC-MS sample preparation when block randomization would average out natural feature distributions, then audits the sequence for aggregated feature occurrences (consecutive same-category runs).

## When to use

Use when a cohort table carries three or fewer classification dimensions (e.g. two-category treatment, three-category gender, four-category age group) and the goal is an unstratified complete-randomization injection order whose natural feature clustering (e.g. runs of consecutive same-group samples, as in Figure 5b) must be preserved or quantified rather than suppressed by block randomization.

## When NOT to use

- The experiment requires suppression of batch/drift bias and confounder balancing — use stratified or optimized block randomization instead, since block randomization is what guarantees no long same-group runs (block size 2 gives no three consecutive samples of the same group).
- More than three classification dimensions must be visualized/managed — InjectionDesign supports visual presentation of up to three classification dimensions.
- The goal is a plate layout (position design) with QC placement rules (Blank/Solvent/Pooled/LTR QC areas) rather than an injection-order randomization outcome.

## Inputs

- Cohort sample table (Excel template or imported project) with up to three classification dimensions, e.g. gender (Male/Female/Unknown), treatment (treatment/placebo), age group (Kids/Teenager/Middle-Aged/Older)
- InjectionDesign project with sample list uploaded via 'Download Sample Excel' template
- Optional block-randomized reference sequence (block size 2) for contrast

## Outputs

- Randomized intra-batch injection sequence over the full unstratified sample set
- CSV of aggregated feature-occurrence tallies per category per dimension (e.g. complete_randomization_feature_aggregation.csv)
- Visualized distribution sequence displayed in real time by the web service

## How to apply

Load the cohort sample table (e.g. the 1300-sample table with gender 615 Male/434 Female/251 Unknown, treatment 650/650, and age group 169 Kids/351 Teenager/481 Middle-Aged/299 Older), importing it via Excel into an InjectionDesign project. In the web service (or locally deployed instance), define the balance dimension and the randomization dimension separately, but for complete randomization do NOT stratify: the algorithm shuffles the entire intra-batch sample set at random across all dimensions simultaneously. After generating the sequence, scan it per dimension per category for aggregated feature occurrences — runs of consecutive samples sharing the same category — and tally these events. Write the tallies to a CSV (e.g. complete_randomization_feature_aggregation.csv) and compare against the block-randomization control, where block size 2 yields no three consecutive samples of the same group. Note that complete randomization is expected to produce a large number of aggregated runs with high probability; this is the paper's reported behavior, not a failure.

## Related tools

- **InjectionDesign** (Web service (or locally deployed instance) that imports the cohort table and runs the complete randomization algorithm, displaying the randomized sequence in real time) — https://github.com/CSi-Studio/InjectionDesign
- **Java (JDK 17)** (Runtime for launching the locally deployed InjectionDesign jar when the security-sensitive lab avoids the public web service) — https://github.com/CSi-Studio/InjectionDesign
- **Excel** (Template format used to import the cohort sample list (dimensions: gender, treatment, age group) into the project)

## Examples

```
java -jar /YOUR_PATH/InjectionDesignDeploy.jar
```

## Evaluation signals

- The aggregation CSV shows a large number of aggregated feature occurrences (runs of ≥3 consecutive same-category samples) per category per dimension, matching the paper's report for complete randomization (red-boxed clustered regions in Figure 5b).
- Contrast check: a block-randomized sequence with block size 2 on the same cohort should show no three consecutive samples of the same group, while the complete-randomization sequence should violate this frequently.
- Every sample in the input cohort (e.g. all 1300) appears exactly once in the randomized sequence — the shuffle must be a permutation with no drops or duplicates.
- Tallies cover all categories of all dimensions simultaneously (2 treatment × 3 gender × 4 age group), with counts per category consistent with scanning the same single sequence.
- The randomized sequence is displayed/validated in the web service's real-time visualization before worksheet export.

## Limitations

- Complete randomization is not a perfect solution for scenarios where block randomization averages out natural sample-feature numerical characteristics — the authors state it can be used 'to some extent' only.
- Because no stratification is applied, the sequence contains many clustered same-group runs, which is precisely the batch-effect/drift pattern the QC process otherwise tries to avoid.
- Even optimized block randomization can lead to local aggregation when there are too many groups in one dimension under some special numerical distributions, so no algorithm here fully resolves clustering in extreme distributions.
- The web service may not yet support newly emerging QC sample types and injection rules.

## Evidence

- [discussion] Complete randomization rationale: "the block randomization algorithm completely averages out the distribution of the sample features, thus losing some of the numerical characteristics that are naturally present in the sample"
- [discussion] Complete randomization adequacy: "Although the complete randomization algorithm provided by InjectionDesign can be used to some extent for such scenarios, it is not a perfect solution."
- [methods] Block randomization control expectation: "since the block size is 2, theoretically there were no three consecutive samples of the same group in the results."
- [intro] Why randomization matters for QC: "Randomizing the pretreated samples according to appropriate dimensions is an important method in the QC process for reducing machine drift and batch effect"
- [readme] Dimension configuration in the service: [evidence span withheld — anchor status: external_doc]
- [readme] Dimension-count limit: "InjectionDesign supports visual presentation of up to three classification dimensions."
