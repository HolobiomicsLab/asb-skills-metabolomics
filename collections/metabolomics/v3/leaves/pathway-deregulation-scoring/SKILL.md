---
name: pathway-deregulation-scoring
description: Use when a raw metabolite measurement table (e.g. metabolite-by-sample
  CSV with a phenotype Label column) must be converted into individual patient-level
  pathway features for classification modeling.
when_to_use_negative:
- Input is already a pathway-level feature table (PDS matrix) — rerunning the transformation
  is redundant; proceed to lilikoi.select_pathways instead.
- The goal is group-level pathway enrichment rather than personalized (per-sample)
  pathway scores — tools like MetPA, IMPaLA, or MPEA operate at group level and do
  not produce individual patient-level pathway matrices.
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: lilikoi
  role: provides lilikoi.get_pd_scores and lilikoi.metab_to_pathway to map metabolites
    to pathways and compute the PDS matrix via Pathifier principal curves
  repo: https://github.com/lanagarmire/lilikoi
- name: R
  role: scripting environment in which the lilikoi dimension transformation module
    is executed
- name: SMPDB
  role: pathway-metabolite reference database (lilikoi::data.smpdb) used to define
    pathway membership for PDS computation
provenance:
  source_task_ids:
  - task_007
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giy136
    title: 'Lilikoi: an R package for personalized pathway-based classification modeling
      using metabolomics data'
  source_content_hash: sha256:5459ed7c2b26d2d91906d1b0bf7c77ae24282dfe452a7b64bb9d384b27412164
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi
  tools:
  - R
  - R/Bioconductor
  - SMPDB
  - lilikoi
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# pathway-deregulation-scoring

## Summary

Computes a personalized pathway deregulation score (PDS) matrix that transforms a metabolite-by-sample profile into a pathway-by-sample profile using the Pathifier principal-curve algorithm, as implemented in the Lilikoi R package's dimension transformation module.

## When to use

Use when a raw metabolite measurement table (e.g. metabolite-by-sample CSV with a phenotype Label column) must be converted into individual patient-level pathway features for classification modeling.

## When NOT to use

- Input is already a pathway-level feature table (PDS matrix) — rerunning the transformation is redundant; proceed to lilikoi.select_pathways instead.
- The goal is group-level pathway enrichment rather than personalized (per-sample) pathway scores — tools like MetPA, IMPaLA, or MPEA operate at group level and do not produce individual patient-level pathway matrices.

## Inputs

- metabolite measurement table (e.g. plasma_breast_cancer.csv from lilikoi extdata: metabolites as columns, samples as rows, plus a Label column)
- metabolite-to-pathway mapping table from lilikoi.metab_to_pathway
- SMPDB pathway-metabolite database (lilikoi::data.smpdb, or a subset such as [1:25,] for fast tests)

## Outputs

- pathway-by-sample PDS score matrix (rows = pathways, columns = patients/samples, values bounded 0–1), e.g. pds_matrix.csv with the reported 224 pathways × 207 patients structure for the plasma dataset

## How to apply

First map metabolite names to pathways with lilikoi.metab_to_pathway(metaboliteNames, "name"), which standardizes user-supplied metabolite names against HMDB/SMPDB-backed databases. Then call lilikoi.get_pd_scores(metaboliteMeasurements, metabolitePathwayTable, lilikoi::data.smpdb[1:25,]) to project each sample onto the principal curve defined per pathway; the PDS is the distance from the start of the principal curve to the projected point, quantifying deviation from the control level. Scores range from 0 to 1. For the plasma breast cancer dataset this yields a 224 pathways (rows) by 207 patients (columns) matrix. The [1:25,] SMPDB subset only accelerates test runs — swap in the full database for production. The resulting PDS matrix is the input for lilikoi.select_pathways feature selection and downstream machine learning.

## Related tools

- **lilikoi** (provides lilikoi.get_pd_scores and lilikoi.metab_to_pathway to map metabolites to pathways and compute the PDS matrix via Pathifier principal curves) — https://github.com/lanagarmire/lilikoi
- **R** (scripting environment in which the lilikoi dimension transformation module is executed)
- **SMPDB** (pathway-metabolite reference database (lilikoi::data.smpdb) used to define pathway membership for PDS computation)

## Examples

```
metabolitePathwayTable <- lilikoi.metab_to_pathway(metaboliteNames, "name"); PDSmatrix <- lilikoi.get_pd_scores(metaboliteMeasurements, metabolitePathwayTable, lilikoi::data.smpdb[1:25,])
```

## Evaluation signals

- The output file (e.g. pds_matrix.csv) exists and is a pathway (rows) by patient (columns) score matrix.
- All PDS values are bounded between 0 and 1.
- For the full plasma dataset, the matrix has the expected 224 pathways × 207 patients structure.
- Every metabolite name fed to lilikoi.metab_to_pathway was standardized (note ~20% may fail to map due to Levenshtein-distance name matching).
- Downstream lilikoi.select_pathways on the PDS matrix runs without error and retains pathways exceeding the 0.42 information-gain threshold.

## Limitations

- The lilikoi::data.smpdb[1:25,] subset is only for fast test runs; production runs should use the full SMPDB database.
- About 20% of metabolites cannot be mapped to standard names in the databases, partly because Levenshtein distance is used as the metabolite name similarity measure.
- Lilikoi depends on external HMDB, SMPDB, and MetaboAnalyst data; updating via lilikoi.update_database() handles >5GB datasets and may take over 20 minutes.

## Evidence

- [results] In the Pathifier-based transformation, each sample is projected onto the principal curve for a pathway, and its pathway deregulation score is the distance from the start of the principal curve: [evidence span withheld — anchor status: not_found]
- [results] PDS matrix structure for plasma dataset: "the metabolites are mapped to 224 pathways, and the resulting PDS score (ranging from 0 to 1) based matrix with 224 pathways (rows) and 207 patients (columns)"
- [readme] get_pd_scores invocation: "PDSmatrix <- lilikoi.get_pd_scores(metaboliteMeasurements, metabolitePathwayTable,
  lilikoi::data.smpdb[1:25,])"
- [abstract] dimension transformation module description: "dimension transformation module, which transforms the metabolomic profiles to personalized pathway-based profiles using pathway deregulation scores"
- [intro] gap relative to group-level tools: "none of these pathway-based methods transform the metabolite-sample matrix into pathway-sample matrix in order to entail pathway representation at the individual sample level (or personalized level)"
