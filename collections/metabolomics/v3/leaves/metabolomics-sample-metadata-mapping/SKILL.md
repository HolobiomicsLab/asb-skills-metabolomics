---
name: metabolomics-sample-metadata-mapping
description: Use when a binned NMR data matrix (rows = bucket intensities over Bin.SpectraPoint
  spectral points per sample) must be annotated with sample class labels from sample
  metadata before export to MetaboAnalyst or Workflows4Metabolomics.
when_to_use_negative:
- If information about classes is not available along with the metabolomics data in
  the public repository, re-analysis oriented to classification problems is not feasible
  and this skill cannot be applied.
- If the input is raw, unbinned NMR acquisition files rather than a binned data matrix,
  the upstream bucketing step must run first; this skill operates on the binned matrix.
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_3520
tools:
- name: CloMet
  role: Performs the sample metadata mapping and the final output-format conversion
    of the binned data matrix for both MetaboAnalyst and Workflows4Metabolomics.
  repo: https://github.com/rmallol/clomet
- name: MetaboAnalyst
  role: Downstream web-based analysis platform that consumes the class-annotated CSV
    produced by this skill.
- name: Workflows4Metabolomics
  role: Downstream Galaxy-based workflow collection that consumes the three TABULAR
    files (dataMatrix, sampleMetadata, variableMetadata) produced by this skill.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:c2c712f1e574d011f274abd73f14625daddc0ea1b0d63965a94a0232cb7a0654
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/rmallol/clomet
  tools:
  - CloMet
  - MetaboAnalyst
  - Workflows4Metabolomics
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# metabolomics-sample-metadata-mapping

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Mapping sample class metadata onto binned NMR metabolomics data so that downstream analysis platforms (MetaboAnalyst, Workflows4Metabolomics) can run classification or statistical analyses on re-harvested public data sets.

## When to use

Use when a binned NMR data matrix (rows = bucket intensities over Bin.SpectraPoint spectral points per sample) must be annotated with sample class labels from sample metadata before export to MetaboAnalyst or Workflows4Metabolomics.

## When NOT to use

- If information about classes is not available along with the metabolomics data in the public repository, re-analysis oriented to classification problems is not feasible and this skill cannot be applied.
- If the input is raw, unbinned NMR acquisition files rather than a binned data matrix, the upstream bucketing step must run first; this skill operates on the binned matrix.

## Inputs

- Binned NMR data matrix with bucket intensities over Bin.SpectraPoint spectral points per sample (output of upstream NMR bucketing step)
- Sample metadata table with class annotations for each sample

## Outputs

- MetaboAnalyst CSV (rows = samples; col 1 = sample name; col 2 = class; remaining cols = Bin.SpectraPoint)
- Workflows4Metabolomics dataMatrix TABULAR (samples-by-variables, tab-separated)
- Workflows4Metabolomics sampleMetadata TABULAR (sample names + class annotations)
- Workflows4Metabolomics variableMetadata TABULAR (Bin.SpectraPoint bucket variable descriptions)

## How to apply

Start from the binned data matrix produced by the upstream NMR bucketing step, in which each row holds bucket intensities over Bin.SpectraPoint spectral points per sample. Pull the class labels from the study's sample metadata and attach them to each sample row; if the class column is absent from the public repository the re-analysis is not feasible and this skill cannot proceed. Transpose or reshape the annotated matrix so rows are samples and write the first column as sample name, the second as class, and name the remaining columns Bin.SpectraPoint to form the MetaboAnalyst CSV. For Workflows4Metabolomics, emit the dataMatrix TABULAR in samples-by-variables layout, the sampleMetadata TABULAR carrying sample names and class annotations, and the variableMetadata TABULAR describing the Bin.SpectraPoint bucket variables. Validate that all four output files exist, are tabular in the required format, and that their row/column dimensions and headers match the binned input matrix (file_exists, file_format_is, row_count_equals checks), deferring any residual format nuance to expert_review.

## Related tools

- **CloMet** (Performs the sample metadata mapping and the final output-format conversion of the binned data matrix for both MetaboAnalyst and Workflows4Metabolomics.) — https://github.com/rmallol/clomet
- **MetaboAnalyst** (Downstream web-based analysis platform that consumes the class-annotated CSV produced by this skill.)
- **Workflows4Metabolomics** (Downstream Galaxy-based workflow collection that consumes the three TABULAR files (dataMatrix, sampleMetadata, variableMetadata) produced by this skill.)

## Examples

```
docker run rmallol/clomet --study-id <ID>
```

## Evaluation signals

- All four output files (MetaboAnalyst CSV, dataMatrix, sampleMetadata, variableMetadata) exist after the run.
- Each output file passes a file_format_is check confirming it is tabular in the required layout (CSV for MetaboAnalyst; TABULAR for Workflows4Metabolomics).
- Row/column dimensions of each output match the binned input matrix (row_count_equals check).
- Headers of the MetaboAnalyst CSV show sample name in column 1, class in column 2, and Bin.SpectraPoint for all remaining columns.
- Every sample row in the MetaboAnalyst CSV and in sampleMetadata carries the class label from the study metadata; any missing class means the classification re-analysis was not feasible.

## Limitations

- Re-analysis oriented to classification problems is infeasible when class metadata is not available along with the metabolomics data in the public repository.
- CloMet itself does not implement data processing algorithms; extension with processing algorithms is left to other developers.
- Maintenance and extension of CloMet depend on the open-source developer community.

## Evidence

- [full_text] CloMet's final transformation produces, for MetaboAnalyst, a csv where each row is a sample, the first column holds sample names, the second the sample class, and remaining columns hold spectral points headed as Bin.SpectraPoint: [evidence span withheld — anchor status: not_found]
- [full_text] for Workflow4Metabolomics it produces three .TABULAR files: a transposed data matrix, a sample metadata table ordering samples, and a variable metadata table ordering spectral features.: [evidence span withheld — anchor status: not_found]
- [discussion] if information about classes is not available along with the metabolomics data in public repositories, re-analysis oriented to classification problems won't be feasible.: [evidence span withheld — anchor status: not_found]
