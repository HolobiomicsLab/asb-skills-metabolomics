---
name: trailing-sample-random-interspersion
description: Use when assigning a sample list to fixed-capacity plates/batches (e.g.
  96-well plates, max 96 experimental samples/plate) where the sample count is not
  an exact multiple of the batch capacity, and per-batch strata balance must be preserved
  including the remainder.
when_to_use_negative:
- The goal is within-batch run-order or within-plate position design (block randomization,
  QC-sample placement) rather than distributing samples across batches — use the intra-batch
  randomization / intra-batch position design steps instead
- The sample count is an exact multiple of the batch capacity, so there is no trailing
  remainder — ordinary inter-batch balancing without remainder interspersion suffices
- One dimension has too many groups under special numerical distributions, where the
  algorithm's averaging step can cause local aggregation and a different strategy
  is needed
edam_topics:
- http://edamontology.org/topic_3369
tools:
- name: InjectionDesign
  role: Web service (or locally deployed JAR) whose inter-batch balancing algorithm
    performs the trailing-sample randomization and interspersion across batches
  repo: https://github.com/CSi-Studio/InjectionDesign
- name: Excel
  role: Format used to import the sample list (via the downloadable template) carrying
    the stratification fields used for balancing
- name: Java (JDK 17)
  role: Runtime for the locally deployed InjectionDesign service when the web service
    is not used
  repo: https://github.com/CSi-Studio/InjectionDesign
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2023.02.26.530140
    title: 'InjectionDesign: Plate Design with Optimized Stratified Block Randomization
      for Modern LC/GC-MS-Based Sample Preparation'
  source_content_hash: sha256:c0330195a47433ecbc659a5d6f59053ea233a3d5179207107563c947e0e99136
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/InjectionDesign
  tools:
  - Excel
  - InjectionDesign
  - Java (JDK 17)
  - MongoDB
  - Redis
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3369
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Trailing-sample random interspersion

## Summary

A sub-behavior of InjectionDesign's inter-batch balancing algorithm: when the cohort size does not divide evenly by the batch (plate) capacity, the trailing leftover samples are randomized and interspersed across batches rather than being appended as a lopsided final batch, so every plate's group distribution stays close to the overall cohort ratio.

## When to use

Use when assigning a sample list to fixed-capacity plates/batches (e.g. 96-well plates, max 96 experimental samples/plate) where the sample count is not an exact multiple of the batch capacity, and per-batch strata balance must be preserved including the remainder.

## When NOT to use

- The goal is within-batch run-order or within-plate position design (block randomization, QC-sample placement) rather than distributing samples across batches — use the intra-batch randomization / intra-batch position design steps instead
- The sample count is an exact multiple of the batch capacity, so there is no trailing remainder — ordinary inter-batch balancing without remainder interspersion suffices
- One dimension has too many groups under special numerical distributions, where the algorithm's averaging step can cause local aggregation and a different strategy is needed

## Inputs

- Sample list imported via the InjectionDesign Excel template, with classification-dimension fields (e.g. gender, treatment, age group)
- Plate specification (x/y axis numbers and labels, e.g. 12x8 96-well template)
- Maximum experimental sample capacity per plate (excluding QC samples)

## Outputs

- Plate-assignment table (one row per sample with its plate/batch and stratification labels)
- Per-plate group-distribution summary table
- Visualized distribution sequence shown in real time by the web UI

## How to apply

Load the sample list (e.g. the 1300-sample cohort with gender, treatment, and age-group dimensions) into InjectionDesign via the Excel template, then define the balance dimensions (e.g. two-category treatment/placebo, three-category gender, four-category age group) separately from the randomization dimensions. Set the plate specification and the maximum experimental sample capacity per plate (96 for a 12x8 96-well template, excluding QC samples). Run the inter-batch balancing step; when the total count leaves a trailing remainder after filling full batches, the algorithm randomizes those trailing samples and intersperses them across the existing batches instead of packing them into one final, unbalanced plate. Export the plate-assignment table and a per-plate group-distribution summary, then verify each plate's distribution against the overall cohort ratio (e.g. 615 Male/434 Female/251 Unknown; 650 treatment/650 placebo; 169 Kids/351 Teenager/481 Middle-Aged/299 Older as in the case study).

## Related tools

- **InjectionDesign** (Web service (or locally deployed JAR) whose inter-batch balancing algorithm performs the trailing-sample randomization and interspersion across batches) — https://github.com/CSi-Studio/InjectionDesign
- **Excel** (Format used to import the sample list (via the downloadable template) carrying the stratification fields used for balancing)
- **Java (JDK 17)** (Runtime for the locally deployed InjectionDesign service when the web service is not used) — https://github.com/CSi-Studio/InjectionDesign

## Examples

```
java -jar /YOUR_PATH/InjectionDesignDeploy.jar  # then in the web UI (localhost:8080): upload the Excel template, set balance dimensions, plate type 96-well, max 96 samples/plate, and run inter-batch balancing
```

## Evaluation signals

- Plate-assignment table contains exactly one row per input sample (e.g. 1300 data rows for the case-study cohort) — no samples dropped or duplicated by the remainder handling
- No plate exceeds the configured maximum experimental sample capacity (96 per plate on 96-well plates, excluding QC samples)
- Per-plate group-distribution summary shows each single plate's distribution is close to the overall cohort ratio, including plates that received interspersed trailing samples
- Trailing samples are not concentrated in a single final batch; visual inspection of the real-time distribution sequence shows they are dispersed across plates
- Batch-to-batch group ratios for each stratification dimension (e.g. treatment/placebo, gender, age group) deviate only slightly from the cohort-level ratios

## Limitations

- When there are too many groups in one dimension, the averaging step can lead to local aggregation under some special numerical distributions
- Block/balanced randomization averages out the natural numerical characteristics of the sample feature distribution; the complete randomization algorithm is only a partial alternative and 'not a perfect solution'
- Trailing samples are randomized, so results are not deterministic; the assignment should be traced and saved (persistent storage) for long-term multi-week or multi-month projects

## Evidence

- [methods] Inter-batch balancing step partitions samples across batches: "We must evenly distribute the samples with the same characteristics in each batch before injection, such as age, gender, and other factors."
- [methods] Handling of samples out of balance: "This work presents an optimized block randomization approach for handling dispersed samples that are out of balance."
- [readme] Balance and randomization dimensions are configured separately with real-time visual feedback: [evidence span withheld — anchor status: external_doc]
- [intro] Rationale for randomizing across batches: "Randomizing the pretreated samples according to appropriate dimensions is an important method in the QC process for reducing machine drift and batch effect"
- [readme] Per-plate capacity is user-set and excludes QC samples: "users need to set the plate type, max samples on single plate(exclude qc samples), layout direction and the 
yAxis label format of the plate."
