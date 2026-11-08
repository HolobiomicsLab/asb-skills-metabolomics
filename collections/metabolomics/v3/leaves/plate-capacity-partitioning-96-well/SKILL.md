---
name: plate-capacity-partitioning-96-well
description: Use when a large cohort (hundreds+ samples) must be split across multiple
  96-well plates with per-plate group distributions matching the overall cohort ratio,
  using 96-well capacity (x=12, y=8, max 96 experimental samples/plate).
when_to_use_negative:
- The cohort fits on a single plate or the run involves few enough samples that manual/Excel-based
  assignment suffices without batch effects
- The analysis must preserve the natural numerical characteristics of the sample distribution,
  since block randomization 'completely averages out the distribution of the sample
  features'
- The scenario requires new QC sample types or injection rules not yet supported by
  the web service
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: InjectionDesign
  role: Runs the inter-batch balancing algorithm that stratifies samples across classification
    dimensions and partitions them onto 96-well plates
  repo: https://github.com/CSi-Studio/InjectionDesign
- name: Excel
  role: Import of the cohort sample list via the downloadable sample Excel template
- name: MongoDB
  role: Persistent storage of the imported sample information and project state for
    long-term projects
- name: Redis
  role: Backend service component required by the local InjectionDesign deployment
- name: Java (JDK 17)
  role: Runtime for the locally deployed InjectionDesign jar that executes the plate
    partitioning
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2023.02.26.530140
    title: 'InjectionDesign: Plate Design with Optimized Stratified Block Randomization
      for Modern LC/GC-MS-Based Sample Preparation'
  source_content_hash: sha256:f4ed30921cd9fe09b6506a8c20ad8191e7d8b1d42a20dccbb574413c7cb3908b
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# plate-capacity-partitioning-96-well

## Summary

InjectionDesign's inter-batch balancing workflow partitions a large sample cohort (e.g., the 1300-sample case study) across multiple 96-well plates so that samples sharing characteristics (gender, treatment, age group) are evenly distributed, yielding per-plate group distributions close to the overall cohort ratio (Figure 4c).

## When to use

Use when a large cohort (hundreds+ samples) must be split across multiple 96-well plates with per-plate group distributions matching the overall cohort ratio, using 96-well capacity (x=12, y=8, max 96 experimental samples/plate).

## When NOT to use

- The cohort fits on a single plate or the run involves few enough samples that manual/Excel-based assignment suffices without batch effects
- The analysis must preserve the natural numerical characteristics of the sample distribution, since block randomization 'completely averages out the distribution of the sample features'
- The scenario requires new QC sample types or injection rules not yet supported by the web service

## Inputs

- Cohort sample list imported via Excel template, with classification-dimension fields (e.g., gender: 615 Male/434 Female/251 Unknown; treatment: 650 treatment/650 placebo; age group: 169 Kids/351 Teenager/481 Middle-Aged/299 Older)
- Plate specification parameters: x-axis=12, y-axis=8 (96-well), axis labels, layout direction
- Max experimental sample capacity per plate (96, excluding QC samples)
- Balance-dimension definitions for inter-batch stratification

## Outputs

- plate_assignments.csv with one row per sample containing plate ID and stratification labels (1300 rows for the case study)
- Per-plate group-distribution summary table comparing each plate's ratio to the overall cohort ratio

## How to apply

Import the full cohort list via the Excel template into an InjectionDesign project backed by persistent storage (MongoDB, Redis). Configure inter-batch balancing to stratify across all classification dimensions (e.g., a two-category gender dimension, a two-category treatment/placebo dimension, and a four-category age dimension) so samples with the same characteristics are evenly distributed into each batch. Set the plate specification to the built-in 96-well template (x-axis=12, y-axis=8) and set the maximum experimental sample capacity to 96 per plate, excluding QC samples. Execute the partitioning so trailing samples that don't fill a complete plate are randomized and interspersed across batches rather than concentrated on the last plate. Export the plate-assignment table and a per-plate group-distribution summary, then verify the row count and that each plate's distribution is close to the cohort ratio.

## Related tools

- **InjectionDesign** (Runs the inter-batch balancing algorithm that stratifies samples across classification dimensions and partitions them onto 96-well plates) — https://github.com/CSi-Studio/InjectionDesign
- **Excel** (Import of the cohort sample list via the downloadable sample Excel template)
- **MongoDB** (Persistent storage of the imported sample information and project state for long-term projects)
- **Redis** (Backend service component required by the local InjectionDesign deployment)
- **Java (JDK 17)** (Runtime for the locally deployed InjectionDesign jar that executes the plate partitioning)

## Examples

```
java -jar /YOUR_PATH/InjectionDesignDeploy.jar
```

## Evaluation signals

- plate_assignments.csv contains exactly one data row per cohort sample (1300 rows for the case study)
- Per-plate summary shows each single plate's group distribution is extremely close to the overall cohort ratio, as verified in Figure 4c
- No plate exceeds the configured maximum experimental sample capacity of 96 (excluding QC samples)
- All three classification dimensions (two-, two-, and four-category) appear as stratification labels on each assignment row
- Trailing samples from the final partial plate are randomized/interspersed rather than clustered on a single batch

## Limitations

- Block randomization completely averages out the distribution of sample features, losing natural numerical characteristics of the sample distribution
- When there are too many groups in one dimension, the averaging-step calculation can lead to local aggregation under some special numerical distributions
- Complete randomization (also provided by InjectionDesign) is 'not a perfect solution' for scenarios where block randomization loses numerical characteristics
- Balancing quality depends on the cohort having known labels for each dimension; the case-study cohort includes 251 Unknown gender entries

## Evidence

- [methods] Inter-batch stratification across sample characteristics: "We must evenly distribute the samples with the same characteristics in each batch before injection, such as age, gender, and other factors."
- [readme] Plate specification and capacity configuration: "users need to set the plate type, max samples on single plate(exclude qc samples), layout direction and the yAxis label format of the plate."
- [readme] Separate balance and randomization dimensions: [evidence span withheld — anchor status: external_doc]
- [methods] Excel import path for sample data: "Alternatively, samples can be imported directly via Excel."
- [readme] Local deployment runtime: "The key software that will be installed includes: - Java Development Kit (JDK) 17 - Redis - MongoDB"
