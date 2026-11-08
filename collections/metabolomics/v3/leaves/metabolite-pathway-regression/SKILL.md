---
name: metabolite-pathway-regression
description: Use when you have a PDS pathway-activity matrix and a metabolite-to-pathway
  mapping table from Lilikoi's feature-mapping/dimension-transformation steps, and
  you need to attribute a selected pathway feature to its constituent metabolites.
when_to_use_negative:
- You only need pathway-level classification/prognosis and do not need per-metabolite
  attribution within a pathway
- You have not yet run Lilikoi's feature mapping (MetaTOpathway) and dimension transformation
  (PDSfun) steps, so no PDSmatrix or Metabolite_pathway_table exists
edam_operation: http://edamontology.org/operation_0394
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_2269
tools:
- name: Lilikoi v2.0
  role: R package providing lilikoi.meta_path for metabolite-pathway regression and
    lilikoi.KEGGplot for pathway visualization
  repo: github.com/lanagarmire/lilikoi2
- name: R
  role: Programming environment in which the Lilikoi v2.0 regression and visualization
    functions are executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:12a93dea83a571b478820847b787dbd429a002808242fe40583de5ef51148a1b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Lilikoi v2.0
  - R
  edam_operation: http://edamontology.org/operation_0394
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# metabolite-pathway-regression

## Summary

Lilikoi v2.0's metabolite-pathway regression module (lilikoi.meta_path) fits single-variate regression models relating each metabolite mapped to a selected pathway to the pathway-level measure, identifying which metabolites significantly (P < 0.05) contribute to that pathway feature.

## When to use

Use when you have a PDS pathway-activity matrix and a metabolite-to-pathway mapping table from Lilikoi's feature-mapping/dimension-transformation steps, and you need to attribute a selected pathway feature to its constituent metabolites.

## When NOT to use

- You only need pathway-level classification/prognosis and do not need per-metabolite attribution within a pathway
- You have not yet run Lilikoi's feature mapping (MetaTOpathway) and dimension transformation (PDSfun) steps, so no PDSmatrix or Metabolite_pathway_table exists

## Inputs

- PDSmatrix from lilikoi.PDSfun (pathway-level activity matrix)
- Metabolite_pathway_table from lilikoi.MetaTOpathway (metabolite-to-pathway annotations)
- selected_Pathways_Weka from lilikoi.featuresSelection (significant pathway list)
- Metabolite intensity matrix (e.g., plasma_breast_cancer.csv workbench metabolite data)

## Outputs

- Single-variate regression coefficients and P-values for each metabolite mapped to the selected pathway
- Coefficient table for metabolites of interest (e.g., citric acid, pyruvate, glutamine, oxaloacetate, asparagine, succinate)
- Optional KEGG pathway figure (e.g., pathid 00250 for hsa) from lilikoi.KEGGplot with metabolite values overlaid

## How to apply

First run the upstream Lilikoi v2.0 pipeline: load metabolite data with lilikoi.Loaddata (e.g., the plasma_breast_cancer.csv workbench data, 207 plasma samples), convert metabolite names to pathway annotations with lilikoi.MetaTOpathway('name') to build Metabolite_pathway_table, transform metabolites into pathway-level features with lilikoi.PDSfun, and select significant pathways with lilikoi.featuresSelection (threshold=0.50, method='gain'). Then call lilikoi.meta_path with PDSmatrix, selected_Pathways_Weka, Metabolite_pathway_table, and the pathway of interest (e.g., 'Pyruvate Metabolism' or the alanine-aspartate-glutamate pathway, KEGG pathid 00250) to fit single-variate regression of each mapped metabolite against the pathway-level measure. Optionally render the matching KEGG pathway diagram with lilikoi.KEGGplot (pathid='00250', specie='hsa') to overlay metabolite-level values on the pathway. Record regression coefficients and P-values per metabolite; significance is judged at P < 0.05 (e.g., in the ER− breast cancer analysis, citric acid, pyruvate, 5-phosphoribosylamine, glutamine, oxaloacetate, and asparagine were significantly increased, with coefficients such as glutamine 0.378, oxaloacetate 0.575, asparagine 0.997, succinate -0.435).

## Related tools

- **Lilikoi v2.0** (R package providing lilikoi.meta_path for metabolite-pathway regression and lilikoi.KEGGplot for pathway visualization) — github.com/lanagarmire/lilikoi2
- **R** (Programming environment in which the Lilikoi v2.0 regression and visualization functions are executed)

## Examples

```
lilikoi.meta_path(PDSmatrix = PDSmatrix, selected_Pathways_Weka = selected_Pathways_Weka, Metabolite_pathway_table = Metabolite_pathway_table, pathway = "Pyruvate Metabolism")
```

## Evaluation signals

- Fitted coefficients reproduce reported values, e.g., glutamine 0.378, oxaloacetate 0.575, asparagine 0.997, succinate -0.435 for the alanine-aspartate-glutamate pathway
- Metabolites reported significant at P < 0.05 (citric acid, pyruvate, 5-phosphoribosylamine, glutamine, oxaloacetate, asparagine) are flagged in the output table
- Only metabolites mapped to the selected pathway (via Metabolite_pathway_table) appear in the regression results
- lilikoi.KEGGplot generates the Pathview image for pathid '00250' (specie 'hsa') without error

## Limitations

- Regression is single-variate per metabolite against the pathway-level measure, so it does not model metabolite-metabolite interactions within the pathway
- Depends on the pathway set surviving the prior feature-selection step (lilikoi.featuresSelection with threshold=0.50, method='gain'); pathways not selected cannot be attributed
- Successful name-to-pathway conversion via lilikoi.MetaTOpathway('name') is required; metabolite names not mapped to KEGG/HMDB pathways cannot be regressed

## Evidence

- [intro] Augment the pathway-based metabolomics analysis with metabolite-pathway regression and pathway visualization: "Third, we augmented the pathway-based metabolomics analysis with metabolite-pathway regression and pathway visualization."
- [readme] lilikoi.meta_path call signature: "lilikoi.meta_path(PDSmatrix = PDSmatrix, selected_Pathways_Weka = selected_Pathways_Weka, Metabolite_pathway_table = Metabolite_pathway_table, pathway = "Pyruvate Metabolism")"
- [abstract] Module scope of Lilikoi v2.0: "Lilikoi v2.0 supports data preprocessing, exploratory analysis, pathway visualization, and metabolite pathway regression."
- [readme] Feature selection parameters: "selected_Pathways_Weka= lilikoi.featuresSelection(PDSmatrix,threshold= 0.50,method="gain")"
