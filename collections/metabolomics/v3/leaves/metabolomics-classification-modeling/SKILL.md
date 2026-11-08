---
name: metabolomics-classification-modeling
description: Use when a metabolomics sample-by-metabolite matrix with phenotype labels
  needs pathway-level feature selection and classification modeling at the individual
  sample level.
when_to_use_negative:
- The input is already a pathway-level feature matrix or the goal is group-level pathway
  enrichment only (MetPA, IMPaLA, MPEA operate at group level and do not generate
  individual patient-level pathway matrices, but if group-level results suffice Lilikoi's
  PDS transformation is unnecessary).
- The dataset is transcriptomics/proteomics rather than metabolomics — Lilikoi specializes
  in metabolomics data with metabolite-to-pathway mapping via standard IDs (PubChem
  CIDs, HMDB IDs, InChiKey, METLIN IDs).
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0203
tools:
- name: lilikoi
  role: 'Core R package providing the four modules: feature mapping (metab_to_pathway),
    dimension transformation (get_pd_scores), feature selection (select_pathways,
    method=''gain'', threshold=0.42), and classification (machine_learning, adjust_model)'
  repo: https://github.com/lanagarmire/lilikoi
- name: R
  role: Runtime environment in which the Lilikoi package pipeline is executed
  repo: https://cran.r-project.org/web/packages/lilikoi/index.html
- name: R/Bioconductor
  role: Ecosystem context; Lilikoi addresses the scarcity of comprehensive metabolomics
    analysis packages in R/Bioconductor
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giy136
    title: 'Lilikoi: an R package for personalized pathway-based classification modeling
      using metabolomics data'
  source_content_hash: sha256:4d0936cffebd649c86bf99a081092dace5ca8c56d1098ea2ef37f08c1e0b519a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi
  tools:
  - R
  - R/Bioconductor
  - lilikoi
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# metabolomics-classification-modeling

## Summary

Builds personalized pathway-based classification models from metabolomics data using the Lilikoi R package, which maps metabolites to pathways, transforms profiles into pathway deregulation scores (PDS), selects informative pathway features, and trains classifiers on phenotype labels (e.g. ER+ vs ER− breast cancer tissue).

## When to use

Use when a metabolomics sample-by-metabolite matrix with phenotype labels needs pathway-level feature selection and classification modeling at the individual sample level.

## When NOT to use

