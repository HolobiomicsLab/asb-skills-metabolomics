---
name: workflow4metabolomics-tabular-export
description: Use when a binned NMR bucket-intensity matrix (rows = buckets over Bin.SpectraPoint
  spectral points, per sample) from MetaboLights or Metabolomics Workbench must be
  reshaped into Workflow4Metabolomics .TABULAR or MetaboAnalyst CSV inputs.
when_to_use_negative:
- Input is already a formatted feature table compatible with MetaboAnalyst or Workflows4Metabolomics
  — no conversion is needed
- Raw, unbucketed NMR spectra that still require NMR_Preprocessing and NMR_Bucketing
  before export
- Classification-oriented re-analysis when class information is not available alongside
  the metabolomics data in the repository — the export cannot supply missing class
  metadata
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CloMet
  role: Performs the final transformation of the binned matrix into the W4M .TABULAR
    files and MetaboAnalyst CSV
  repo: https://github.com/rmallol/clomet
- name: Workflows4Metabolomics
  role: Target analysis platform whose NMR_Read, NMR_Preprocessing, and NMR_Bucketing
    tools consume the exported TABULAR files
- name: MetaboAnalyst
  role: Target web-based platform that consumes the exported samples-by-rows CSV
- name: Docker
  role: Distribution container through which CloMet is available to run the conversion
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:b097749d6cd6531aafb4efa03727ec51f6711cac0e3c008b6e08f3efcfa4ae56
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/rmallol/clomet
  tools:
  - CloMet
  - Docker
  - MetaboAnalyst
  - Workflows4Metabolomics
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# workflow4metabolomics-tabular-export

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Export CloMet's harmonized NMR binned data matrix into the three .TABULAR files required by Workflow4Metabolomics (dataMatrix, sampleMetadata, variableMetadata), plus the samples-by-rows MetaboAnalyst CSV, so repository data can be analyzed directly in these platforms.

## When to use

Use when a binned NMR bucket-intensity matrix (rows = buckets over Bin.SpectraPoint spectral points, per sample) from MetaboLights or Metabolomics Workbench must be reshaped into Workflow4Metabolomics .TABULAR or MetaboAnalyst CSV inputs.

## When NOT to use

- Input is already a formatted feature table compatible with MetaboAnalyst or Workflows4Metabolomics — no conversion is needed
- Raw, unbucketed NMR spectra that still require NMR_Preprocessing and NMR_Bucketing before export
- Classification-oriented re-analysis when class information is not available alongside the metabolomics data in the repository — the export cannot supply missing class metadata

## Inputs

- Binned NMR data matrix from the upstream NMR bucketing step (bucket intensities over Bin.SpectraPoint spectral points per sample)
- Sample metadata with class labels from the source repository (MetaboLights / Metabolomics Workbench)

## Outputs

- MetaboAnalyst CSV: rows = samples; column 1 = sample name, column 2 = class, remaining columns = Bin.SpectraPoint spectral points
- Workflow4Metabolomics dataMatrix .TABULAR file (samples-by-variables, tab-separated)
- Workflow4Metabolomics sampleMetadata .TABULAR file (sample names and class annotations)
- Workflow4Metabolomics variableMetadata .TABULAR file (Bin.SpectraPoint bucket variable descriptions)

## How to apply

Load the binned data matrix produced by the upstream NMR bucketing step and attach the study class labels from the sample metadata to each sample. For MetaboAnalyst, transpose/reshape so rows are samples, write sample names in the first column, class in the second, and name all remaining columns Bin.SpectraPoint, saving as CSV. For Workflow4Metabolomics, emit three tab-separated .TABULAR files: a samples-by-variables dataMatrix, a sampleMetadata table carrying sample names and class annotations, and a variableMetadata table describing the Bin.SpectraPoint bucket variables. Rationale: public repository raw data lack file-system standardization, so CloMet harmonizes heterogeneous repository layouts into its custom directory-based internal standard before this final format conversion, letting users plug output directly into tools like NMR_Read, NMR_Preprocessing, and NMR_Bucketing in Workflows4Metabolomics.

## Related tools

- **CloMet** (Performs the final transformation of the binned matrix into the W4M .TABULAR files and MetaboAnalyst CSV) — https://github.com/rmallol/clomet
- **Workflows4Metabolomics** (Target analysis platform whose NMR_Read, NMR_Preprocessing, and NMR_Bucketing tools consume the exported TABULAR files)
- **MetaboAnalyst** (Target web-based platform that consumes the exported samples-by-rows CSV)
- **Docker** (Distribution container through which CloMet is available to run the conversion)

## Evaluation signals

- All four output files (MetaboAnalyst CSV + three W4M .TABULAR files) exist at the expected paths (file_exists)
- Output files are tabular in the required format (file_format_is)
- Row/column dimensions of the exported matrices match the binned input matrix (row_count_equals)
- MetaboAnalyst CSV headers: first column sample names, second column class, remaining columns headed Bin.SpectraPoint
- W4M dataMatrix is in samples-by-variables layout with tab-separated values consistent with the input matrix

## Limitations

- If class information is not available with the data in the public repository, re-analysis oriented to classification problems is not feasible — the sampleMetadata export cannot fabricate class labels
- Any residual format nuance not caught by file_exists / file_format_is / row_count_equals checks is deferred to expert_review
- CloMet does not implement data processing algorithms (e.g., bucketing itself); it only converts and harmonizes formats

## Evidence

- [discussion] if information about classes is not available along with the metabolomics data in public repositories, re-analysis oriented to classification problems won't be feasible.: [evidence span withheld — anchor status: not_found]
