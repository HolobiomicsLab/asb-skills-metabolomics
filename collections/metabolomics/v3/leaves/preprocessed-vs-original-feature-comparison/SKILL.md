---
name: preprocessed-vs-original-feature-comparison
description: Use when evaluating whether raw-data preprocessing (multidimensional
  smoothing, spike removal) changes low-abundance feature detectability across LC-IM-MS
  replicates, using annotation lists from original vs PreProcessor-generated MS-files.
when_to_use_negative:
- Input is already a feature/annotation table derived only from preprocessed MS-files,
  with no paired original-data lists to compare against
- The analysis goal is quantification of high-abundance saturated peaks (use the saturation-repair
  evaluation with the 40% abundance-limit threshold instead)
- Data were not processed with (or are incompatible with) the PNNL PreProcessor's
  smoothing/demultiplexing parameterization, so an original-vs-preprocessed pairing
  does not exist
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: PNNL PreProcessor
  role: Generates the smoothed LC-IM-MS MS-files whose annotation lists are compared
    against the original files
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
- name: Lipid Annotator
  role: Performs feature finding and lipid annotation on both the original and the
    PreProcessor-generated MS-files to produce the lists being compared
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:4df5ce41b46e704940953ef94ee46708e6546010734e06678b455fd01b78e11f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Lipid Annotator
  - PNNL PreProcessor
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# preprocessed-vs-original-feature-comparison

## Summary

Reproduce the detection/annotation gain from PNNL PreProcessor preprocessing by comparing lipid annotation lists generated from original versus smoothed LC-IM-MS replicate files, benchmarking against paper-reported coordinates (e.g. Sphingomyelin d34:5, m/z 695.5088, RT 6.2 min, AT 36.7 ms).

## When to use

Use when evaluating whether raw-data preprocessing (multidimensional smoothing, spike removal) changes low-abundance feature detectability across LC-IM-MS replicates, using annotation lists from original vs PreProcessor-generated MS-files.

## When NOT to use

- Input is already a feature/annotation table derived only from preprocessed MS-files, with no paired original-data lists to compare against
- The analysis goal is quantification of high-abundance saturated peaks (use the saturation-repair evaluation with the 40% abundance-limit threshold instead)
- Data were not processed with (or are incompatible with) the PNNL PreProcessor's smoothing/demultiplexing parameterization, so an original-vs-preprocessed pairing does not exist

## Inputs

- Original LC-IM-MS replicate annotation lists from the triplicate human plasma lipid extract (MSV000087295)
- PreProcessor-preprocessed replicate annotation lists (smoothing points: 3 in LC and 7 in IM; intensity threshold 50 counts; 1 adjacent point spike removal)
- Reported feature coordinates for the target lipid (m/z 695.5088, RT 6.2 min, AT 36.7 ms)

## Outputs

- Summary table with columns: lipid annotation, feature m/z, RT, AT, data type (original/preprocessed), per-replicate detection flags
- Per-replicate tally of annotation presence for original vs preprocessed data
- Verdict on whether the target lipid is detected in all 3 replicates only after preprocessing

## How to apply

Download the original and preprocessed detected-feature and lipid-annotation lists for the triplicate human plasma lipid extract LC-DT IM-MS run from the MassIVE deposit MSV000087295 (ftp://massive.ucsd.edu/MSV000087295). Load the annotation tables, query records matching the target lipid annotation (e.g. Sphingomyelin d34:5), and cross-check reported feature coordinates (m/z 695.5088, RT 6.2 min, AT 36.7 ms) from the smoothed data. Tally, per data type, in how many of the 3 replicates' annotation lists the record appears. Emphasize low-abundance ions, since multidimensional smoothing improves peak shapes of poorly defined low-abundance signals that fail feature finding in original files. Compile a summary table with columns for lipid annotation, feature m/z, RT, AT, data type (original/preprocessed), and per-replicate detection flags.

## Related tools

- **PNNL PreProcessor** (Generates the smoothed LC-IM-MS MS-files whose annotation lists are compared against the original files) — https://omics.pnl.gov/software/pnnl-preprocessor
- **Lipid Annotator** (Performs feature finding and lipid annotation on both the original and the PreProcessor-generated MS-files to produce the lists being compared)

## Evaluation signals

- Sphingomyelin d34:5 (m/z 695.5088, RT 6.2 min, AT 36.7 ms) is present in all 3 preprocessed replicate annotation lists and absent from all 3 original replicate lists
- Smoothing increases the number of LC-IM-MS lipid features consistently detected across 3 replicates by ~70.7% relative to original data
- Total lipid annotations for features found in at least 2 replicates increase by ~19.4% in the preprocessed condition
- Summary table contains the required columns (lipid annotation, m/z, RT, AT, data type, per-replicate flags) with no missing replicate entries
- Downstream annotation throughput checks: Lipid Annotator should process PreProcessor-generated files faster (~7 min, 3 GB RAM) than originals (~34 min, 6 GB RAM)

## Limitations

- Preprocessing gains are demonstrated for LC-DT IM-MS TOF data converted to UIMF; applicability to other IM-MS instrumentation requires vendor conversion ('converting to UIMF as an intermediate step')
- No tool version, dataset accessions, or quantitative performance metrics are stated in the article's discussion section
- Detection gains target poorly defined low-abundance ions and detector-saturated high-abundance ions; intermediate-abundance features may show little or no difference between original and preprocessed lists

## Evidence
