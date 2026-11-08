---
name: principal-curve-dimensionality-reduction
description: Use when a metabolite-by-sample measurement matrix (e.g. plasma_breast_cancer.csv)
  must be converted into a pathway-by-sample PDS matrix to enable personalized, patient-level
  pathway classification modeling.
when_to_use_negative:
- Input is already a pathway-level feature table (PDS matrix) — no further dimension
  transformation is needed before feature selection or classification
- The goal is group-level pathway enrichment rather than personalized (per-sample)
  pathway matrices — the paper notes MetPA, IMPaLA, and MPEA 'are not capable of generating
  such individual patient-level pathway matrices', which is exactly this skill's purpose
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: lilikoi
  role: R package providing metab_to_pathway and get_pd_scores to project samples
    onto per-pathway principal curves and derive the PDS matrix
  repo: https://github.com/lanagarmire/lilikoi
- name: R
  role: implementation environment for the lilikoi dimension transformation workflow
  repo: https://cran.r-project.org/web/packages/lilikoi/index.html
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giy136
    title: 'Lilikoi: an R package for personalized pathway-based classification modeling
      using metabolomics data'
  source_content_hash: sha256:95e5ecc8af6943568e361b921e365a04172a4439989768b093e8c9b6fd8d76ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi
  tools:
  - R
  - lilikoi
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# principal-curve-dimensionality-reduction

## Summary

Transforms a metabolite-sample profile into a personalized pathway-level profile by computing Pathifier-based pathway deregulation scores (PDS), where each sample is projected onto a principal curve per pathway and scored 0–1 by its distance from the curve start.

## When to use

Use when a metabolite-by-sample measurement matrix (e.g. plasma_breast_cancer.csv) must be converted into a pathway-by-sample PDS matrix to enable personalized, patient-level pathway classification modeling.

## When NOT to use

- Input is already a pathway-level feature table (PDS matrix) — no further dimension transformation is needed before feature selection or classification
- The goal is group-level pathway enrichment rather than personalized (per-sample) pathway matrices — the paper notes MetPA, IMPaLA, and MPEA 'are not capable of generating such individual patient-level pathway matrices', which is exactly this skill's purpose

## Inputs

- metabolite-by-sample measurement CSV (e.g. plasma_breast_cancer.csv, metabolite columns plus a Label column)
- metabolite names list (column names of the measurement matrix)
- metabolite-to-pathway mapping table (output of lilikoi.metab_to_pathway)
- pathway database (lilikoi::data.smpdb, full or subset e.g. [1:25,])

## Outputs

- pathway-by-sample PDS matrix (rows = pathways, columns = patients, values bounded 0–1; e.g. 224 pathways x 207 patients for the full plasma dataset)
- pds_matrix.csv file containing the PDS score matrix

## How to apply

Load the metabolite measurements (e.g. plasma_breast_cancer.csv from lilikoi's extdata) and extract metabolite names. Map metabolite names to pathways with lilikoi.metab_to_pathway(metaboliteNames, "name"), then compute the PDS matrix via lilikoi.get_pd_scores: each sample is projected onto the principal curve for each pathway and its deregulation score is the distance from the start of the principal curve to the projected point, quantifying deviation from the control level. For fast test runs pass a subset of the pathway database such as lilikoi::data.smpdb[1:25,]; swap in the full database for production, which for the plasma dataset yields 224 pathways by 207 patients. Write the resulting pathway-by-sample matrix (rows = pathways, columns = patients) to pds_matrix.csv and validate that all PDS values lie between 0 and 1.

## Related tools

- **lilikoi** (R package providing metab_to_pathway and get_pd_scores to project samples onto per-pathway principal curves and derive the PDS matrix) — https://github.com/lanagarmire/lilikoi
- **R** (implementation environment for the lilikoi dimension transformation workflow) — https://cran.r-project.org/web/packages/lilikoi/index.html

## Examples

```
PDSmatrix <- lilikoi.get_pd_scores(metaboliteMeasurements, metabolitePathwayTable, lilikoi::data.smpdb[1:25,])
```

## Evaluation signals

- Output file pds_matrix.csv exists and parses as a pathway (rows) by patient (columns) score matrix
- All PDS values are bounded between 0 and 1
- For the full plasma dataset, the PDS matrix has the reported structure of 224 pathways by 207 patients
- Pathway names in the matrix correspond to the pathways returned by lilikoi.metab_to_pathway for the input metabolite names
- Using the smpdb subset [1:25,] instead of the full database yields a smaller but consistently structured PDS matrix suitable for test runs

## Limitations

- lilikoi::data.smpdb[1:25,] is only a test-run subset; production runs require swapping in the full pathway database
- ~20% of metabolites cannot be mapped to standard names in databases, partly because Levenshtein distance is used as the similarity measure for metabolite name matching
- Lilikoi depends on external data from HMDB, SMPDB, and MetaboAnalyst; updating these via lilikoi.update_database() involves >5GB datasets and can take over 20 minutes

## Evidence

- [abstract] dimension transformation module, which transforms the metabolomic profiles to personalized pathway-based profiles using pathway deregulation scores: "dimension transformation module, which transforms the metabolomic profiles to personalized pathway-based profiles using pathway deregulation scores"
- [methods] each sample is projected onto the principal curve for a pathway, and its pathway deregulation score is the distance from the start of the principal curve: [evidence span withheld — anchor status: not_found]
- [results] the metabolites are mapped to 224 pathways, and the resulting PDS score (ranging from 0 to 1) based matrix with 224 pathways (rows) and 207 patients (columns): "the metabolites are mapped to 224 pathways, and the resulting PDS score (ranging from 0 to 1) based matrix with 224 pathways (rows) and 207 patients (columns)"
- [readme] PDSmatrix <- lilikoi.get_pd_scores(metaboliteMeasurements, metabolitePathwayTable, lilikoi::data.smpdb[1:25,]): "PDSmatrix <- lilikoi.get_pd_scores(metaboliteMeasurements, metabolitePathwayTable,
  lilikoi::data.smpdb[1:25,])"
- [readme] metabolitePathwayTable <- lilikoi.metab_to_pathway(metaboliteNames, "name"): "metabolitePathwayTable <- lilikoi.metab_to_pathway(metaboliteNames, "name")"
