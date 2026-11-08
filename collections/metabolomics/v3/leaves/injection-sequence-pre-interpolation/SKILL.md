---
name: injection-sequence-pre-interpolation
description: Use when a randomized LC/GC-MS sample list must be converted into an
  instrument injection sequence where QC samples are evenly interspersed and QC wells
  stay confined to a dedicated plate column.
when_to_use_negative:
- The sample list has not yet been inter-batch balanced and intra-batch block randomized
  — QC pre-interpolation must operate on an already-randomized sequence
- The laboratory requires new QC sample types or injection rules not among InjectionDesign's
  predefined four QC types plus one custom QC
- The experiment exceeds the configured plate load capacity (e.g., max experimental
  samples per 96-well plate, excluding QC), so per-plate position design cannot hold
  the samples
tools:
- name: InjectionDesign
  role: Web service (or local JAR) that performs the Pre-Interpolation of QC step
    within Intra-Batch Position Design and emits the worksheet with decoupled layout
    and injection order
  repo: https://github.com/CSi-Studio/InjectionDesign
- name: Excel
  role: File format used to import sample information into the project before randomization
    and sequence design
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2023.02.26.530140
    title: 'InjectionDesign: Plate Design with Optimized Stratified Block Randomization
      for Modern LC/GC-MS-Based Sample Preparation'
  source_content_hash: sha256:013d34209b0afbe30e6206bd1a50730aa5c0b1020ec41f4809ff72bc502c1c22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/InjectionDesign
  tools:
  - Excel
  - InjectionDesign
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# injection-sequence-pre-interpolation

## Summary

Pre-interpolates QC sample types (Solvent QC, Long Term Reference QC, Blank, Pooled/custom QC) evenly in a circular fashion among already-randomized experimental samples to build the injection order, independently of the 96-well plate position layout in InjectionDesign.

## When to use

Use when a randomized LC/GC-MS sample list must be converted into an instrument injection sequence where QC samples are evenly interspersed and QC wells stay confined to a dedicated plate column.

## When NOT to use

- The sample list has not yet been inter-batch balanced and intra-batch block randomized — QC pre-interpolation must operate on an already-randomized sequence
- The laboratory requires new QC sample types or injection rules not among InjectionDesign's predefined four QC types plus one custom QC
- The experiment exceeds the configured plate load capacity (e.g., max experimental samples per 96-well plate, excluding QC), so per-plate position design cannot hold the samples

## Inputs

- Randomized (inter-batch balanced, intra-batch block-randomized) experimental sample list from InjectionDesign
- Sample metadata imported via the InjectionDesign sample Excel template
- Plate specification: x/y axis numbers and labels (e.g., 12x8, x-labels [1..12], y-labels [a..h])
- QC sample counts per type (Solvent QC, Long Term Reference QC, Blank, Pooled/custom QC)
- Plate load capacity (max experimental samples per plate, excluding QC)

## Outputs

- Injection sequence (run order) with QC samples pre-interpolated evenly among experimental samples
- Worksheet with separate well-position layout and injection-order sections (downloadable individually or via Export All)
- Visualized distribution sequence of the final ordering

## How to apply

After intra-batch block randomization has ordered the experimental samples, apply the Pre-Interpolation of QC step: reserve a predefined QC location (e.g., the first or last column of a 12x8 plate with x-labels [1..12] and y-labels [a..h]) so QC wells never interpenetrate experimental wells, then weave each QC type (Solvent QC, LTR QC, Blank, Pooled/custom QC) circularly and evenly into the randomized injection sequence. Because InjectionDesign separates position design from sequence design, well coordinates and injection order are emitted as two distinct sections of one worksheet, letting the instrument sample a standardized layout while the run order follows the interpolated sequence. Configure plate load capacity (max experimental samples per plate, excluding QC) before interpolation, and verify the result in the visualized distribution view before exporting via the worksheet/Export All step.

## Related tools

- **InjectionDesign** (Web service (or local JAR) that performs the Pre-Interpolation of QC step within Intra-Batch Position Design and emits the worksheet with decoupled layout and injection order) — https://github.com/CSi-Studio/InjectionDesign
- **Excel** (File format used to import sample information into the project before randomization and sequence design)

## Examples

```
java -jar /YOUR_PATH/InjectionDesignDeploy.jar  # then at http://localhost:8080 configure plate 12x8, QC column, and export the worksheet
```

## Evaluation signals

- The exported worksheet contains two distinct sections — well-position layout and injection order — confirming position/sequence decoupling
- All QC sample wells (Solvent QC, LTR QC, Blank) fall only within the predefined QC column and never interpenetrate experimental sample wells
- QC samples appear evenly distributed (circular pre-interpolation) throughout the injection sequence, not clustered
- With block size 2, no three consecutive samples of the same group occur in the sequence, matching the block-randomization invariant
- The visualized distribution sequence shown after each adjustment matches the downloaded worksheet order

## Limitations

- New QC sample types and injection rules may emerge that the web service does not yet support
- Block randomization averages out sample feature distribution, losing natural numerical characteristics; complete randomization is 'not a perfect solution' for such scenarios
- Optimized block randomization can produce local aggregation when there are too many groups in one dimension under special numerical distributions
- QC placement relies on predefined QC locations; only four predefined QC types plus one custom QC are supported

## Evidence

- [abstract] separation of position and sequence: "It offers the ability to separate the position design from the sequence design, making the output more compatible with the requirements of a modern mass spectrometer-based laboratory."
- [intro] QC area placement benefit: "we can put QC samples in the same area of the left part to reduce the operational complexity caused by QC sample interpenetration."
- [methods] QC sample types supported: "InjectionDesign includes common QC sample types. Solvent QC Samples, Long Term Reference (LTR) QC Samples, Blank Samples, and so on."
- [methods] filter step heading: "## Materials And Methods > Intra-Batch Position Design > Pre-Interpolation Of Qc"
- [readme] README step 2 QC predefinition: "InjectionDesign predefined four QC type: Blank QC, Solvent QC, Pooled QC, Long-Term Reference QC. and one custom QC."
