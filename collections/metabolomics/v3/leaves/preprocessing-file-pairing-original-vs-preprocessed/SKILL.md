---
name: preprocessing-file-pairing-original-vs-preprocessed
description: Use when a MassIVE deposit pairs original raw LC-IM-MS files with PreProcessor-generated
  counterparts (suffix _MA-d3-c3-Min20_SR) and you must quantify preprocessing storage
  savings as a preprocessed/original byte-size ratio.
when_to_use_negative:
- The deposit holds only original raw files with no PreProcessor-generated counterparts,
  so no 1:1 size pairing or ratio can be computed.
- The goal is signal-quality evaluation (lipid annotation counts, mass error, feature
  detection speed) rather than quantifying storage-size reduction.
- Comparing byte sizes of files that are not a preprocessed/original pair, e.g. unrelated
  acquisition replicates or runs with different parameter suffixes.
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0153
tools:
- name: PNNL PreProcessor
  role: Generated the preprocessed MS-file counterparts (suffix _MA-d3-c3-Min20_SR)
    whose storage sizes are paired and compared against the original raw LC-IM-MS
    files
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
- name: R
  role: Assembles the 6-file sizes table, computes the preprocessed/original size-reduction
    ratio, and writes the TSV output for downstream reporting
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:5121fcd4015cfe96f7ccba139c92b848e8840c410d98c4ded726836a76fab58d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PNNL PreProcessor
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# preprocessing-file-pairing-original-vs-preprocessed

## Summary

Pairs each original raw LC-IM-MS file in a deposit (e.g. MassIVE MSV000087295, human plasma lipid extract triplicates) with its PNNL PreProcessor-generated counterpart and computes a byte-size reduction ratio quantifying the paper's claim of smaller preprocessed MS-files for faster downstream processing.

## When to use

Use when a MassIVE deposit pairs original raw LC-IM-MS files with PreProcessor-generated counterparts (suffix _MA-d3-c3-Min20_SR) and you must quantify preprocessing storage savings as a preprocessed/original byte-size ratio.

## When NOT to use

- The deposit holds only original raw files with no PreProcessor-generated counterparts, so no 1:1 size pairing or ratio can be computed.
- The goal is signal-quality evaluation (lipid annotation counts, mass error, feature detection speed) rather than quantifying storage-size reduction.
- Comparing byte sizes of files that are not a preprocessed/original pair, e.g. unrelated acquisition replicates or runs with different parameter suffixes.

## Inputs

- MassIVE deposit MSV000087295 file listing: 3 original raw human plasma lipid extract LC-DT IM-MS files and 3 PreProcessor-generated files with suffix _MA-d3-c3-Min20_SR
- Storage size in bytes of each of the 6 MS-files (original and preprocessed counterparts)

## Outputs

- TSV sizes table with columns: original file, preprocessed file, byte size of each pair member
- Single numeric size-reduction ratio = preprocessed total bytes / original total bytes, constrained to 0 < ratio < 1

## How to apply

Access the deposit (e.g. MassIVE MSV000087295) and enumerate the file listings, pairing each of the 3 original raw human plasma lipid extract LC-DT IM-MS replicate files with its PreProcessor-generated counterpart identified by the parameter suffix '_MA-d3-c3-Min20_SR' (minimum intensity threshold of 20 counts plus saturation repair). Extract the storage size in bytes of all 6 files into a table with columns for original file, preprocessed file, and their byte sizes. Compute the size-reduction ratio as (preprocessed total bytes)/(original total bytes); compression during preprocessing (4 LC frames become 1, smoothing at 3 points in LC and 7 in IM, Min20 intensity threshold, saturation repair with a threshold of 40% of the abundance limit) is why preprocessed files shrink, as in the paper's demultiplexing example shrinking a 20 GB file to 7.68 GB. Verify the ratio is a single numeric value strictly between 0 and 1 and that the sizes table matches the deposited file listing in MSV000087295. Save the sizes table and the ratio to a TSV using R for downstream reporting.

## Related tools

- **PNNL PreProcessor** (Generated the preprocessed MS-file counterparts (suffix _MA-d3-c3-Min20_SR) whose storage sizes are paired and compared against the original raw LC-IM-MS files) — https://omics.pnl.gov/software/pnnl-preprocessor
- **R** (Assembles the 6-file sizes table, computes the preprocessed/original size-reduction ratio, and writes the TSV output for downstream reporting)

## Examples

```
Rscript -e 'd<-read.delim("sizes.tsv");ratio<-sum(d$preprocessed_bytes)/sum(d$original_bytes);writeLines(as.character(ratio),"ratio.txt")'
```

## Evaluation signals

- Sizes table matches the MSV000087295 deposited file listing: exactly 3 original files paired 1:1 with 3 preprocessed files carrying the suffix _MA-d3-c3-Min20_SR.
- Size-reduction ratio is a single numeric value strictly between 0 and 1 (output_matches_reference check).
- Ratio is reproducible from the TSV: independently recomputed sum(preprocessed bytes)/sum(original bytes) equals the reported value.
- Order-of-magnitude sanity check against the paper's demultiplexing example where a 20 GB file shrank to 7.68 GB (ratio ~0.384).
- All sizes are positive byte counts and each original is paired with a same-replicate preprocessed counterpart from the triplicate LC-DT IM-MS analysis.

## Limitations

- The ratio quantifies storage size only; it does not measure signal-quality gains (e.g. 19.4% more lipid annotations, 70.7% more consistently detected lipid features), which require separate downstream analyses.
- Size reduction depends on the specific preprocessing parameters encoded in the suffix (Min20 intensity threshold, saturation repair at 40% of the abundance limit); other parameter sets may yield different ratios.
- The article's discussion cites no dataset accessions and no PreProcessor version, so reproducibility relies on the deposit's file listing and the paper's parameter reporting.
- Direct application is tied to formats the PreProcessor handles; other IM-MS instrumentation requires vendor support or converting to UIMF as an intermediate step.

## Evidence
