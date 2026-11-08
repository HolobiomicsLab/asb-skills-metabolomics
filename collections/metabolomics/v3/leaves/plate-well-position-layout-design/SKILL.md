---
name: plate-well-position-layout-design
description: Use when preparing an LC/GC-MS multi-omics sample plate and you must
  assign randomized experimental samples to specific wells while keeping QC sample
  types spatially separated in a designated plate area.
when_to_use_negative:
- You only need the injection order and already have physical well positions fixed
  — the injection-sequence/pre-interpolation-of-QC skill handles ordering and should
  be applied separately.
- Your samples need interleaving of QC wells among experimental wells on the plate
  (this skill deliberately confines QC to a dedicated column and would not match such
  a protocol).
- The task is multi-plate inter-batch balancing of sample features (age, gender, etc.)
  across batches rather than within-batch plate position assignment — use the inter-batch
  balancing algorithm instead.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: InjectionDesign
  role: Free online plate-design web service (or locally deployed Java application)
    that performs the four-part workflow — Plate Specification, Plate Design Load,
    Predefined QC Location, Injection Order — and emits the worksheet with the well-position
    layout
  repo: https://github.com/CSi-Studio/InjectionDesign
- name: Excel
  role: Format of the downloadable sample template used to import sample basic information
    and the worksheet format for the plate layout output
- name: MongoDB
  role: Persistent storage backend for project/sample injection information so plate
    layouts can be traced and adjusted for long-term projects
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2023.02.26.530140
    title: 'InjectionDesign: Plate Design with Optimized Stratified Block Randomization
      for Modern LC/GC-MS-Based Sample Preparation'
  source_content_hash: sha256:6a8babd1f0a95f270e5967d41add44d62f75e3b2c1724d655442a83ae0d6468e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/InjectionDesign
  tools:
  - Excel
  - InjectionDesign
  - MongoDB
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# plate-well-position-layout-design

## Summary

Designs the physical well-position layout of experimental and QC samples on a plate (e.g., a 96-well plate with x-axis 1–12 and y-axis a–h) for LC/GC-MS-based sample preparation, deliberately decoupling position design from injection-sequence design. Confining QC samples (Solvent QC, Long Term Reference QC, Blank, Pooled QC) to a predefined plate column reduces operational complexity, preprocessing time, and error rate caused by QC interpenetration.

## When to use

Use when preparing an LC/GC-MS multi-omics sample plate and you must assign randomized experimental samples to specific wells while keeping QC sample types spatially separated in a designated plate area.

## When NOT to use

- You only need the injection order and already have physical well positions fixed — the injection-sequence/pre-interpolation-of-QC skill handles ordering and should be applied separately.
- Your samples need interleaving of QC wells among experimental wells on the plate (this skill deliberately confines QC to a dedicated column and would not match such a protocol).
- The task is multi-plate inter-batch balancing of sample features (age, gender, etc.) across batches rather than within-batch plate position assignment — use the inter-batch balancing algorithm instead.

## Inputs

- Randomized sample assignment from the intra-batch balancing/block-randomization step (imported via the sample Excel template or the InjectionDesign sample list)
- Plate specification (x-axis numbers/labels, e.g., 12; y-axis numbers/labels, e.g., a–h for a 96-well plate)
- QC sample counts by type (Blank QC, Solvent QC, Pooled QC, Long-Term Reference QC, custom QC)
- Plate load capacity (maximum experimental samples per plate, excluding QC) and layout direction / y-axis label format

## Outputs

- Well-position layout mapping every experimental sample and QC sample to a specific plate well (QC confined to the predefined column)
- Plate design worksheet (Excel) with a dedicated well-position layout section, visually presented in the web interface and downloadable individually or via 'Export All'

## How to apply

