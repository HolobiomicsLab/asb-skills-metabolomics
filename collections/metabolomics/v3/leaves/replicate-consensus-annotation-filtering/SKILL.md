---
name: replicate-consensus-annotation-filtering
description: Use when comparing annotation counts between two LC-IM-MS replicate groups
  (e.g. original vs preprocessed MS-files) and you need a per-replicate consensus
  cutoff instead of per-file totals.
when_to_use_negative:
- When the downstream tool already outputs a consensus feature table across replicates
  and per-file lists must not be re-filtered
- When the goal is a per-file (individual) annotation gain — that figure (41% ± 7%)
  is computed without the replicate-consensus cutoff and mixing the two bases invalidates
  the comparison
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Lipid Annotator
  role: Produces per-file lipid annotation lists from Agilent .d MS-files; its replicate-consensus
    rule (feature present in at least half of analyzed MS-files) defines the counting
    basis used for the consensus filtering.
- name: PNNL PreProcessor
  role: Generates the preprocessed MS-file set whose consensus annotation counts are
    compared against the original set to compute the 19.4% annotation increase.
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:eb4c78d43c112394031625ee832266b6506b6f620560a15c6855919efa005578
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Lipid Annotator
  - PNNL PreProcessor
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# replicate-consensus-annotation-filtering

## Summary

Counts lipid annotations in consensus across replicates, using the Lipid Annotator rule that an MS1-level feature must be present in at least half the analyzed MS-files, and quantifies the annotation gain attributable to a preprocessing step.

## When to use

Use when comparing annotation counts between two LC-IM-MS replicate groups (e.g. original vs preprocessed MS-files) and you need a per-replicate consensus cutoff instead of per-file totals.

## When NOT to use

- When the downstream tool already outputs a consensus feature table across replicates and per-file lists must not be re-filtered
- When the goal is a per-file (individual) annotation gain — that figure (41% ± 7%) is computed without the replicate-consensus cutoff and mixing the two bases invalidates the comparison

## Inputs

- Lists of detected features per MS-file (e.g. Agilent .d files processed by Lipid Annotator)
- Lipid annotation lists per replicate from Lipid Annotator
- Group labels distinguishing original vs preprocessed replicate sets

## Outputs

- Consensus-filtered annotation counts per group (features present in ≥ half of replicates, e.g. ≥2 of 3)
- Percent increase in total lipid annotations between groups
- Comparison table of annotation counts (e.g. lipid_annotation_comparison.csv)

## How to apply

Load the deposited lists of detected features and lipid annotations for each replicate group, then apply the Lipid Annotator consensus rule: an MS1-level feature must be present in at least half of the analyzed MS-files, so for a triplicate run count annotations supported in at least 2 of 3 replicates per group. Do this separately for the original and the preprocessed file sets (e.g. PNNL PreProcessor output) so the two groups are counted on the same basis. Compute the percent increase in total annotations between groups, which is the metric reported when PreProcessor preprocessing boosted annotations by 19.4% for features found in at least 2 replicates (comparable to an average 41% ± 7% increase when each file is analyzed individually). Decision point: if you instead sum per-file annotations without the replicate cutoff, you will report the 41%-per-file figure rather than the 19.4% consensus figure. Validation is the recomputed percentage increase falling in the 19–20% range and the counts matching the deposited annotation lists.

## Related tools

- **Lipid Annotator** (Produces per-file lipid annotation lists from Agilent .d MS-files; its replicate-consensus rule (feature present in at least half of analyzed MS-files) defines the counting basis used for the consensus filtering.)
- **PNNL PreProcessor** (Generates the preprocessed MS-file set whose consensus annotation counts are compared against the original set to compute the 19.4% annotation increase.) — https://omics.pnl.gov/software/pnnl-preprocessor

## Examples

```
python consensus_counts.py --group original MSV000087295/original/*.tsv --group preprocessed MSV000087295/preprocessed/*.tsv --cutoff 2of3 > lipid_annotation_comparison.csv
```

## Evaluation signals

- Recomputed percent increase in consensus annotations falls within the published 19–20% range (19.4% reported).
- Per-group annotation counts match the deposited lists in MassIVE accession MSV000087295.
- The replicate cutoff (≥ half of analyzed MS-files, e.g. ≥2 of 3) is applied consistently to both original and preprocessed groups.
- Switching to per-file counting reproduces the alternative published figure of 41% ± 7% increase, confirming the two counting bases are kept distinct.

## Limitations

- The 19.4% gain applies to features found in at least 2 replicates; the per-file figure (41% ± 7%) is a different counting basis and is not interchangeable.
- The consensus rule ('at least half') requires a known replicate count; results shift if the number of replicates changes.
- Counts depend on the deposited annotation lists in MassIVE accession MSV000087295; if those lists change, the 19–20% validation range may not hold.

## Evidence
