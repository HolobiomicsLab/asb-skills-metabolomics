---
name: qc-sample-column-placement
description: Use when designing a GC/LC-MS injection plate with multiple QC sample
  types and you want QC wells confined to one plate column while the injection order
  is generated separately via pre-interpolation of QC.
when_to_use_negative:
- When the injection order must be tied 1:1 to physical well positions with no separation
  (e.g., an instrument or legacy workflow that reads plates strictly in row/column
  order)
- When QC samples must be distributed uniformly across the whole plate rather than
  confined to a designated area (opposite of this paper's design rationale)
- When the plate layout is not an InjectionDesign-supported plate format with definable
  x/y-axis numbers and labels
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: InjectionDesign
  role: Free online plate-design web service (also locally installable) that implements
    Plate Specification, Plate Load Capacity, Predefined QC Location, and Pre-Interpolation
    of QC, and emits the two-section worksheet
  repo: https://github.com/CSi-Studio/InjectionDesign
- name: Excel
  role: Input format for importing sample information via the downloadable sample
    Excel template before position design
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2023.02.26.530140
    title: 'InjectionDesign: Plate Design with Optimized Stratified Block Randomization
      for Modern LC/GC-MS-Based Sample Preparation'
  source_content_hash: sha256:4511eac0a7b01f3e68aaddd8f4d4d7e612bf7422ed2db854da3758f4022d94a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/InjectionDesign
  tools:
  - Excel
  - InjectionDesign
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# qc-sample-column-placement

## Summary

In InjectionDesign, reserve a dedicated plate column (e.g., the first or last column of a 96-well plate) exclusively for QC sample types (Solvent QC, Long-Term Reference QC, Blank) while the injection sequence is designed independently of well positions, so QC wells never interpenetrate experimental sample wells.

## When to use

Use when designing a GC/LC-MS injection plate with multiple QC sample types and you want QC wells confined to one plate column while the injection order is generated separately via pre-interpolation of QC.

## When NOT to use

- When the injection order must be tied 1:1 to physical well positions with no separation (e.g., an instrument or legacy workflow that reads plates strictly in row/column order)
- When QC samples must be distributed uniformly across the whole plate rather than confined to a designated area (opposite of this paper's design rationale)
- When the plate layout is not an InjectionDesign-supported plate format with definable x/y-axis numbers and labels

## Inputs

- Randomized sample assignment from inter-batch balancing / intra-batch block randomization (uploaded via the InjectionDesign sample Excel template)
- 96-well plate template (x-axis=12, x-labels=[1..12], y-axis=8, y-labels=[a..h])
- QC sample counts per type (Solvent QC, Long-Term Reference QC, Blank, Pooled, custom) and maximum experimental sample capacity per plate (e.g., 96)

## Outputs

- Worksheet file containing two distinct sections: well-position layout and injection order
- Visualized plate map in which all QC wells are confined to the predefined column (e.g., left part of the plate)

## How to apply

After loading the randomized sample assignment into InjectionDesign, configure the Plate Specification (e.g., 96-well plate: x-axis=12 with labels [1..12], y-axis=8 with labels [a..h]) and set the Plate Load Capacity (max 96 experimental samples per plate, excluding QC). Under Predefined QC Location, reserve the first (or last) plate column for QC sample types — InjectionDesign predefined types are Blank QC, Solvent QC, Pooled QC, Long-Term Reference QC, plus one custom QC — so that QC wells occupy only that column and never interpenetrate experimental sample wells. Assign every experimental sample to a well according to the plate load capacity, keeping QC in the designated column for operator-friendly layout and traceability. Independently of well positions, generate the injection sequence by Pre-Interpolation of QC: QC samples are evenly interleaved in a circular fashion among the randomized experimental samples. Finally, export the worksheet, which contains a well-position layout section and a separate injection-order section; InjectionDesign predefined four QC types each marked with a different color for visual verification.

## Related tools

- **InjectionDesign** (Free online plate-design web service (also locally installable) that implements Plate Specification, Plate Load Capacity, Predefined QC Location, and Pre-Interpolation of QC, and emits the two-section worksheet) — https://github.com/CSi-Studio/InjectionDesign
- **Excel** (Input format for importing sample information via the downloadable sample Excel template before position design)

## Examples

```
java -jar /YOUR_PAHT/InjectionDesignDeploy.jar  # then open http://localhost:8080/ to configure Predefined QC Location and export the worksheet
```

## Evaluation signals

- The exported worksheet contains both a well-position layout section and a separate injection-order section, confirming the two designs are decoupled
- All QC wells (Blank, Solvent QC, Long-Term Reference QC, etc.) lie exclusively within the single predefined plate column; no QC well appears among experimental sample wells
- Each experimental sample is assigned to a well within the configured plate load capacity (max experimental samples per plate, excluding QC)
- In the injection sequence, QC samples appear evenly pre-interpolated in a circular fashion among the randomized experimental samples rather than as one contiguous block
- The plate specification matches the configured template (12 columns labelled 1–12, 8 rows labelled a–h) and QC types are color-distinguished in the visualized layout

## Limitations

- New QC sample types and new injection rules may emerge that are not yet supported in the web service
- Block randomization completely averages out the distribution of sample features, losing natural numerical characteristics of the sample distribution; complete randomization is not a perfect solution for such scenarios
- Optimized block randomization can lead to local aggregation when there are too many groups in one dimension under some special numerical distributions
- Separation depends on instrument capability: the mass spectrometer must be able to customize the sampling/injection sequence independently of plate layout

## Evidence

- [intro] reduce operational complexity from QC interpenetration: "we can put QC samples in the same area of the left part to reduce the operational complexity caused by QC sample interpenetration. This can greatly reduce the preprocessing time and error rate."
- [intro] position/sequence separation: "The instrument can customize the sampling sequence independently so that the layout of samples on the plate is more standardized and humanized."
- [methods] QC types predefined: "InjectionDesign includes common QC sample types. Solvent QC Samples, Long Term Reference (LTR) QC Samples, Blank Samples, and so on."
- [readme] four-part workflow: "InjectionDesign predefined four QC type: Blank QC, Solvent QC, Pooled QC, Long-Term Reference QC. and one custom QC."
- [readme] output worksheet: "All the final sequence will display with a visualized way and users can download the worksheet one by one all download all the worksheet at one time with the "Export All" button."
