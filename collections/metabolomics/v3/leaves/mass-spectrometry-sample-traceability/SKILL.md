---
name: mass-spectrometry-sample-traceability
description: Use when preparing an injection worksheet for GC/LC-MS metabolomics/proteomics
  where QC samples must occupy a dedicated plate area, well positions must not interpenetrate
  experimental wells, and injection order must be traceable independently of well
  layout.
when_to_use_negative:
- The analysis goal is sequence randomization itself (inter-batch balancing or stratified
  block randomization algorithm selection) rather than plate-position layout and QC
  confinement.
- Samples exceed the plate's maximum experimental sample capacity (96 wells minus
  QC column) and a different plate format or multi-plate balancing strategy is required.
- The workflow relies on automated pre-processing equipment whose plate formats or
  injection rules are not supported by InjectionDesign's predefined QC types.
edam_operation: http://edamontology.org/operation_0321
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: InjectionDesign
  role: Free online (or locally installed) plate design service that separates position
    design from sequence design, supports predefined QC locations, and exports the
    worksheet
  repo: https://github.com/CSi-Studio/InjectionDesign
- name: Excel
  role: Input format for importing sample information via the downloadable sample
    template
- name: Java (JDK 17)
  role: Runtime for local deployment of the InjectionDesign service (java -jar InjectionDesignDeploy.jar)
  repo: https://github.com/CSi-Studio/InjectionDesign
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2023.02.26.530140
    title: 'InjectionDesign: Plate Design with Optimized Stratified Block Randomization
      for Modern LC/GC-MS-Based Sample Preparation'
  source_content_hash: sha256:3f6fd0d083d8e649b8f60551fbffc0236c5f06d7aebe752458748664cf87f024
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/InjectionDesign
  tools:
  - Excel
  - InjectionDesign
  - Java (JDK 17)
  edam_operation: http://edamontology.org/operation_0321
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# mass-spectrometry-sample-traceability

## Summary

Designs 96-well injection plates for LC/GC-MS multi-omics so that sample position layout (plate design) is decoupled from injection sequence design, confining QC samples (Solvent QC, Long-Term Reference QC, Blank, Pooled QC) to a predefined plate column while QC injections are pre-interpolated evenly among experimental samples.

## When to use

Use when preparing an injection worksheet for GC/LC-MS metabolomics/proteomics where QC samples must occupy a dedicated plate area, well positions must not interpenetrate experimental wells, and injection order must be traceable independently of well layout.

## When NOT to use

- The analysis goal is sequence randomization itself (inter-batch balancing or stratified block randomization algorithm selection) rather than plate-position layout and QC confinement.
- Samples exceed the plate's maximum experimental sample capacity (96 wells minus QC column) and a different plate format or multi-plate balancing strategy is required.
- The workflow relies on automated pre-processing equipment whose plate formats or injection rules are not supported by InjectionDesign's predefined QC types.

## Inputs

- Randomized sample assignment list (from block/stratified randomization) with up to three classification dimensions
- Sample Excel template filled with basic sample information (from InjectionDesign)
- 96-well plate template specification: x-axis=12 (labels 1..12), y-axis=8 (labels a..h)
- QC sample counts per type (Blank QC, Solvent QC, Pooled QC, Long-Term Reference QC, custom QC)

## Outputs

- Worksheet file with two sections: well-position layout and injection order
- Visualized plate layout with QC samples color-coded by type in the predefined column
- Downloadable worksheet set (single or 'Export All') for each plate, persisted for long-term projects

## How to apply

In InjectionDesign, first create a project and import the randomized sample list (e.g., from task_002 block randomization) via the template Excel file, which supports up to three classification dimensions. Configure the plate specification (x-axis=12 with labels [1..12], y-axis=8 with labels [a..h] for a 96-well plate) and set the plate load capacity (max 96 experimental samples per plate, excluding QC) and QC sample counts per type (Blank QC, Solvent QC, Pooled QC, Long-Term Reference QC, plus one custom QC). Reserve the first (or last) plate column as the Predefined QC Location so all QC wells sit together, which reduces operational complexity from QC sample interpenetration, preprocessing time, and error rate. Run inter-batch balancing and intra-batch randomization, letting the tool pre-interpolate QC samples evenly in a circular fashion among the randomized experimental samples for the injection order. Download the worksheet (individually or via 'Export All'), which contains two distinct sections — the well-position layout and the injection sequence — so position and order remain decoupled and traceable; validate that QC wells appear only in the designated column and that both sections are present.

## Related tools

- **InjectionDesign** (Free online (or locally installed) plate design service that separates position design from sequence design, supports predefined QC locations, and exports the worksheet) — https://github.com/CSi-Studio/InjectionDesign
- **Excel** (Input format for importing sample information via the downloadable sample template)
- **Java (JDK 17)** (Runtime for local deployment of the InjectionDesign service (java -jar InjectionDesignDeploy.jar)) — https://github.com/CSi-Studio/InjectionDesign

## Examples

```
java -jar /YOUR_PATH/InjectionDesignDeploy.jar  # then open http://localhost:8080/, upload the sample Excel, set plate 12x8 + QC column, and 'Export All' worksheets
```

## Evaluation signals

- The worksheet contains both a well-position layout section and a separate injection sequence section (decoupled position vs. order).
- All QC samples (Solvent QC, Long Term Reference QC, Blank, Pooled/custom QC) occupy wells only within the predefined QC column; no QC well interpenetrates an experimental sample well.
- QC samples in the injection sequence are evenly pre-interpolated in a circular fashion among randomized experimental samples.
- Total experimental wells do not exceed the configured max samples on a single plate (96 for the standard 12x8 plate), excluding QC samples.
- For block randomization with block size 2, no three consecutive samples in the injection sequence belong to the same group in the balance dimension.

## Limitations

- Block randomization completely averages out the distribution of sample features, losing naturally present numerical characteristics; complete randomization is only a partial remedy.
- Optimized block randomization can cause local aggregation when there are too many groups in one dimension under some special numerical distributions.
- New QC sample types and injection rules may emerge that are not yet supported by the web service.
- Local deployment requires JDK 17, Redis 5+ (port 6379), and MongoDB 4.2+ (port 27017), with at least 8GB RAM and 40GB disk.

## Evidence

- [abstract] It offers the ability to separate the position design from the sequence design, making the output more compatible with the requirements of a modern mass spectrometer-based laboratory.: "It offers the ability to separate the position design from the sequence design, making the output more compatible with the requirements of a modern mass spectrometer-based laboratory."
- [intro] we can put QC samples in the same area of the left part to reduce the operational complexity caused by QC sample interpenetration. This can greatly reduce the preprocessing time and error rate.: "we can put QC samples in the same area of the left part to reduce the operational complexity caused by QC sample interpenetration. This can greatly reduce the preprocessing time and error rate."
- [methods] InjectionDesign includes common QC sample types. Solvent QC Samples, Long Term Reference (LTR) QC Samples, Blank Samples, and so on.: "InjectionDesign includes common QC sample types. Solvent QC Samples, Long Term Reference (LTR) QC Samples, Blank Samples, and so on."
- [readme] InjectionDesign predefined four QC type: Blank QC, Solvent QC, Pooled QC, Long-Term Reference QC. and one custom QC.: "InjectionDesign predefined four QC type: Blank QC, Solvent QC, Pooled QC, Long-Term Reference QC. and one custom QC."
- [methods] Users should determine the number of QC samples of each type used. Select the custom plate's maximum experimental sample capacity as well.: [evidence span withheld — anchor status: not_found]
