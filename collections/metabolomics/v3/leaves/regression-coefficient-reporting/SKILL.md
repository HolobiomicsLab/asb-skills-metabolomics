---
name: regression-coefficient-reporting
description: Use when a fitted metabolite-pathway regression in Lilikoi v2.0 must
  be documented as a coefficient table with significance calls (P < 0.05) for metabolites
  mapped to a selected pathway.
when_to_use_negative:
- The goal is disease classification or feature selection — use Lilikoi's classification
  predictor or featuresSelection modules instead, since regression reporting assumes
  a pathway set has already been selected.
- The goal is prognosis modeling — Lilikoi v2.0 handles survival via the Cox-PH and
  Cox-nnet models (lilikoi.prognosis), not the metabolite-pathway regression module.
- The data have not yet been transformed from metabolite names/IDs into pathway-space
  features (no PDS matrix or Metabolite_pathway_table is available).
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Lilikoi v2.0
  role: Provides lilikoi.meta_path() for single-variate metabolite-pathway regression
    and lilikoi.KEGGplot() for the accompanying Pathview visualization
  repo: https://github.com/lanagarmire/lilikoi2
- name: R
  role: Programming environment in which the Lilikoi v2.0 regression and reporting
    workflow is executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:750fdd80d9fce1d9b7ff460848fdf719baedbc5a2637928074e0727bceec38d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi2
  tools:
  - Lilikoi v2.0
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# regression-coefficient-reporting

## Summary

Reports single-variate metabolite-pathway regression results from Lilikoi v2.0 as a coefficient table, identifying which metabolites mapped to a selected pathway (e.g., alanine-aspartate-glutamate metabolism, KEGG pathid 00250) significantly (P < 0.05) contribute to the pathway-level measure.

## When to use

Use when a fitted metabolite-pathway regression in Lilikoi v2.0 must be documented as a coefficient table with significance calls (P < 0.05) for metabolites mapped to a selected pathway.

## When NOT to use

- The goal is disease classification or feature selection — use Lilikoi's classification predictor or featuresSelection modules instead, since regression reporting assumes a pathway set has already been selected.
- The goal is prognosis modeling — Lilikoi v2.0 handles survival via the Cox-PH and Cox-nnet models (lilikoi.prognosis), not the metabolite-pathway regression module.
- The data have not yet been transformed from metabolite names/IDs into pathway-space features (no PDS matrix or Metabolite_pathway_table is available).

## Inputs

- Metabolite dataset (e.g., PR000284 workbench metabolite data, 207 plasma samples) loaded via lilikoi.Loaddata
- Metabolite-to-pathway mapping table from lilikoi.MetaTOpathway('name')
- PDS matrix from lilikoi.PDSfun(Metabolite_pathway_table)
- Selected pathway list from lilikoi.featuresSelection (e.g., the pathway with KEGG pathid 00250)

## Outputs

- Coefficient table of single-variate metabolite-pathway regression coefficients for all metabolites mapped to the selected pathway
- Significance calls (P < 0.05) flagging metabolites that significantly contribute to the pathway feature
- Optional Pathview graph for the pathway (pathid 00250) rendered by lilikoi.KEGGplot

## How to apply

Run Lilikoi v2.0's metabolite-pathway regression via lilikoi.meta_path(), passing the PDS matrix built by lilikoi.PDSfun(), the pathway set selected by lilikoi.featuresSelection(), and the metabolite-to-pathway table produced by lilikoi.MetaTOpathway('name'), specifying the pathway of interest (e.g., the alanine-aspartate-glutamate metabolism pathway, KEGG pathid 00250). Fit a single-variate regression model for each metabolite mapped to that pathway, relating the metabolite to the pathway-level measure. Record the regression coefficients for each mapped metabolite — for pathid 00250 this includes citric acid, pyruvate, 5-phosphoribosylamine, glutamine, oxaloacetate, asparagine, succinate, and aspartate — in a coefficient table. Call metabolites with P < 0.05 as significant contributors (in the ER− breast-cancer plasma dataset, citric acid, pyruvate, 5-phosphoribosylamine, glutamine, oxaloacetate, and asparagine were significantly increased). Optionally pair the table with the Pathview figure (lilikoi.KEGGplot, pathid '00250', specie 'hsa') so coefficients can be interpreted in the context of the KEGG pathway diagram.

## Related tools

- **Lilikoi v2.0** (Provides lilikoi.meta_path() for single-variate metabolite-pathway regression and lilikoi.KEGGplot() for the accompanying Pathview visualization) — https://github.com/lanagarmire/lilikoi2
- **R** (Programming environment in which the Lilikoi v2.0 regression and reporting workflow is executed)

## Examples

```
lilikoi.meta_path(PDSmatrix = PDSmatrix, selected_Pathways_Weka = selected_Pathways_Weka, Metabolite_pathway_table = Metabolite_pathway_table, pathway = "Pyruvate Metabolism")
```

## Evaluation signals

- Fitted coefficients reproduce the reported values for pathid 00250: glutamine 0.378, oxaloacetate 0.575, asparagine 0.997, succinate -0.435.
- The metabolites flagged significant at P < 0.05 include citric acid, pyruvate, 5-phosphoribosylamine, glutamine, oxaloacetate, and asparagine for the alanine-aspartate-glutamate metabolism pathway.
- The coefficient table contains one row per mapped metabolite, covering all eight metabolites named in the workflow (citric acid, pyruvate, 5-phosphoribosylamine, glutamine, oxaloacetate, asparagine, succinate, aspartate).
- The Pathview image for pathid 00250 (specie 'hsa') is generated without error when lilikoi.KEGGplot is run on the same pathway.
- The set of metabolites regressed matches those mapped to the pathway in Metabolite_pathway_table, not the full metabolite panel.

## Limitations

- Regression is single-variate — each metabolite is modeled separately against the pathway-level measure, so joint/metabolite-set effects are not captured.
- Reported coefficient values and P < 0.05 significance calls are specific to the plasma breast-cancer dataset (207 samples, ER− vs ER+ disease); other datasets may yield different coefficients and significance patterns.
- Interpretation of the Pathview overlay depends on correct metabolite-to-KEGG ID mapping via lilikoi.MetaTOpathway('name'); mis-mapped metabolite names would distort which metabolites appear in the regression table.

## Evidence

- [abstract] Lilikoi v2.0 supports data preprocessing, exploratory analysis, pathway visualization, and metabolite pathway regression.: "Lilikoi v2.0 supports data preprocessing, exploratory analysis, pathway visualization, and metabolite pathway regression."
- [intro] Third, we augmented the pathway-based metabolomics analysis with metabolite-pathway regression and pathway visualization.: "Third, we augmented the pathway-based metabolomics analysis with metabolite-pathway regression and pathway visualization."
- [readme] lilikoi.meta_path(PDSmatrix = PDSmatrix, selected_Pathways_Weka = selected_Pathways_Weka, Metabolite_pathway_table = Metabolite_pathway_table, pathway = "Pyruvate Metabolism"): "# Metabolites-pathway regression
lilikoi.meta_path(PDSmatrix = PDSmatrix, selected_Pathways_Weka = selected_Pathways_Weka, Metabolite_pathway_table = Metabolite_pathway_table, pathway = "Pyruvate"
- [other] citric acid, pyruvate, 5-phosphoribosylamine, glutamine, oxaloacetate, and asparagine were significantly (P < 0.05) increased in patients with ER− disease.: [evidence span withheld — anchor status: not_found]
