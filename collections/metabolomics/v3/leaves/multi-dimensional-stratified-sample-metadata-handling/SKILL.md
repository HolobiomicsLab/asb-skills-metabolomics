---
name: multi-dimensional-stratified-sample-metadata-handling
description: Use when a sample cohort table carries 2–3 categorical metadata dimensions
  (e.g., two-category treatment, three-category gender, four-category age group) that
  must be balanced across batches and randomized within a plate before LC/GC-MS injection
  to avoid drift and batch effects.
when_to_use_negative:
- You need to preserve the natural numerical characteristics of the sample feature
  distribution — block randomization completely averages out the distribution, and
  complete randomization is explicitly not a perfect substitute
- A dimension has too many groups under special numerical distributions — the optimized
  block randomization's averaging-step calculation can lead to local aggregation
- Samples have no categorical metadata dimensions at all and only require QC placement
  (use the Predefined QC Location / Pre-Interpolation of QC workflow instead)
tools:
- name: InjectionDesign
  role: Web service performing project creation, Excel-based sample import, inter-batch
    balancing, and multi-dimensional intra-batch stratified block randomization with
    visualized worksheet export
  repo: https://github.com/CSi-Studio/InjectionDesign
- name: Java (JDK 17)
  role: Runtime environment for local deployment of the InjectionDesign service (java
    -jar InjectionDesignDeploy.jar)
  repo: https://github.com/CSi-Studio/InjectionDesign
- name: Excel
  role: Input format for the sample metadata template uploaded into the project's
    persistent storage
- name: MongoDB
  role: Persistent storage backend for sample injection information in long-term projects
    when self-hosting
  repo: https://github.com/CSi-Studio/InjectionDesign
- name: Redis
  role: Required service dependency (port 6379) for the local InjectionDesign deployment
  repo: https://github.com/CSi-Studio/InjectionDesign
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2023.02.26.530140
    title: 'InjectionDesign: Plate Design with Optimized Stratified Block Randomization
      for Modern LC/GC-MS-Based Sample Preparation'
  source_content_hash: sha256:888b9772a609b32e0c68228e25021fa2370ce2353d50e094593578f394d6ff64
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/InjectionDesign
  tools:
  - Excel
  - InjectionDesign
  - Java
  - Java (JDK 17)
  - MongoDB
  - Redis
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# multi-dimensional-stratified-sample-metadata-handling

## Summary

Import, balance, and randomize a sample cohort across multiple metadata classification dimensions (e.g., treatment/placebo, gender, age group) using InjectionDesign's inter-batch balancing and intra-batch stratified block randomization for LC/GC-MS plate and injection-sequence design.

## When to use

Use when a sample cohort table carries 2–3 categorical metadata dimensions (e.g., two-category treatment, three-category gender, four-category age group) that must be balanced across batches and randomized within a plate before LC/GC-MS injection to avoid drift and batch effects.

## When NOT to use

- You need to preserve the natural numerical characteristics of the sample feature distribution — block randomization completely averages out the distribution, and complete randomization is explicitly not a perfect substitute
- A dimension has too many groups under special numerical distributions — the optimized block randomization's averaging-step calculation can lead to local aggregation
- Samples have no categorical metadata dimensions at all and only require QC placement (use the Predefined QC Location / Pre-Interpolation of QC workflow instead)

## Inputs

- Sample cohort table (Excel template or imported table) with categorical metadata columns, e.g., gender (Male/Female/Unknown), treatment (treatment/placebo), age group (Kids/Teenager/Middle-Aged/Older)
- Plate specification: x- and y-axis numbers/labels, layout direction, yAxis label format
- Plate load capacity settings: per-type QC sample counts (Blank, Solvent, Pooled, LTR, custom QC) and max experimental sample capacity per plate
- User-selected balance dimension and randomization dimension definitions

## Outputs

- Inter-batch-balanced sample distribution across batches
- Intra-batch randomized injection sequence per plate, visualized in real time
- Downloadable worksheet files per plate (or via 'Export All')
- Feature-aggregation tallies such as complete_randomization_feature_aggregation.csv when auditing randomized sequences

