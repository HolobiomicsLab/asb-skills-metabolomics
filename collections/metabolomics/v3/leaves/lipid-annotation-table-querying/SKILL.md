---
name: lipid-annotation-table-querying
description: Use when you have annotation lists from Lipid Annotator for original
  vs. preprocessed LC-IM-MS replicates and need per-replicate detection/annotation
  counts for a specific lipid feature.
when_to_use_negative:
- You already have a single consolidated feature table and only need downstream statistics
  (e.g. fold change, p-values), not per-replicate presence/absence tallies.
- The data are not LC-IM-MS lipidomics replicates and the tool used is not Lipid Annotator
  or a compatible annotation table producer.
edam_operation: http://edamontology.org/operation_3636
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PNNL PreProcessor
  role: Produces the preprocessed MS-files whose annotation tables are queried; smoothing
    is what makes low-abundance features like Sphingomyelin d34:5 detectable in all
    replicates.
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
- name: Lipid Annotator
  role: Generates the lipid feature finding and annotation lists that are the target
    of the per-replicate querying and tallying.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:700f993704d84344a4d98cde99cd748ce052bcd2813d95297389c6c3d78fd552
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Lipid Annotator
  - PNNL PreProcessor
  edam_operation: http://edamontology.org/operation_3636
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# lipid-annotation-table-querying

## Summary

Query and tally lipid annotation records across replicate LC-IM-MS feature lists, e.g. to determine in which of the 3 replicates a low-abundance feature such as Sphingomyelin d34:5 (m/z 695.5088, RT 6.2 min, AT 36.7 ms) is annotated after PNNL PreProcessor smoothing versus in the original data.

## When to use

Use when you have annotation lists from Lipid Annotator for original vs. preprocessed LC-IM-MS replicates and need per-replicate detection/annotation counts for a specific lipid feature.

## When NOT to use

- You already have a single consolidated feature table and only need downstream statistics (e.g. fold change, p-values), not per-replicate presence/absence tallies.
- The data are not LC-IM-MS lipidomics replicates and the tool used is not Lipid Annotator or a compatible annotation table producer.

## Inputs

- Original MS-files (.d) from a triplicate human plasma lipid extract analyzed by LC-DT IM-MS
- PreProcessor-generated MS-files (.d) after LC compression, IM interpolation, demultiplexing (75% pulses), smoothing (points: 3 in LC and 7 in IM), intensity threshold of 50 counts, and 1 adjacent point spike removal
- Feature/annotation lists produced by Lipid Annotator for original and preprocessed files

## Outputs

- Summary table with columns: lipid annotation, feature m/z, RT, AT, data type (original/preprocessed), and per-replicate detection flags
- Per-replicate annotation presence/absence counts for the queried lipid feature

## How to apply

Download the original and preprocessed lists of detected features and lipid annotations for the triplicate human plasma lipid extract from the MassIVE deposit MSV000087295 (ftp://massive.ucsd.edu/MSV000087295). Load the annotation tables and query all records for the target lipid annotation (e.g. Sphingomyelin d34:5), cross-checking against the reported feature coordinates such as m/z 695.5088, RT 6.2 min, AT 36.7 ms from the smoothed data. For each data type (original vs PreProcessor-preprocessed), tally in which of the 3 LC-IM-MS replicate annotation lists the record appears. The rationale is that multidimensional smoothing in the PreProcessor improves peak shapes of poorly defined low-abundance signals, so only features present in the preprocessed tables would be expected to appear consistently across replicates.

## Related tools

- **PNNL PreProcessor** (Produces the preprocessed MS-files whose annotation tables are queried; smoothing is what makes low-abundance features like Sphingomyelin d34:5 detectable in all replicates.) — https://omics.pnl.gov/software/pnnl-preprocessor
- **Lipid Annotator** (Generates the lipid feature finding and annotation lists that are the target of the per-replicate querying and tallying.)

## Examples

```
python scripts/lipid_annotation_query.py --annotations original/*.tsv preprocessed/*.tsv --lipid 'Sphingomyelin d34:5' --mz 695.5088 --rt 6.2 --at 36.7
```

## Evaluation signals

- The queried record (e.g. Sphingomyelin d34:5) appears in all 3 preprocessed replicate annotation lists.
- The same record is absent from all 3 original replicate annotation lists, reproducing the paper's reported result.
- The summary table includes the feature m/z, RT, AT, data type, and a per-replicate flag for each of the 3 replicates.
- Cross-check that the queried record's m/z/RT/AT match the coordinates reported from the smoothed data (m/z 695.5088, RT 6.2 min, AT 36.7 ms).

## Limitations

- The evaluation depends on the specific data deposit (MSV000087295) and the PreProcessor parameters used in the article; results will differ if different smoothing or threshold parameters are applied.
- Feature coordinates are reported from the smoothed data, so original-file tables may not contain the queried record at all, making 'absence' the expected outcome rather than a missing-data error.
- The article does not state tool versions for the PreProcessor or Lipid Annotator used.

## Evidence
