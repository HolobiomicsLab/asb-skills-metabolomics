---
name: massive-deposit-file-listing-parsing
description: Use when a MassIVE accession's file listing must be enumerated to compare
  byte sizes of original raw MS-files against PreProcessor-generated files (suffix
  "_MA-d3-c3-Min20_SR") in order to quantify a size-reduction ratio.
when_to_use_negative:
- Input is already a feature table or lipid annotation result — the paper's annotation-gain
  metrics (19.4% more annotations, 70.7% more features) are out of scope here
- The goal is measuring downstream runtime or memory rather than storage size (the
  34 min → 7 min and 6 GB → 3 GB Lipid Annotator comparisons need benchmarking, not
  file listing)
- The deposit lacks paired original/PreProcessor-generated files, so no size-reduction
  ratio can be computed
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: PNNL PreProcessor
  role: Generated the preprocessed MS-files (with intensity threshold of 20 counts,
    smoothing, saturation repair) whose byte sizes are compared against the originals
    in the MassIVE listing
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
- name: R
  role: Assembles the sizes table from the deposit file listing, sums byte sizes per
    group, computes the reduction ratio, and writes the TSV output
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:ce483d0603a3b7976e015abda244dde72d333bfa216ff799bc55e9c19f5097ed
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PNNL PreProcessor
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# massive-deposit-file-listing-parsing

## Summary

Parses the file listing of a MassIVE deposit (e.g. MSV000087295, the human plasma lipid extract LC-DT IM-MS triplicates from the PNNL PreProcessor paper) to pair original raw MS-files with PreProcessor-generated files and compute a byte-size reduction ratio quantifying preprocessing storage savings.

## When to use

Use when a MassIVE accession's file listing must be enumerated to compare byte sizes of original raw MS-files against PreProcessor-generated files (suffix "_MA-d3-c3-Min20_SR") in order to quantify a size-reduction ratio.

## When NOT to use

- Input is already a feature table or lipid annotation result — the paper's annotation-gain metrics (19.4% more annotations, 70.7% more features) are out of scope here
- The goal is measuring downstream runtime or memory rather than storage size (the 34 min → 7 min and 6 GB → 3 GB Lipid Annotator comparisons need benchmarking, not file listing)
- The deposit lacks paired original/PreProcessor-generated files, so no size-reduction ratio can be computed

## Inputs

- MassIVE deposit accession (MSV000087295) with its file listing
- 3 original raw LC-DT IM-MS MS-files from the human plasma lipid extract triplicates
- 3 PreProcessor-generated MS-files with the "_MA-d3-c3-Min20_SR" suffix

## Outputs

- Sizes table (TSV) with columns: original file, preprocessed file, byte sizes
- Single numeric size-reduction ratio (preprocessed total / original total), with 0 < ratio < 1

## How to apply

Access the MassIVE deposit (MSV000087295) and enumerate the deposited files: 3 original raw lipid-extract MS-files from the LC-DT IM-MS triplicate runs and 3 corresponding PreProcessor-generated MS-files identifiable by the "_MA-d3-c3-Min20_SR" suffix (encoding demultiplexing/smoothing parameters and the 20-count minimum intensity threshold). Extract the byte size of each of the 6 files and assemble a sizes table with columns for original file, preprocessed file, and their byte sizes. Sum each group and compute the size-reduction ratio as (preprocessed total bytes)/(original total bytes). Validate that the ratio is a single numeric value strictly between 0 and 1, mirroring the paper's claim that preprocessing reduces file sizes (e.g. the proteomics demultiplexing file shrank from 20 GB to 7.68 GB). Save the sizes table and the ratio to a TSV for downstream reporting, using R or an equivalent scripting tool.

## Related tools

- **PNNL PreProcessor** (Generated the preprocessed MS-files (with intensity threshold of 20 counts, smoothing, saturation repair) whose byte sizes are compared against the originals in the MassIVE listing) — https://omics.pnl.gov/software/pnnl-preprocessor
- **R** (Assembles the sizes table from the deposit file listing, sums byte sizes per group, computes the reduction ratio, and writes the TSV output)

## Examples

```
Rscript -e 'd<-read.delim("sizes.tsv");r<-sum(d$preprocessed_bytes)/sum(d$original_bytes);cat(r,"\n")'
```

## Evaluation signals

- The computed size-reduction ratio is a single numeric value strictly between 0 and 1
- The sizes table matches the deposited file listings in MSV000087295 (output_matches_reference against the MassIVE file listing)
- Exactly 3 original–preprocessed file pairs are present, with preprocessed files identifiable by the "_MA-d3-c3-Min20_SR" suffix
- Every original file size exceeds its preprocessed counterpart's size in the table
- The output TSV contains both the per-file sizes table and one ratio value

## Limitations

- The ratio quantifies storage size only; downstream speed and memory benefits are separate metrics (the paper reports Lipid Annotator 34 min/6 GB on originals vs 7 min/3 GB on preprocessed files)
- The suffix and size behavior depend on the specific preprocessing parameters (20-count intensity threshold, demultiplexing with 75% pulses, smoothing of 3 points in LC and 7 in IM); other parameterizations yield different files and sizes
- Validation requires the MassIVE listing to contain both original and PreProcessor-generated files for the same LC-DT IM-MS triplicate runs

## Evidence