## How to apply

Create a project in InjectionDesign and import sample metadata via the downloadable sample Excel template (or Excel import), which supports visual presentation of up to three classification dimensions. Define the balance dimension and the randomization dimension separately: first run inter-batch balancing to evenly distribute samples with the same characteristics (age, gender, etc.) across batches, then run intra-batch stratified block randomization on the intra-batch samples. Apply the randomization across all dimensions simultaneously (e.g., two-category treatment, three-category gender, four-category age group); with block size 2, no three consecutive samples of the same group should occur. To audit a sequence, scan it for runs of consecutive samples sharing the same category per dimension and tally these aggregation events (e.g., writing counts to complete_randomization_feature_aggregation.csv) — block randomization suppresses these runs, whereas complete randomization produces many aggregated features with high probability, matching the clustered regions in Figure 5b.

## Related tools

- **InjectionDesign** (Web service performing project creation, Excel-based sample import, inter-batch balancing, and multi-dimensional intra-batch stratified block randomization with visualized worksheet export) — https://github.com/CSi-Studio/InjectionDesign
- **Java (JDK 17)** (Runtime environment for local deployment of the InjectionDesign service (java -jar InjectionDesignDeploy.jar)) — https://github.com/CSi-Studio/InjectionDesign
- **Excel** (Input format for the sample metadata template uploaded into the project's persistent storage)
- **MongoDB** (Persistent storage backend for sample injection information in long-term projects when self-hosting) — https://github.com/CSi-Studio/InjectionDesign
- **Redis** (Required service dependency (port 6379) for the local InjectionDesign deployment) — https://github.com/CSi-Studio/InjectionDesign

## Examples

```
java -jar /YOUR_PATH/InjectionDesignDeploy.jar
```

## Evaluation signals

- With block size 2, the intra-batch randomized sequence contains no three consecutive samples of the same group in any randomized dimension (e.g., treatment/placebo)
- Inter-batch balancing yields an even distribution of samples sharing the same metadata characteristics (e.g., gender, age group) across all batches
- Aggregation tally from sequence scan (e.g., complete_randomization_feature_aggregation.csv) shows near-zero runs under block randomization but many high-probability aggregated features under complete randomization, matching Figure 5b's red-boxed clusters
- All 1300 cohort samples are accounted for exactly once in the output sequence with their category labels intact across all three dimensions
- Visualized distribution sequence in the web UI matches the exported worksheet for each adjustment

## Limitations

- Block randomization completely averages out the distribution of sample features, losing naturally present numerical characteristics
- Complete randomization (the no-stratification fallback) generates a large number of aggregated features with high probability and is not a perfect solution for preserving natural sample distribution
- Optimized block randomization can produce local aggregation when a dimension has too many groups under special numerical distributions
- Self-hosted deployment requires JDK 17, Redis 5+, MongoDB 4.2+ and adequate hardware (min 8GB RAM / 40GB disk); the downloadable installer URL is external (Google Drive) and not version-pinned in the article
- No changelog is available in the repository discussion section

## Evidence

- [intro] A stratified block randomization algorithm is an important and fast method to reduce the confounding variables’ interference: "A stratified block randomization algorithm is an important and fast method to reduce the confounding variables’ interference"
- [methods] since the block size is 2, theoretically there were no three consecutive samples of the same group in the results.: "since the block size is 2, theoretically there were no three consecutive samples of the same group in the results."
- [discussion] the block randomization algorithm completely averages out the distribution of the sample features, thus losing some of the numerical characteristics that are naturally present in the sample: "the block randomization algorithm completely averages out the distribution of the sample features, thus losing some of the numerical characteristics that are naturally present in the sample"
- [readme] InjectionDesign supports visual presentation of up to three classification dimensions.: "InjectionDesign supports visual presentation of up to three classification dimensions."
- [readme] Users need to define the balance dimension and the randomization dimension separately.: "Users need to define the balance dimension and the randomization dimension separately."
