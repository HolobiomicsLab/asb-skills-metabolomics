---
name: spectral-bucket-variable-labelling
description: Use when a binned NMR bucket intensity matrix (rows of bucket intensities
  over Bin.SpectraPoint spectral points per sample) plus sample class metadata must
  be converted into MetaboAnalyst or Workflow4Metabolomics input files.
when_to_use_negative:
- Input is raw NMR acquisition files rather than an already-binned bucket matrix —
  run NMR_Read/NMR_Bucketing upstream first, since CloMet does not implement data
  processing algorithms
- The target platform is neither MetaboAnalyst nor Workflow4Metabolomics and expects
  a different native format
- Class metadata is unavailable in the source repository, so the labelling step cannot
  produce a class column for classification-oriented re-analysis
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CloMet
  role: 'Performs the transformation: harmonizes repository file systems and emits
    the MetaboAnalyst CSV and the three Workflow4Metabolomics TABULAR files from the
    binned matrix'
  repo: github.com/rmallol/clomet
- name: MetaboAnalyst
  role: Downstream target platform whose expected CSV layout (sample rows, sample/class
    first two columns, Bin.SpectraPoint spectral-point headers) defines the output
    schema
- name: Workflows4Metabolomics
  role: Downstream target platform whose three-file TABULAR convention (dataMatrix,
    sampleMetadata, variableMetadata) defines the alternative output schema
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:ca05b4d86e310824331f014df8fe614694fba4c851af737e96f2d7fe98f89f3f
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - CloMet
  - MetaboAnalyst
  - Workflows4Metabolomics
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# spectral-bucket-variable-labelling

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Reshapes and labels a binned NMR bucket data matrix into the exact input file formats required by MetaboAnalyst (CSV with Bin.SpectraPoint columns) and Workflow4Metabolomics (dataMatrix, sampleMetadata, variableMetadata TABULAR files), closing the gap between repository data and analysis platforms.

## When to use

Use when a binned NMR bucket intensity matrix (rows of bucket intensities over Bin.SpectraPoint spectral points per sample) plus sample class metadata must be converted into MetaboAnalyst or Workflow4Metabolomics input files.

## When NOT to use

- Input is raw NMR acquisition files rather than an already-binned bucket matrix — run NMR_Read/NMR_Bucketing upstream first, since CloMet does not implement data processing algorithms
- The target platform is neither MetaboAnalyst nor Workflow4Metabolomics and expects a different native format
- Class metadata is unavailable in the source repository, so the labelling step cannot produce a class column for classification-oriented re-analysis

## Inputs

- Binned NMR bucket data matrix (rows = bucket intensities over Bin.SpectraPoint spectral points, one series per sample) from the upstream bucketing step
- Sample metadata table carrying study class labels per sample

## Outputs

- MetaboAnalyst CSV: rows = samples, column 1 = sample name, column 2 = sample class, remaining columns headed Bin.SpectraPoint
- Workflow4Metabolomics dataMatrix .TABULAR file (samples-by-variables, tab-separated)
- Workflow4Metabolomics sampleMetadata .TABULAR file (sample names and class annotations)
- Workflow4Metabolomics variableMetadata .TABULAR file (Bin.SpectraPoint bucket variables)

## How to apply

Load the binned data matrix produced by the upstream NMR bucketing step, where each row holds bucket intensities over Bin.SpectraPoint spectral points per sample, and attach the study class labels from the sample metadata. For MetaboAnalyst, reshape so each row is a sample: write the first column as sample name, the second as sample class, and header all remaining spectral-point columns as Bin.SpectraPoint, saving as CSV. For Workflow4Metabolomics, emit three tab-separated .TABULAR files: a samples-by-variables transposed dataMatrix, a sampleMetadata table ordering samples with class annotations, and a variableMetadata table describing the Bin.SpectraPoint bucket variables. Rationale: public repository data lack standardized file-system formats, so this harmonization is what makes downstream tools such as NMR_Read, NMR_Preprocessing, and NMR_Bucketing in Workflows4Metabolomics actually runnable.

## Related tools

- **CloMet** (Performs the transformation: harmonizes repository file systems and emits the MetaboAnalyst CSV and the three Workflow4Metabolomics TABULAR files from the binned matrix) — github.com/rmallol/clomet
- **MetaboAnalyst** (Downstream target platform whose expected CSV layout (sample rows, sample/class first two columns, Bin.SpectraPoint spectral-point headers) defines the output schema)
- **Workflows4Metabolomics** (Downstream target platform whose three-file TABULAR convention (dataMatrix, sampleMetadata, variableMetadata) defines the alternative output schema)

## Evaluation signals

- file_exists check: all four output files (MetaboAnalyst CSV plus dataMatrix, sampleMetadata, variableMetadata TABULAR files) are present
- file_format_is check: the three Workflow4Metabolomics files are tabular/tab-separated as required
- row_count_equals check: row/column dimensions of each output match the binned input matrix
- Header check: all non-class spectral columns in the MetaboAnalyst CSV are headed Bin.SpectraPoint
- Transpose check: the W4M dataMatrix is in samples-by-variables layout and sampleMetadata sample order matches the dataMatrix

## Limitations

- If information about classes is not available along with the metabolomics data in public repositories, re-analysis oriented to classification problems will not be feasible (no class column can be labelled)
- CloMet itself does not implement data processing algorithms; bucketing/processing quality is inherited from upstream tools and out of scope
- Any residual format nuance beyond the existence/format/row-count checks is deferred to expert review of the target platform's parser

## Evidence

- [other] MetaboAnalyst CSV layout with Bin.SpectraPoint columns and W4M three-file convention: [evidence span withheld — anchor status: not_found]
- [other] W4M output files and validation checks: [evidence span withheld — anchor status: not_found]