In InjectionDesign, first configure the plate specification by setting the x- and y-axis numbers and labels (e.g., x=12 columns, y=8 rows for a 96-well plate) and choose the layout direction and y-axis label format. Next, set the plate load capacity: the number of QC samples of each type (Blank QC, Solvent QC, Pooled QC, Long-Term Reference QC, plus one custom QC) and the maximum experimental sample capacity per plate (max 96 samples per plate, excluding QC). Then define the predefined QC location by reserving one plate column (e.g., the first or last column) so QC wells never interpenetrate experimental wells, and assign the randomized experimental samples from the intra-batch balancing/randomization step to the remaining wells. The key design decision is separation of concerns: well position is decided independently of the injection sequence (QC samples are pre-interpolated evenly in a circular fashion in the sequence), which makes the output compatible with modern mass spectrometer-based laboratories and keeps the plate layout standardized and humanized. Position design can be traced and adjusted for long multi-week or multi-month projects because sample injection information is stored persistently (MongoDB) in a project.

## Related tools

- **InjectionDesign** (Free online plate-design web service (or locally deployed Java application) that performs the four-part workflow — Plate Specification, Plate Design Load, Predefined QC Location, Injection Order — and emits the worksheet with the well-position layout) — https://github.com/CSi-Studio/InjectionDesign
- **Excel** (Format of the downloadable sample template used to import sample basic information and the worksheet format for the plate layout output)
- **MongoDB** (Persistent storage backend for project/sample injection information so plate layouts can be traced and adjusted for long-term projects)

## Examples

```
java -jar /YOUR_PATH/InjectionDesignDeploy.jar  # then at http://localhost:8080/ configure Plate Specification (x=12, y=a–h), QC location, and download the plate-layout worksheet
```

## Evaluation signals

- The worksheet contains two distinct sections — a well-position layout section and an injection order section — confirming position/sequence decoupling.
- All QC samples (Solvent QC, Long Term Reference QC, Blank) occupy wells only in the predefined QC column; no QC well interpenetrates an experimental sample well.
- Number of experimental samples assigned to wells does not exceed the configured maximum experimental sample capacity per plate (e.g., ≤ 96 on a 12×8 plate, excluding QC).
- Plate specification matches the requested template (x-axis 12 with labels 1–12, y-axis 8 with labels a–h) and the chosen layout direction and y-axis label format.
- Every randomized experimental sample from the balancing/randomization step appears exactly once in the layout, with no wells left unassigned or double-assigned.

## Limitations

- Block randomization averages out the distribution of sample features, losing some naturally present numerical characteristics; the complete randomization algorithm only partially covers such scenarios and 'is not a perfect solution'.
- Optimized block randomization can produce local aggregation when there are too many groups in one dimension under some special numerical distributions.
- New QC sample types and injection rules may emerge that are not yet supported by the web service, and automated pre-processing equipment that could bypass manual plate handling is frequently costly.

## Evidence

- [abstract] It offers the ability to separate the position design from the sequence design, making the output more compatible with the requirements of a modern mass spectrometer-based laboratory.: "It offers the ability to separate the position design from the sequence design, making the output more compatible with the requirements of a modern mass spectrometer-based laboratory."
- [intro] we can put QC samples in the same area of the left part to reduce the operational complexity caused by QC sample interpenetration.: "we can put QC samples in the same area of the left part to reduce the operational complexity caused by QC sample interpenetration. This can greatly reduce the preprocessing time and error rate."
- [methods] Users must customize the plate’s specifications. The plate information includes x- and y-axis numbers as well as x- and y-axis labels.: "Users must customize the plate’s specifications. The plate information includes x- and y-axis numbers as well as x- and y-axis labels."
- [methods] Users should determine the number of QC samples of each type used. Select the custom plate’s maximum experimental sample capacity as well.: "Users should determine the number of QC samples of each type used. Select the custom plate’s maximum experimental sample capacity as well."
- [readme] users need to set the plate type, max samples on single plate(exclude qc samples), layout direction and the yAxis label format of the plate.: "Besides, users need to set the plate type, max samples on single plate(exclude qc samples), layout direction and the 
yAxis label format of the plate."
- [readme] InjectionDesign predefined four QC type: Blank QC, Solvent QC, Pooled QC, Long-Term Reference QC. and one custom QC.: "InjectionDesign predefined four QC type: Blank QC, Solvent QC, Pooled QC, Long-Term Reference QC. and one custom QC."
