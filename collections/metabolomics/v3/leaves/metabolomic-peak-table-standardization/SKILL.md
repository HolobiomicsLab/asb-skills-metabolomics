---
name: metabolomic-peak-table-standardization
description: Use when a raw or tool-generated metabolomic peak table (e.g., from MetaboLights
  MTBLS403, MTBLS6, MTBLS656) must be reshaped to NOREVA's unified format before running
  the R-based workflow scan and ranking.
when_to_use_negative:
- The peak table already satisfies NOREVA's unified format (rows = samples by name,
  columns preserved), in which case it can be loaded directly without re-standardization
- You need raw-spectra feature detection/alignment rather than reshaping an existing
  peak table — use the upstream peak-table preparation tools (XCMS, NMRProc Flow)
  instead
- The goal is statistical analysis or biomarker discovery on an already-processed
  table (MetaboAnalyst, W4M territory) rather than NOREVA pre-processing workflow
  optimization
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: NOREVA
  role: Consumes the standardized peak table via PrepareInuputFiles, which accepts
    dataformat='1' (NOREVA standardized format) or '2' (customized format from 12
    tools)
  repo: https://github.com/idrblab/NOREVA
- name: Proteowizard
  role: Converts proprietary raw instrument files to an open format before peak-table
    construction and standardization
- name: XCMS
  role: Prepares the metabolomic peak table that is then manually standardized to
    the unified NOREVA format
- name: nmrML
  role: Open-format conversion target for NMR raw files prior to peak-table standardization
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s41596-021-00636-9
    title: Optimization of metabolomic data processing using NOREVA
  source_content_hash: sha256:f4d41f1f1c9f3ef1b6fc7eb5db492f17361e8460dc01171db4953c0f0c6fc9eb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idrblab/NOREVA
  tools:
  - NOREVA
  - Proteowizard
  - R
  - XCMS
  - nmrML
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# metabolomic-peak-table-standardization

## Summary

Reformats a metabolomic peak table into NOREVA's unified input format — preserving rows as samples (by name) and columns as metabolites — so that the four-stage processing workflow scan (S1 filtering/imputation through S4 normalization) and S5 performance assessment can run on it.

## When to use

Use when a raw or tool-generated metabolomic peak table (e.g., from MetaboLights MTBLS403, MTBLS6, MTBLS656) must be reshaped to NOREVA's unified format before running the R-based workflow scan and ranking.

## When NOT to use

- The peak table already satisfies NOREVA's unified format (rows = samples by name, columns preserved), in which case it can be loaded directly without re-standardization
- You need raw-spectra feature detection/alignment rather than reshaping an existing peak table — use the upstream peak-table preparation tools (XCMS, NMRProc Flow) instead
- The goal is statistical analysis or biomarker discovery on an already-processed table (MetaboAnalyst, W4M territory) rather than NOREVA pre-processing workflow optimization

## Inputs

- Raw metabolomic data files downloaded from a MetaboLights accession (e.g., MTBLS403, MTBLS6) or proprietary instrument files convertible via Proteowizard/nmrML
- Peak table file from one of 12 supported processing tools (e.g., XCMS, NMRProc Flow) or already in NOREVA standardized format
- Label file naming samples' class or time-point membership for time-course/multi-class studies
- Column indices of internal-standard (IS) metabolites, if present

## Outputs

- A NOREVA-format peak table (rows = samples by name, columns = metabolites) accepted by PrepareInuputFiles
- Standardized input object that can be fed directly to NOREVA assessment functions (normulticlassqcall, nortimecourseqcall, normulticlassisall, etc.)

## How to apply

Download the raw data files of the benchmark study (e.g., MTBLS403 from https://www.ebi.ac.uk/metabolights/MTBLS403/files) and manually standardize the peak table so that rows are the samples (provided by name) and columns are preserved as-is; this manual step is required because raw data files are organized in diverse ways. If starting from proprietary instrument files, first convert to an open format with tools such as Proteowizard or nmrML, or prepare the peak table with XCMS or NMRProc Flow. Load the standardized table into NOREVA via PrepareInuputFiles, setting dataformat='1' for NOREVA's standardized peak-table format or dataformat='2' for a customized format produced by one of the 12 supported software tools, and pass a label file for time-course/multi-class studies. If internal standards (IS) are present, record their column numbers (e.g., '2,6,9,n' means columns 2, 6, 9 and n are IS metabolites) since functions like normulticlassisall require them. The rationale is that NOREVA's S1–S4 processing stages and S5 multi-criteria assessment assume this exact row/column layout; preserving it is a prerequisite for every downstream workflow combination.

## Related tools

- **NOREVA** (Consumes the standardized peak table via PrepareInuputFiles, which accepts dataformat='1' (NOREVA standardized format) or '2' (customized format from 12 tools)) — https://github.com/idrblab/NOREVA
- **Proteowizard** (Converts proprietary raw instrument files to an open format before peak-table construction and standardization)
- **XCMS** (Prepares the metabolomic peak table that is then manually standardized to the unified NOREVA format)
- **nmrML** (Open-format conversion target for NMR raw files prior to peak-table standardization)

## Examples

```
PrepareInuputFiles(dataformat = "1", rawdata = "MTBLS403_peaktable.csv", label = "MTBLS403_labels.csv")
```

## Evaluation signals

- The standardized file loads without error through PrepareInuputFiles with dataformat='1' (standardized format)
- Row identities match the sample names in the raw data and the column set is preserved exactly (no reordering or dropping of metabolite columns)
- For datasets with internal standards, the IS column indices passed to normulticlassisall (e.g., '2,6,9,n') correctly point to the spiked/reference metabolites such as aspartic acid and malic acid in MTBLS6
- Downstream NOREVA functions (e.g., normulticlassqcall, nortimecourseqcall) execute over the full S1–S5 scan on the standardized table without input-format failures

## Limitations

- Manual standardization is relatively time consuming because preservation of row/column is challenging due to diverse organizations of the studied raw data files
- Successful application depends on upfront experimental design — adoption of both QCSs and ISs — which NOREVA does not enforce during data preparation
- Users need preliminary R programming experience; the standardization/ preparation step itself must be done by hand rather than automatically

## Evidence

- [discussion] Manually standardize a peak table file of the unified format by preserving the rows (samples provided by name) and columns: [evidence span withheld — anchor status: not_found]
- [discussion] Preservation of row/column is challenging due to diverse organizations of the studied raw data files, which makes this step relatively time consuming: [evidence span withheld — anchor status: not_found]
- [intro] tools have been constructed to convert the raw file of proprietary format to an open format (e.g., Proteowizard, nmrML): [evidence span withheld — anchor status: not_found]