- The input is already a pathway-level feature matrix or the goal is group-level pathway enrichment only (MetPA, IMPaLA, MPEA operate at group level and do not generate individual patient-level pathway matrices, but if group-level results suffice Lilikoi's PDS transformation is unnecessary).
- The dataset is transcriptomics/proteomics rather than metabolomics — Lilikoi specializes in metabolomics data with metabolite-to-pathway mapping via standard IDs (PubChem CIDs, HMDB IDs, InChiKey, METLIN IDs).

## Inputs

- sample-by-metabolite measurement matrix (e.g. CSV such as plasma_breast_cancer.csv with a Label column)
- phenotype labels (e.g. ER+ / ER− or cancer vs normal)
- optional clinical covariate table (e.g. plasma_breast_cancer_Meta.csv with Age, Race)
- SMPDB/HMDB pathway-metabolite reference database (lilikoi::data.smpdb)

## Outputs

- metabolite-to-pathway mapping table (metabolitePathwayTable)
- PDS score matrix (pathways × patients, scores 0–1)
- selected significant pathways with information-gain relevance scores
- machine-learning classification results on pathway features (mlResults)
- final adjusted model incorporating clinical factors (finalModel)
- clustering figure of pathway-level profiles

## How to apply

Load the metabolite measurement matrix (e.g. 162 metabolites × 271 ER+/ER− tissue samples) and standardize metabolite names, mapping them to pathways with lilikoi.metab_to_pathway using searchType 'name' (in the plasma dataset, 182 of 227 metabolites mapped to HMDB IDs; 120 of 162 in the tissue dataset). Transform the metabolite profile into a personalized pathway-based profile by computing the pathway deregulation score (PDS) matrix (scores 0–1) with lilikoi.get_pd_scores against the SMPDB pathway-metabolite database. Select significant pathway features related to the phenotype with lilikoi.select_pathways (threshold = 0.42, method = 'gain'); for the ER+ vs ER− tissue dataset this should yield exactly six pathways, with pantothenate and CoA biosynthesis ranked highest at information gain 0.19 and histidine metabolism second at 0.16. Build pathway-level classification models with lilikoi.machine_learning on the selected pathways using the phenotype labels, and optionally refine with lilikoi.adjust_model incorporating clinical factors such as Age and Race; optionally cluster the selected pathway PDS profiles (e.g. hierarchical clustering) to verify phenotype separation.

## Related tools

- **lilikoi** (Core R package providing the four modules: feature mapping (metab_to_pathway), dimension transformation (get_pd_scores), feature selection (select_pathways, method='gain', threshold=0.42), and classification (machine_learning, adjust_model)) — https://github.com/lanagarmire/lilikoi
- **R** (Runtime environment in which the Lilikoi package pipeline is executed) — https://cran.r-project.org/web/packages/lilikoi/index.html
- **R/Bioconductor** (Ecosystem context; Lilikoi addresses the scarcity of comprehensive metabolomics analysis packages in R/Bioconductor)

## Examples

```
PDSmatrix <- lilikoi.get_pd_scores(metaboliteMeasurements, metabolitePathwayTable, lilikoi::data.smpdb[1:25,]); significantPathways <- lilikoi.select_pathways(PDSmatrix, metaboliteMeasurements, threshold = 0.42, method = "gain")
```

## Evaluation signals

- The metabolite-to-pathway mapping rate is plausible (plasma dataset: 182 of 227 metabolites mapped to HMDB IDs; tissue dataset: 120 of 162).
- The PDS matrix has the expected shape and range (e.g. 224 pathways × 207 patients for dataset 1; scores ranging 0 to 1).
- Feature selection at threshold 0.42 with method 'gain' selects exactly six pathways for the ER+ vs ER− tissue dataset, with pantothenate and CoA biosynthesis highest (information gain 0.19) and histidine metabolism second (0.16).
- Hierarchical clustering of the six selected pathway PDS scores visually separates ER+ from ER− samples.
- The classification model (mlResults) trains successfully on the selected pathway features using the phenotype Label column.

## Limitations

- A small percentage (20%) of metabolites still cannot be mapped to standard names in databases.
- Levenshtein distance similarity measure is a possible cause of metabolite name mismatching.
- No AutoML for automatic classification algorithm selection is available (planned for future versions).
- Updating external databases (HMDB, SMPDB, MetaboAnalyst) via lilikoi.update_database is slow: datasets are large (>5GB) and may take greater than 20 minutes.

## Evidence

- [methods] significantPathways <- lilikoi.select_pathways(PDSmatrix, metaboliteMeasurements, threshold = 0.42, method = "gain"): "significantPathways <- lilikoi.select_pathways(PDSmatrix, metaboliteMeasurements, threshold = 0.42, method = "gain")"
- [abstract] Lilikoi is a new and comprehensive R package for personalized pathway-based classification modeling using metabolomics data: "Lilikoi (the Hawaiian word for passion fruit) is a new and comprehensive R package for personalized pathway-based classification modeling using metabolomics data"
- [abstract] Four basic modules are presented as the backbone of the package: feature mapping module ... dimension transformation module ... feature selection module ... classification and prediction module: [evidence span withheld — anchor status: not_found]
- [readme] mlResults <- lilikoi.machine_learning(PDSmatrix, metaboliteMeasurements$Label, significantPathways): "mlResults <- lilikoi.machine_learning(PDSmatrix, metaboliteMeasurements$Label,
  significantPathways)"
- [readme] Lilikoi depends on data from HMDB, SMPDB, and MetaboAnalyst.: "Lilikoi depends on data from HMDB, SMPDB, and MetaboAnalyst."
- [discussion] a small percentage (20%) of the metabolites still cannot be mapped to the standard names in databases: "a small percentage (20%) of the metabolites still cannot be mapped to the standard names in databases"
