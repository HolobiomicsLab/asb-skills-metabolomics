---
name: pathway-level-clustering
description: Use when you have a PDS score matrix (0–1 pathway deregulation scores)
  restricted to Lilikoi-selected pathway features and want to check whether pathway-level
  profiles visually separate phenotype classes before or after classification modeling.
when_to_use_negative:
- Input is raw metabolite measurements rather than a pathway-level PDS matrix — run
  metab_to_pathway and get_pd_scores first
- No phenotype labels are available, so cluster separation relative to classes cannot
  be validated
- The goal is group-level pathway enrichment (e.g. MetPA/IMPaLA/MPEA) rather than
  personalized, sample-level pathway representation and classification
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0203
tools:
- name: lilikoi
  role: R package providing metab_to_pathway, get_pd_scores, select_pathways, and
    machine_learning used to build the PDS matrix and selected pathway features that
    are clustered
  repo: https://github.com/lanagarmire/lilikoi
- name: R
  role: Implementation language for the Lilikoi pipeline and hierarchical clustering
    of pathway-level profiles
  repo: https://cran.r-project.org/web/packages/lilikoi/index.html
- name: R/Bioconductor
  role: Ecosystem in which Lilikoi's metabolomics pathway analysis is executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giy136
    title: 'Lilikoi: an R package for personalized pathway-based classification modeling
      using metabolomics data'
  source_content_hash: sha256:1fa479f663042f7e6211986ba4ec6a7f40d38aab7f3e5dc5a0a049717bcd0ce6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi
  tools:
  - R
  - R/Bioconductor
  - lilikoi
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# pathway-level-clustering

## Summary

Cluster samples on Lilikoi's personalized pathway deregulation score (PDS) matrix — e.g. hierarchical clustering on the selected pathway features — to verify that the pathway-level representation separates disease phenotypes such as ER+ vs ER− breast cancer tissue samples.

## When to use

Use when you have a PDS score matrix (0–1 pathway deregulation scores) restricted to Lilikoi-selected pathway features and want to check whether pathway-level profiles visually separate phenotype classes before or after classification modeling.

## When NOT to use

- Input is raw metabolite measurements rather than a pathway-level PDS matrix — run metab_to_pathway and get_pd_scores first
- No phenotype labels are available, so cluster separation relative to classes cannot be validated
- The goal is group-level pathway enrichment (e.g. MetPA/IMPaLA/MPEA) rather than personalized, sample-level pathway representation and classification

## Inputs

- Metabolite abundance matrix with phenotype labels (e.g. 162 metabolites × 271 ER+/ER− Charité tissue samples)
- PDS score matrix (pathways × samples, scores ranging 0–1) from lilikoi.get_pd_scores against the SMPDB pathway-metabolite database
- Selected pathway list from lilikoi.select_pathways (threshold = 0.42, method = 'gain')

## Outputs

- Hierarchical clustering figure/heatmap of the selected pathway-level (PDS) profiles colored by phenotype
- Selected-pathway PDS submatrix (e.g. 6 selected pathways × 271 samples)
- Information-gain feature ranking table for the selected pathways

## How to apply

First run the Lilikoi pipeline up to feature selection: map metabolites to pathways with lilikoi.metab_to_pathway (searchType 'name'), compute the PDS matrix with lilikoi.get_pd_scores against SMPDB, and select significant pathways with lilikoi.select_pathways (threshold = 0.42, method = 'gain'). Then restrict the PDS matrix to the selected pathway features — in the ER+/ER− Charité tissue dataset (162 metabolites × 271 samples) exactly six pathways are selected, with pantothenate and CoA biosynthesis highest at information gain 0.19 and histidine metabolism second at 0.16. Perform hierarchical clustering on the six selected pathway PDS scores per sample and inspect the dendrogram/heatmap against the ER+/ER− labels; a correct run shows the clustering separating ER+ from ER− samples, with metabolites such as beta-alanine associated with all six selected feature pathways. Write the information-gain table, PDS matrix, and clustering figure to disk as validation artifacts.

## Related tools

- **lilikoi** (R package providing metab_to_pathway, get_pd_scores, select_pathways, and machine_learning used to build the PDS matrix and selected pathway features that are clustered) — https://github.com/lanagarmire/lilikoi
- **R** (Implementation language for the Lilikoi pipeline and hierarchical clustering of pathway-level profiles) — https://cran.r-project.org/web/packages/lilikoi/index.html
- **R/Bioconductor** (Ecosystem in which Lilikoi's metabolomics pathway analysis is executed)

## Examples

```
significantPathways <- lilikoi.select_pathways(PDSmatrix, metaboliteMeasurements, threshold = 0.42, method = "gain"); mlResults <- lilikoi.machine_learning(PDSmatrix, metaboliteMeasurements$Label, significantPathways)
```

## Evaluation signals

- Exactly six pathways are selected at threshold = 0.42 with method = 'gain'
- Top-ranked selected pathway is pantothenate and CoA biosynthesis with information gain 0.19; histidine metabolism is second with 0.16
- PDS scores fall within the 0–1 range
- Hierarchical clustering on the six selected pathway scores visually separates ER+ from ER− samples
- Beta-alanine is associated with all six selected feature pathways

## Limitations

- Lilikoi cannot map roughly 20% of metabolites to standard names in databases, leaving gaps in the pathway mapping
- Levenshtein distance similarity matching may cause metabolite name mismatches during standardization
- Lilikoi lacks AutoML for automatic classification algorithm selection (planned for future)

## Evidence

- [results] the metabolomics profile of the training set is transformed to a pathway-based profile through module 2: dimension transformation: "the metabolomics profile of the training set is transformed to a pathway-based profile through module 2: dimension transformation"
- [methods] significantPathways <- lilikoi.select_pathways(PDSmatrix, metaboliteMeasurements, threshold = 0.42, method = "gain"): "significantPathways <- lilikoi.select_pathways(PDSmatrix, metaboliteMeasurements,
  threshold = 0.42, method = "gain")"
- [abstract] dimension transformation module, which transforms the metabolomic profiles to personalized pathway-based profiles using pathway deregulation scores: "dimension transformation module, which transforms the metabolomic profiles to personalized pathway-based profiles using pathway deregulation scores"
- [results] in the second breast cancer tissue sample set, 120 of 162 metabolites are mapped to standard HMDB IDs: "in the second breast cancer tissue sample set, 120 of 162 metabolites are mapped to standard HMDB IDs"
- [readme] The package is freely available under the GPLv3 license through the github repository at: https://github.com/lanagarmire/lilikoi: "The package is freely available under the GPLv3 license through the github repository at: https://github.com/lanagarmire/lilikoi"
