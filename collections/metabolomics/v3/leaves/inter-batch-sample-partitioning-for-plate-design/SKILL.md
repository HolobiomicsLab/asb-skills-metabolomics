---
name: inter-batch-sample-partitioning-for-plate-design
description: Use when a cohort of samples (e.g. hundreds of samples spanning 2-, 3-,
  and 4-category strata like treatment/placebo, gender, age group) must be partitioned
  into multiple 96-well plates so each plate's group distribution mirrors the full
  cohort, prior to intra-batch block randomization.
when_to_use_negative:
- The task is ordering samples within a single plate/injection sequence — that is
  the intra-batch block/optimized block randomization step, a different algorithm
- The cohort already fits on one plate, so no between-plate stratification is needed
- You need to preserve natural imbalances in sample feature distribution — block-style
  averaging 'completely averages out the distribution of the sample features, thus
  losing some of the numerical characteristics that are naturally present in the sample'
tools:
- name: InjectionDesign
  role: Web service / local application that performs the inter-batch balancing algorithm
    and produces the plate-assignment table
  repo: https://github.com/CSi-Studio/InjectionDesign
- name: Excel
  role: Format used to import the sample cohort and download/export the worksheet
    and assignment tables
- name: MongoDB
  role: Backend dependency for persistent project/sample storage when deploying InjectionDesign
    locally
- name: Redis
  role: Backend dependency (port 6379) required when running the InjectionDesign jar
    locally
- name: Java (JDK 17)
  role: Runtime used to launch the InjectionDesign application (java -jar)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2023.02.26.530140
    title: 'InjectionDesign: Plate Design with Optimized Stratified Block Randomization
      for Modern LC/GC-MS-Based Sample Preparation'
  source_content_hash: sha256:99c44bc4d3e7acc00d956ff7ce7d675db2cc7a0f4da3759f6c292f2e3fb3bafc
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# inter-batch-sample-partitioning-for-plate-design

## Summary

Evenly distributes a large sample cohort across multiple injection plates (batches) so that each batch's group distribution matches the overall cohort ratio across classification dimensions such as gender, treatment, and age group, as implemented in InjectionDesign's inter-batch balancing algorithm for LC/GC-MS plate design.

## When to use

Use when a cohort of samples (e.g. hundreds of samples spanning 2-, 3-, and 4-category strata like treatment/placebo, gender, age group) must be partitioned into multiple 96-well plates so each plate's group distribution mirrors the full cohort, prior to intra-batch block randomization.

## When NOT to use

- The task is ordering samples within a single plate/injection sequence — that is the intra-batch block/optimized block randomization step, a different algorithm
- The cohort already fits on one plate, so no between-plate stratification is needed
- You need to preserve natural imbalances in sample feature distribution — block-style averaging 'completely averages out the distribution of the sample features, thus losing some of the numerical characteristics that are naturally present in the sample'

## Inputs

- Sample cohort list via InjectionDesign's Excel template (fields: sample ID plus up to three classification dimensions, e.g. gender, treatment, age group)
- Plate specification (x/y-axis numbers and labels, e.g. 96-well: x=12, y=8)
- Maximum experimental sample capacity per plate (e.g. 96, excluding QC wells)

## Outputs

- plate_assignments.csv with exactly one row per sample carrying its plate assignment and stratification labels
- Per-plate group-distribution summary table showing each plate's stratum counts vs. the overall cohort ratio
- Visually rendered batch/plate distribution in the web interface

## How to apply

Import the sample list into InjectionDesign via the downloadable Excel template (persistent project storage in MongoDB), supporting up to three classification dimensions. Configure the inter-batch balancing algorithm to stratify across all selected dimensions so samples with the same characteristics (e.g. 615 Male/434 Female/251 Unknown gender; 650 treatment/650 placebo; Kids/Teenager/Middle-Aged/Older age groups) are evenly distributed into each batch. Set the plate specification to the 96-well template (x-axis=12, y-axis=8) and the maximum experimental sample capacity to 96 per plate, excluding predefined QC wells (Blank QC, Solvent QC, Pooled QC, Long-Term Reference QC). Execute the balancing step so the cohort is partitioned into plates of up to 96 samples each, with the algorithm randomizing and interspersing trailing samples across batches rather than leaving them as a block. Export the plate-assignment table (one row per sample with plate and stratification labels) plus a per-plate group-distribution summary, then verify each single plate's distribution against the overall cohort ratio.

## Related tools

- **InjectionDesign** (Web service / local application that performs the inter-batch balancing algorithm and produces the plate-assignment table) — https://github.com/CSi-Studio/InjectionDesign
- **Excel** (Format used to import the sample cohort and download/export the worksheet and assignment tables)
- **MongoDB** (Backend dependency for persistent project/sample storage when deploying InjectionDesign locally)
- **Redis** (Backend dependency (port 6379) required when running the InjectionDesign jar locally)
- **Java (JDK 17)** (Runtime used to launch the InjectionDesign application (java -jar))

## Examples

```
java -jar /YOUR_PAHT/InjectionDesignDeploy.jar
```

## Evaluation signals

- plate_assignments.csv contains exactly one data row per input sample (e.g. 1300 rows for the case-study cohort)
- Each single plate's group distribution across all stratification dimensions is extremely close to the overall cohort ratio, as verified in the case study (Figure 4c)
- No plate exceeds the configured maximum experimental sample capacity (96 samples on the 96-well template, excluding QC wells)
- Trailing samples that do not fill a complete batch are randomized and interspersed across batches rather than concentrated in the last plate
- Samples with the same characteristics (gender/treatment/age strata) are present in each batch rather than clustered into a subset of plates

## Limitations

- Block-style balancing completely averages out sample feature distributions, losing natural numerical characteristics present in the sample; the complete randomization algorithm is only a partial remedy
- When there are too many groups in one dimension, the averaging step can cause local aggregation under some special numerical distributions
- Balancing dimensions and randomization dimensions must be defined separately and re-checked after each adjustment; new QC sample types and injection rules may not yet be supported by the web service

## Evidence

- [methods] inter-batch balancing rationale: "We must evenly distribute the samples with the same characteristics in each batch before injection, such as age, gender, and other factors."
- [readme] defining balance vs randomization dimensions: [evidence span withheld — anchor status: external_doc]
- [intro] stratification rationale: "A stratified block randomization algorithm is an important and fast method to reduce the confounding variables’ interference"
- [discussion] loss of natural distribution: "the block randomization algorithm completely averages out the distribution of the sample features, thus losing some of the numerical characteristics that are naturally present in the sample"
