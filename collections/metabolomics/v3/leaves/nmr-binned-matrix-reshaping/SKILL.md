---
name: nmr-binned-matrix-reshaping
description: Use when a binned NMR matrix (rows of bucket intensities over Bin.SpectraPoint
  per sample) must be converted into MetaboAnalyst CSV or Workflows4Metabolomics .TABULAR
  input files for downstream statistical analysis.
when_to_use_negative:
- The data are not NMR-based metabolomics data from MetaboLights or Metabolomics Workbench
  (CloMet's harmonization focuses on NMR-based metabolomics)
- The data are already in a MetaboAnalyst CSV or W4M dataMatrix/sampleMetadata/variableMetadata
  layout, so reshaping would be redundant
- The goal is to apply data processing algorithms rather than convert file formats
  — CloMet explicitly leaves processing out of scope
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0593
tools:
- name: CloMet
  role: Performs the final transformation of the binned matrix into the MetaboAnalyst
    CSV and the three Workflows4Metabolomics .TABULAR files; available as a Docker
    file and via GitHub
  repo: https://github.com/rmallol/clomet
- name: MetaboAnalyst
  role: Target analysis platform whose required CSV layout (samples as rows, class
    column, Bin.SpectraPoint headers) the reshape must match
- name: Workflows4Metabolomics
  role: Target workflow platform whose required dataMatrix, sampleMetadata, and variableMetadata
    .TABULAR formats the outputs must conform to
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:98d2ceb9ebc4b156b593befcd8f1c79478e6b4d349b9518321258a6a7235f33e
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
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# nmr-binned-matrix-reshaping

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Reshape the binned NMR data matrix produced by upstream bucketing into the exact input formats required by MetaboAnalyst (samples-by-variables CSV) and Workflows4Metabolomics (three .TABULAR files), so repository data can be analyzed without manual reformatting.

## When to use

Use when a binned NMR matrix (rows of bucket intensities over Bin.SpectraPoint per sample) must be converted into MetaboAnalyst CSV or Workflows4Metabolomics .TABULAR input files for downstream statistical analysis.

## When NOT to use

- The data are not NMR-based metabolomics data from MetaboLights or Metabolomics Workbench (CloMet's harmonization focuses on NMR-based metabolomics)
- The data are already in a MetaboAnalyst CSV or W4M dataMatrix/sampleMetadata/variableMetadata layout, so reshaping would be redundant
- The goal is to apply data processing algorithms rather than convert file formats — CloMet explicitly leaves processing out of scope

## Inputs

- Binned NMR data matrix (rows of bucket intensities over Bin.SpectraPoint spectral points, one sample per column or row) from the upstream NMR bucketing step
- Sample metadata table with study class labels

## Outputs

- MetaboAnalyst CSV: rows = samples, column 1 = sample names, column 2 = sample class, remaining columns = spectral points headed Bin.SpectraPoint
- Workflows4Metabolomics dataMatrix .TABULAR file (samples-by-variables, tab-separated)
- Workflows4Metabolomics sampleMetadata .TABULAR file (sample names and class annotations)
- Workflows4Metabolomics variableMetadata .TABULAR file (Bin.SpectraPoint bucket variables)

## How to apply

Load the binned data matrix from the upstream NMR bucketing step, where each row holds bucket intensities over Bin.SpectraPoint spectral points per sample. Attach the study class labels from the sample metadata to each sample row. For MetaboAnalyst, transpose/reshape so rows are samples, write the first column as sample name, the second as class, and header all remaining columns as Bin.SpectraPoint, saving as CSV. For Workflows4Metabolomics, emit three tab-separated .TABULAR files: a samples-by-variables dataMatrix, a sampleMetadata table ordering samples with class annotations, and a variableMetadata table describing the Bin.SpectraPoint bucket variables. The rationale is that public repositories lack file-system standardization, so harmonizing into these platform-specific layouts is what makes the data directly reusable.

## Related tools

- **CloMet** (Performs the final transformation of the binned matrix into the MetaboAnalyst CSV and the three Workflows4Metabolomics .TABULAR files; available as a Docker file and via GitHub) — https://github.com/rmallol/clomet
- **MetaboAnalyst** (Target analysis platform whose required CSV layout (samples as rows, class column, Bin.SpectraPoint headers) the reshape must match)
- **Workflows4Metabolomics** (Target workflow platform whose required dataMatrix, sampleMetadata, and variableMetadata .TABULAR formats the outputs must conform to)

## Evaluation signals

- All four output files exist (file_exists checks on the MetaboAnalyst CSV and the three .TABULAR files)
- Each output passes a file_format_is check: CSV for MetaboAnalyst, tab-separated tabular for the W4M files
- row_count_equals checks confirm output dimensions match the binned input matrix (one row per sample; one variableMetadata entry per Bin.SpectraPoint bucket)
- MetaboAnalyst CSV headers: first column = sample names, second = class, remaining columns named Bin.SpectraPoint
- W4M dataMatrix is in the samples-by-variables layout and sampleMetadata orders the same samples as the dataMatrix

## Limitations

- If information about classes is not available with the metabolomics data in public repositories, re-analysis oriented to classification problems won't be feasible
- CloMet does not implement data processing algorithms; format conversion only — processing is left to other developers
- Maintenance and extension of CloMet rely on the open-source developer community; residual format nuances may require expert_review beyond automated checks

## Evidence

- [discussion] if information about classes is not available along with the metabolomics data in public repositories, re-analysis oriented to classification problems won't be feasible.: [evidence span withheld — anchor status: not_found]
