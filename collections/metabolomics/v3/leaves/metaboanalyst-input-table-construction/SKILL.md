---
name: metaboanalyst-input-table-construction
description: Use when a binned NMR bucket-intensity matrix (rows = samples, columns
  = Bin.SpectraPoint buckets) plus sample class labels must be converted into MetaboAnalyst-
  or Workflows4Metabolomics-ready files.
when_to_use_negative:
- The data are already in MetaboAnalyst or Workflows4Metabolomics format (dataMatrix/sampleMetadata/variableMetadata),
  so reconversion risks mangling a valid table
- Class information is not available with the metabolomics data in the public repository
  — re-analysis oriented to classification problems won't be feasible, and the class
  column cannot be populated
- The goal is to implement data processing algorithms on the spectra — the article
  states this is out of scope for CloMet
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: CloMet
  role: Performs the final transformation that emits the MetaboAnalyst CSV and the
    three Workflows4Metabolomics TABULAR files from the binned matrix
  repo: github.com/rmallol/clomet
- name: MetaboAnalyst
  role: Consumer of the emitted sample-by-variables CSV (sample name + class + Bin.SpectraPoint
    columns) for downstream statistical analysis
- name: Workflows4Metabolomics
  role: Consumer of the dataMatrix/sampleMetadata/variableMetadata TABULAR trio; its
    NMR_Read, NMR_Preprocessing, and NMR_Bucketing tools define the upstream tool
    format the output must match
- name: Docker
  role: Deployment mechanism for running CloMet to generate the converted output files
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:79ed58273a91ca82b269ca0c35be1d4722801a9acf70e3b959447f4a68bc4d04
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - CloMet
  - Docker
  - MetaboAnalyst
  - Workflows4Metabolomics
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# metaboanalyst-input-table-construction

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Reshapes the binned NMR data matrix produced by upstream bucketing into the exact input file formats required by MetaboAnalyst and Workflow4Metabolomics, so repository-derived NMR metabolomics data can be analyzed without manual reformatting.

## When to use

Use when a binned NMR bucket-intensity matrix (rows = samples, columns = Bin.SpectraPoint buckets) plus sample class labels must be converted into MetaboAnalyst- or Workflows4Metabolomics-ready files.

## When NOT to use

- The data are already in MetaboAnalyst or Workflows4Metabolomics format (dataMatrix/sampleMetadata/variableMetadata), so reconversion risks mangling a valid table
- Class information is not available with the metabolomics data in the public repository — re-analysis oriented to classification problems won't be feasible, and the class column cannot be populated
- The goal is to implement data processing algorithms on the spectra — the article states this is out of scope for CloMet

## Inputs

- Binned NMR data matrix from the upstream bucketing step (bucket intensities per sample over Bin.SpectraPoint spectral points)
- Sample metadata with study class labels
- Metabolomics data retrieved via study id from MetaboLights or Metabolomics Workbench (optionally harmonized by CloMet's custom directory-based format)

## Outputs

- MetaboAnalyst CSV: one row per sample; column 1 = sample name, column 2 = sample class, remaining columns = Bin.SpectraPoint spectral points
- Workflows4Metabolomics dataMatrix TABULAR file (samples-by-variables, tab-separated)
- Workflows4Metabolomics sampleMetadata TABULAR file (sample names and class annotations)
- Workflows4Metabolomics variableMetadata TABULAR file (Bin.SpectraPoint bucket variables)

## How to apply

Load the binned data matrix produced by the upstream NMR bucketing step (e.g. NMR_Bucketing in Workflows4Metabolomics), in which each sample row holds bucket intensities over Bin.SpectraPoint spectral points. Attach the study class labels from the sample metadata to each sample row; if class information is absent in the source repository, classification-oriented re-analysis is not feasible, so flag this early. For MetaboAnalyst, transpose/reshape so rows are samples, write the first column as sample name, the second column as class, name all remaining columns Bin.SpectraPoint, and save as CSV. For Workflows4Metabolomics, emit three tab-separated .TABULAR files: dataMatrix in samples-by-variables layout, sampleMetadata carrying sample names and class annotations, and variableMetadata describing the Bin.SpectraPoint bucket variables. Validate that all four files exist, are tabular, and that row/column dimensions and headers match the binned input matrix.

## Related tools

- **CloMet** (Performs the final transformation that emits the MetaboAnalyst CSV and the three Workflows4Metabolomics TABULAR files from the binned matrix) — github.com/rmallol/clomet
- **MetaboAnalyst** (Consumer of the emitted sample-by-variables CSV (sample name + class + Bin.SpectraPoint columns) for downstream statistical analysis)
- **Workflows4Metabolomics** (Consumer of the dataMatrix/sampleMetadata/variableMetadata TABULAR trio; its NMR_Read, NMR_Preprocessing, and NMR_Bucketing tools define the upstream tool format the output must match)
- **Docker** (Deployment mechanism for running CloMet to generate the converted output files)

## Evaluation signals

- file_exists: all four output files (MetaboAnalyst CSV + three W4M TABULAR files) are present after conversion
- file_format_is: each output is tabular in the required format (CSV for MetaboAnalyst, tab-separated .TABULAR for W4M)
- row_count_equals: row/column dimensions of the outputs match the binned input matrix
- Header check: MetaboAnalyst CSV has sample name in column 1, class in column 2, and Bin.SpectraPoint headers on all remaining columns; W4M dataMatrix is in samples-by-variables layout
- sampleMetadata class annotations agree with the class labels attached from the source sample metadata

## Limitations

- If information about classes is not available along with the metabolomics data in public repositories, re-analysis oriented to classification problems won't be feasible
- CloMet itself does not implement data processing algorithms; any residual format nuance beyond the existence/format/dimension checks is deferred to expert review
- Integration with existing analysis platforms remains a future plan, not implemented

## Evidence
