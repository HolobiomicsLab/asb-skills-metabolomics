---
name: two-regime-threshold-comparison
description: Use when you need to quantify whether retention-time drift for monitored
  compounds in a pooled QC pool is dominated by batch-level offsets rather than by
  short-term (within-batch) scatter, e.g. to justify batch-wise RT recalibration.
when_to_use_negative:
- Input is already a post-correction feature table or intensity drift data rather
  than raw RT deviations from module_TrackCmp
- The goal is to monitor contaminant levels or m/z calibration offsets instead of
  retention-time drift regimes
- The QC pool has not been acquired across multiple batches, so the interbatch regime
  cannot be defined
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: QC4Metabolomics
  role: Provides module_TrackCmp, which detects peaks from a user-defined list of
    m/z and RT pairs and yields the RT deviation data for the long-term QC pool used
    in this comparison.
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: R
  role: Implementation language of the QC4Metabolomics package used to group RT deviations
    by batch and compute the two-regime comparison statistics.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:fa4da936a439be9eebed044c560a8d888fcabf10854c061f7d04f7bc383ac872
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - QC4Metabolomics
  - R
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# two-regime-threshold-comparison

## Summary

Reproduces the QC4Metabolomics observation that short-term (within-batch) RT deviations for a long-term QC pool are normally below 0.05 min while interbatch RT offsets can be at least twice as large, by grouping per-compound RT deviations by batch and comparing the two regimes.

## When to use

Use when you need to quantify whether retention-time drift for monitored compounds in a pooled QC pool is dominated by batch-level offsets rather than by short-term (within-batch) scatter, e.g. to justify batch-wise RT recalibration.

## When NOT to use

- Input is already a post-correction feature table or intensity drift data rather than raw RT deviations from module_TrackCmp
- The goal is to monitor contaminant levels or m/z calibration offsets instead of retention-time drift regimes
- The QC pool has not been acquired across multiple batches, so the interbatch regime cannot be defined

## Inputs

- RT deviation data for the long-term QC pool analyzed across all batches, produced by module_TrackCmp from a user-defined m/z/RT pair list
- Batch identifier / acquisition-batch labels for each QC injection

## Outputs

- Per-compound within-batch RT deviation statistics grouped by batch
- Between-batch RT offset statistics relative to reference RTs
- Two-regime summary comparison table (within-batch < 0.05 min vs interbatch ≥ 2× larger)

## How to apply

Load the RT deviation data for the long-term QC pool (e.g. MetNexs) as tracked by module_TrackCmp from a user-defined list of m/z and retention time pairs. Group each injection's RT deviation by its acquisition batch identifier so that within-batch and between-batch regimes can be separated. Compute the typical magnitude of RT deviation per monitored compound within each batch, then compute the offsets between batch-level RTs and the reference RTs for the same compounds. Verify the two-regime result: within-batch deviations should normally be less than 0.05 min, while differences between different batches can be at least twice as large. Save the per-compound RT deviations with batch labels and the regime comparison as a structured table.

## Related tools

- **QC4Metabolomics** (Provides module_TrackCmp, which detects peaks from a user-defined list of m/z and RT pairs and yields the RT deviation data for the long-term QC pool used in this comparison.) — https://github.com/stanstrup/QC4Metabolomics
- **R** (Implementation language of the QC4Metabolomics package used to group RT deviations by batch and compute the two-regime comparison statistics.)

## Evaluation signals

- Within-batch RT deviations for the QC pool are normally less than 0.05 min across the monitored compounds.
- Between-batch RT differences are at least twice the magnitude of the within-batch deviations.
- Every injection is assigned a batch identifier, and per-compound deviations carry both batch and compound labels in the output table.
- The deviation table is reproducible from the module_TrackCmp output for the same user-defined m/z/RT pair list.

## Limitations

- The 0.05 min threshold and the 'at least twice as large' interbatch ratio are empirical observations for the MetNexs long-term QC pool; they are not universal cutoffs for other instruments or pools.
- The instruments typically do not alert the user if calibration is inaccurate, so RT drift may go unnoticed without explicit tracking.
- Results depend on the quality of the user-defined m/z/RT pair list and on correct batch labeling; misassignment of batches would confound the two regimes.

## Evidence

- [results] RT deviations are normally less than 0.05 min in the shorter term, while differences between different batches can be at least twice as large: "in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large"
- [methods] module_TrackCmp detects peaks based on a user-defined list of m/z and RT pairs: "module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs"
- [results] RT and m/z deviation data for the long-term QC pool MetNexs analyzed within all batches: "RT and m/z deviation data for our long-term QC pool (MetNexs) analyzed within all batches"
