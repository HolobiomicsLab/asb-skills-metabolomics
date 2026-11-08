---
name: cohort-stratified-balancing-across-demographic-dimensions
description: Use when a large cohort (hundreds to thousands of samples) must be partitioned
  into fixed-capacity batches/plates so that each batch's distribution of confounding
  variables (gender, treatment/placebo, age group) mirrors the whole-cohort ratio
  before LC/GC-MS injection.
when_to_use_negative:
- The cohort is small or a single batch fits all samples — inter-batch balancing only
  matters when samples must be split across multiple batches/plates.
- The goal is to control within-plate positional layout or injection-sequence adjacency
  (e.g. avoiding three consecutive same-group samples) — use intra-batch block randomization
  / position design instead.
- You need to preserve natural, unequal numerical characteristics of the sample feature
  distribution — the discussion notes block randomization 'completely averages out
  the distribution of the sample features'.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: InjectionDesign
  role: Web service (or local JAR) that imports the cohort, runs the inter-batch balancing
    algorithm, and exports plate assignments
  repo: https://github.com/CSi-Studio/InjectionDesign
- name: Excel
  role: Template-based import format for the cohort sample list with classification
    labels
- name: MongoDB
  role: Persistent storage backend for projects and imported sample information in
    long-term studies
- name: Java (JDK 17)
  role: Runtime for the local installation package (java -jar InjectionDesignDeploy.jar)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2023.02.26.530140
    title: 'InjectionDesign: Plate Design with Optimized Stratified Block Randomization
      for Modern LC/GC-MS-Based Sample Preparation'
  source_content_hash: sha256:c12207507aa73d41ecef56d396e2ca473e08fb1c983a8af55a53f7b032133fb3
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# cohort-stratified-balancing-across-demographic-dimensions

## Summary

Evenly distributes a large sample cohort across batches (plates) so that each batch's group composition across multiple classification dimensions (e.g. gender, treatment, age group) matches the overall cohort ratio, using InjectionDesign's inter-batch balancing algorithm.

## When to use

Use when a large cohort (hundreds to thousands of samples) must be partitioned into fixed-capacity batches/plates so that each batch's distribution of confounding variables (gender, treatment/placebo, age group) mirrors the whole-cohort ratio before LC/GC-MS injection.

## When NOT to use

- The cohort is small or a single batch fits all samples — inter-batch balancing only matters when samples must be split across multiple batches/plates.
- The goal is to control within-plate positional layout or injection-sequence adjacency (e.g. avoiding three consecutive same-group samples) — use intra-batch block randomization / position design instead.
- You need to preserve natural, unequal numerical characteristics of the sample feature distribution — the discussion notes block randomization 'completely averages out the distribution of the sample features'.

## Inputs

- Sample list imported via the InjectionDesign Excel template (per-sample classification labels: e.g. gender, treatment, age group)
- Plate specification (x/y axis numbers and labels, e.g. 96-well: 12 x 8)
- Maximum experimental sample capacity per plate (e.g. 96, excluding QC samples)
- Selected balance dimensions (classification columns to stratify across)

## Outputs

- Plate-assignment table (one row per sample with its plate assignment and stratification labels, exportable as plate_assignments.csv)
- Per-plate group-distribution summary table showing each plate's composition vs. the overall cohort ratio
- Visualized distribution sequence of the balanced assignment

## How to apply

Load the cohort sample list (e.g. a 1300-sample cohort with gender 615 Male/434 Female/251 Unknown, 650 treatment/650 placebo, and age groups 169 Kids/351 Teenager/481 Middle-Aged/299 Older) into InjectionDesign via the downloaded Excel template, relying on persistent project storage (MongoDB) for long-term projects. Configure the inter-batch balancing step to stratify across all classification dimensions simultaneously (two-, three-, and four-category dimensions) so samples with the same characteristics are evenly distributed into each batch. Set the plate specification (e.g. built-in 96-well template, x-axis=12, y-axis=8) and the maximum experimental sample capacity per plate (excluding QC samples) to 96, choosing layout direction and y-axis label format. Execute inter-batch balancing so the 1300 samples are partitioned into plates of up to 96 samples each; trailing samples that do not fill a complete plate are randomized and interspersed across batches rather than clustered. Export the plate-assignment table (one row per sample with plate and stratification labels) plus a per-plate group-distribution summary, then verify each single plate's group distribution is extremely close to the overall cohort ratio.

## Related tools

- **InjectionDesign** (Web service (or local JAR) that imports the cohort, runs the inter-batch balancing algorithm, and exports plate assignments) — https://github.com/CSi-Studio/InjectionDesign
- **Excel** (Template-based import format for the cohort sample list with classification labels)
- **MongoDB** (Persistent storage backend for projects and imported sample information in long-term studies)
- **Java (JDK 17)** (Runtime for the local installation package (java -jar InjectionDesignDeploy.jar))

## Examples

```
java -jar /YOUR_PATH/InjectionDesignDeploy.jar  # then in the web UI at http://localhost:8080: import Excel sample list, set plate 96-well / max 96 samples, select balance dimensions, run inter-batch balancing, Export All
```

## Evaluation signals

- plate_assignments.csv contains exactly one data row per sample (e.g. 1300 rows for the 1300-sample case study), with no sample dropped or duplicated.
- No plate exceeds the configured maximum experimental sample capacity (e.g. 96 samples per 96-well plate, excluding QC).
- Per-plate group-distribution summary shows each single plate's group proportions are extremely close to the overall cohort ratio (e.g. ~47.3% Male, 50% treatment), as verified in Figure 4c.
- Trailing samples that do not fill a final plate are randomized and interspersed across batches, not clustered into one batch.
- Each selected balance dimension (two-, three-, and four-category) is represented in the stratification output.

## Limitations

- Block randomization completely averages out the sample feature distribution, losing natural numerical characteristics present in the sample.
- When there are too many groups in one dimension, the averaging step calculation can lead to local aggregation under some special numerical distributions.
- Complete randomization is only a partial remedy for scenarios where block randomization loses numerical characteristics — 'it is not a perfect solution'.

## Evidence

- [methods] Inter-batch balancing rationale: "We must evenly distribute the samples with the same characteristics in each batch before injection, such as age, gender, and other factors."
- [results] Case-study outcome (Figure 4c): [evidence span withheld — anchor status: not_found]
- [methods] Treatment of trailing samples: [evidence span withheld — anchor status: not_found]
- [discussion] Averaging-out limitation: "the block randomization algorithm completely averages out the distribution of the sample features, thus losing some of the numerical characteristics that are naturally present in the sample"
- [readme] Excel import workflow (README): "Click "Download Sample Excel" button to download the template excel. Then fill the basic information of samples to the excel and upload the file."
- [readme] Balance-dimension configuration (README): [evidence span withheld — anchor status: external_doc]
