---
name: replicate-consistency-annotation-checking
description: Use when triplicate LC-IM-MS replicate annotation lists must be cross-checked
  to verify that smoothing/preprocessing improves consistent detection of low-abundance
  annotated features.
when_to_use_negative:
- Data are not from a replicate set (n<3) — replicate-consistency tallies require
  at least 3 LC-IM-MS replicates of the same sample (e.g. the human plasma lipid extract
  analyzed in triplicate).
- No lipid annotation lists are available (e.g. only raw MS1 spectra without downstream
  Lipid Annotator output) — the skill operates on annotation tables, not raw spectra.
- The feature of interest is a high-abundance ion that is already consistently detected
  in all original replicates — the skill's value is showing where smoothing rescues
  low-abundance features.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PNNL PreProcessor
  role: Generates the preprocessed (smoothed) MS-files whose annotation lists are
    compared against the original ones to assess replicate consistency
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
- name: Lipid Annotator
  role: Produces the lipid annotation lists for features found in at least 2 replicates;
    its per-replicate outputs are the objects being cross-checked
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:65eeab1e5077750dcd9913b0b5beff7dc42be42dbedacc700b33beb26bc8adf2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Lipid Annotator
  - PNNL PreProcessor
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# replicate-consistency-annotation-checking

## Summary

A QC skill that compares per-replicate lipid annotation lists (original vs. preprocessed LC-IM-MS data) to determine whether low-abundance features — e.g. Sphingomyelin d34:5 — become consistently detectable and annotatable across all replicates only after PNNL PreProcessor smoothing.

## When to use

Use when triplicate LC-IM-MS replicate annotation lists must be cross-checked to verify that smoothing/preprocessing improves consistent detection of low-abundance annotated features.

## When NOT to use

- Data are not from a replicate set (n<3) — replicate-consistency tallies require at least 3 LC-IM-MS replicates of the same sample (e.g. the human plasma lipid extract analyzed in triplicate).
- No lipid annotation lists are available (e.g. only raw MS1 spectra without downstream Lipid Annotator output) — the skill operates on annotation tables, not raw spectra.
- The feature of interest is a high-abundance ion that is already consistently detected in all original replicates — the skill's value is showing where smoothing rescues low-abundance features.

## Inputs

- Original (unprocessed) lipid annotation lists for each of 3 LC-IM-MS replicates of a human plasma lipid extract (from MassIVE MSV000087295)
- PreProcessor-generated (smoothed) lipid annotation lists for the same 3 replicates
- Feature coordinates of the target annotation, e.g. m/z 695.5088, RT 6.2 min, AT 36.7 ms for Sphingomyelin d34:5

## Outputs

- Summary table with columns: lipid annotation, feature m/z, RT, AT, data type (original/preprocessed), and per-replicate detection flags
- Per-replicate presence/absence tallies for the target lipid in original vs. preprocessed data

## How to apply

Download the original and preprocessed lists of detected features and lipid annotations for the triplicate human plasma lipid extract from the MassIVE deposit MSV000087295 (ftp://massive.ucsd.edu/MSV000087295). Load the annotation tables and query all records for the target lipid annotation (e.g. Sphingomyelin d34:5), cross-checking against reported feature coordinates (m/z 695.5088, RT 6.2 min, AT 36.7 ms) from the smoothed data. For each data type — original vs. PreProcessor-preprocessed — tally in which of the 3 LC-IM-MS replicate annotation lists the record appears. Write a summary table with columns for lipid annotation, feature m/z, RT, AT, data type, and per-replicate detection flags. The PreProcessor smoothing increased the number of features consistently detected across 3 replicates by 70.7%, which is the rationale for expecting improved replicate consistency in preprocessed data.

## Related tools

- **PNNL PreProcessor** (Generates the preprocessed (smoothed) MS-files whose annotation lists are compared against the original ones to assess replicate consistency) — https://omics.pnl.gov/software/pnnl-preprocessor
- **Lipid Annotator** (Produces the lipid annotation lists for features found in at least 2 replicates; its per-replicate outputs are the objects being cross-checked)

## Evaluation signals

- The Sphingomyelin d34:5 record (m/z 695.5088, RT 6.2 min, AT 36.7 ms) is present in all 3 preprocessed replicate annotation lists and absent from all 3 original replicate lists, reproducing the paper's reported result.
- The summary table must contain a per-replicate detection flag column with one flag per replicate (3 flags total) for each data type.
- The overall count of lipid annotations found in at least 2 replicates should increase by ~19.4% when comparing preprocessed to original data, consistent with the paper's reported gain.
- The number of lipid features consistently detected across 3 replicates should be increased by ~70.7% in preprocessed data, matching the article's metric.
- Feature coordinates (m/z, RT, AT) used for cross-checking must match the values reported in the smoothed-data annotation list.

## Limitations

- The check depends on downstream Lipid Annotator processing: Lipid Annotator took 34 min and 6 GB RAM on the 3 original MS-files versus 7 min and 3 GB RAM on the PreProcessor-generated files, so runtime/memory constraints may limit scaling to larger datasets.
- The skill does not correct annotation errors themselves; it only verifies whether annotations appear consistently across replicates, so misannotations can still be consistently present.
- The 70.7% and 19.4% gains are specific to the LC-DT IM-MS triplicate human plasma lipid extract reported in the article and may not generalize to other sample types or instruments.

## Evidence
