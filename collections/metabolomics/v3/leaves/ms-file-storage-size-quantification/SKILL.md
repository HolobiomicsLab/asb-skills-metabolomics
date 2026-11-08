---
name: ms-file-storage-size-quantification
description: Use when both original raw MS-files and preprocessed counterparts of
  the same dataset exist in a deposit (e.g. MassIVE MSV000087295) and the goal is
  to numerically verify a claim that preprocessing reduces file size.
when_to_use_negative:
- Input is a single MS-file without a matched preprocessed counterpart, so no paired
  size comparison can be made
- The analysis goal is signal-quality improvement (e.g. lipid annotation counts, mass
  error), not storage-size quantification
- File listings cannot be matched one-to-one between original and preprocessed versions,
  making the paired ratio undefined
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: PNNL PreProcessor
  role: Generates the preprocessed MS-files whose reduced byte sizes are being quantified
    against the originals
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
- name: R
  role: Used to assemble the sizes table, sum original vs. preprocessed byte sizes,
    compute the ratio, and write the TSV output
- name: UIMF-Viewer
  role: PNNL-Comp-Mass-Spec repository associated with the UIMF intermediate format
    used by the PreProcessor workflow
  repo: PNNL-Comp-Mass-Spec/UIMF-Viewer
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:5def697df31452cc1420f53ab8e96b0a20951e521ce9ce830810684cac71afa0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PNNL PreProcessor
  - R
  - UIMF-Viewer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# ms-file-storage-size-quantification

## Summary

Quantifies the storage-size benefit of MS-file preprocessing by comparing byte sizes of original raw LC-IM-MS files against PreProcessor-generated files (e.g. the "_MA-d3-c3-Min20_SR" lipid extract files in MassIVE MSV000087295) and expressing the result as a size-reduction ratio that must lie strictly between 0 and 1.

## When to use

Use when both original raw MS-files and preprocessed counterparts of the same dataset exist in a deposit (e.g. MassIVE MSV000087295) and the goal is to numerically verify a claim that preprocessing reduces file size.

## When NOT to use

- Input is a single MS-file without a matched preprocessed counterpart, so no paired size comparison can be made
- The analysis goal is signal-quality improvement (e.g. lipid annotation counts, mass error), not storage-size quantification
- File listings cannot be matched one-to-one between original and preprocessed versions, making the paired ratio undefined

## Inputs

- MassIVE deposit MSV000087295 file listing (3 original raw lipid extract MS-files)
- 3 corresponding PreProcessor-generated MS-files with suffix "_MA-d3-c3-Min20_SR"
- Per-file storage size in bytes for all 6 files

## Outputs

- Sizes table (original file, preprocessed file, byte sizes) saved as TSV
- Single numeric size-reduction ratio = preprocessed total size / original total size, with 0 < ratio < 1

## How to apply

Access the MassIVE deposit MSV000087295 and enumerate the file listings for the 3 original raw lipid extract MS-files and the 3 corresponding PreProcessor-generated MS-files carrying the suffix "_MA-d3-c3-Min20_SR" (minimum intensity threshold 20 counts). Extract each file's storage size in bytes and assemble a sizes table with columns for the original file, the preprocessed file, and their byte sizes. Sum the original-file sizes and the preprocessed-file sizes, then compute the size-reduction ratio as (preprocessed total size)/(original total size). Validate that the ratio is a single numeric value strictly between 0 and 1, mirroring the paper's claim that preprocessing reduces file sizes for faster and memory-efficient downstream processing; as an external anchor, the paper reports the proteomics demultiplexing file shrinking from 20 GB to 7.68 GB. Save the sizes table and the ratio to a TSV output file for downstream reporting.

## Related tools

- **PNNL PreProcessor** (Generates the preprocessed MS-files whose reduced byte sizes are being quantified against the originals) — https://omics.pnl.gov/software/pnnl-preprocessor
- **R** (Used to assemble the sizes table, sum original vs. preprocessed byte sizes, compute the ratio, and write the TSV output)
- **UIMF-Viewer** (PNNL-Comp-Mass-Spec repository associated with the UIMF intermediate format used by the PreProcessor workflow) — PNNL-Comp-Mass-Spec/UIMF-Viewer

## Examples

```
Rscript -e 'd<-read.delim("sizes.tsv"); r<-sum(d$preprocessed_bytes)/sum(d$original_bytes); stopifnot(r>0 && r<1); write.table(d,"sizes_out.tsv",sep="\t",quote=FALSE,row.names=FALSE); cat(r,"\n")'
```

## Evaluation signals

- The output contains a single numeric size-reduction ratio satisfying 0 < ratio < 1
- The sizes table rows match the deposited file listings in MSV000087295 (output_matches_reference against the MassIVE file listing)
- The table contains exactly 3 original/preprocessed file pairs with the preprocessed files carrying the "_MA-d3-c3-Min20_SR" suffix
- Ratio is reproducible as (sum of preprocessed byte sizes)/(sum of original byte sizes) recomputed independently from the table
- Output TSV persists both the sizes table and the ratio for downstream reporting

## Limitations

- File-size reduction depends on PreProcessor settings (e.g. minimum intensity threshold of 20 counts, saturation repair threshold of 40% of the abundance limit, LC compression where 4 frames become 1, IM interpolation where every drift bin becomes 3); different settings yield different ratios
- The ratio is dataset- and format-specific: results from the human plasma lipid extract LC-DT IM-MS triplicates or the proteomics demultiplexing case do not automatically transfer to other instruments or workflows
- No dataset accessions or tool version are referenced in the article discussion, so provenance of parameters must be established from the deposit itself

## Evidence

- [other] deposit-level size finding: [evidence span withheld — anchor status: not_found]
