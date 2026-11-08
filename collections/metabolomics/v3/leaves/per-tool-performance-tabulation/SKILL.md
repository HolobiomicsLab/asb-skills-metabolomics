---
name: per-tool-performance-tabulation
description: Use when comparing in silico metabolism predictors on a common test set
  (e.g., EBoMD2, 60Reactants.sdf, or BioTransformer's hold-out SDF) and you need per-tool,
  per-enzyme Jaccard matrices to compute relative improvement margins.
when_to_use_negative:
- The goal is to evaluate CyProduct's internal modules (CypReact, CypBoM, MetaboGen)
  via cross-validation on EBoMD training data rather than head-to-head external tool
  comparison
- The evaluation set lacks ground-truth metabolite annotations, making Jaccard computation
  impossible
- The comparison involves non-CYP450 enzymes outside the nine isoforms (1A2, 2A6,
  2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4) covered by CyProduct
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_0209
- http://edamontology.org/topic_3336
tools:
- name: CyProduct
  role: Metabolism predictor whose per-isoform predictions on EBoMD2 and the BioTransformer
    hold-out are tabulated against competitor tools
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: CypBoM
  role: CyProduct's SoM/BoM prediction module whose Jaccard scores are compared against
    FAME2 and FAME3 (Table 3)
- name: FAME2
  role: SoM-predicting competitor benchmark; its Jaccard scores serve as the baseline
    for the 25% improvement claim
- name: FAME3
  role: SoM-predicting competitor benchmark; its Jaccard scores serve as the baseline
    for the 36% improvement claim
- name: BioTransformer
  role: Competitor metabolite predictor evaluated on its 59-compound hold-out dataset
    (13321_2018_324_MOESM6_ESM.sdf)
- name: GLORY
  role: Competitor metabolite predictor evaluated on HDFAME (60Reactants.sdf); note
    MaxEfficiency/MaxCoverage modes predicted 871/2376 metabolites vs 90 true ones
- name: GLORYx
  role: Competitor metabolite predictor included in the tools × isoform Jaccard matrix
- name: SmartCyp
  role: SoM-predicting competitor scored directly on atom sites in the Jaccard matrix
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:30deea82db0f830ff48b22938eabec11d183ad5b7daba5a45d3f423152245286
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ADMET Predictor
  - BioTransformer
  - CyProduct
  - CypBoM
  - CypReact
  - FAME2
  - FAME3
  - GLORY
  - GLORYx
  - MetaboGen
  - Meteor Nexus
  - SmartCyp
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_0209
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 28
---

# per-tool-performance-tabulation

## Summary

A benchmarking skill that tabulates per-tool, per-CYP450-isoform Jaccard scores (TP/(TP+FN+FP)) for SoM and byproduct prediction from the PredictedResults outputs of the CyProduct repository, enabling quantitative comparison of CyProduct/CypBoM against competitors like FAME2, FAME3, and BioTransformer.

## When to use

Use when comparing in silico metabolism predictors on a common test set (e.g., EBoMD2, 60Reactants.sdf, or BioTransformer's hold-out SDF) and you need per-tool, per-enzyme Jaccard matrices to compute relative improvement margins.

## When NOT to use

- The goal is to evaluate CyProduct's internal modules (CypReact, CypBoM, MetaboGen) via cross-validation on EBoMD training data rather than head-to-head external tool comparison
- The evaluation set lacks ground-truth metabolite annotations, making Jaccard computation impossible
- The comparison involves non-CYP450 enzymes outside the nine isoforms (1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4) covered by CyProduct

## Inputs

- PredictedResults folder outputs from the CyProduct repository (e.g., CyProductResults_For_9CYPs for EBoMD2, CyProductResult_ForCYP2C9_2D6_3A4 for 60Reactants.sdf, CyProduct_Results_For59BiotransformerMolecules)
- Ground-truth annotated test sets: EBoMD2.sdf, HDFAME/60Reactants.sdf, HDFAME/30NonReactants.sdf, and 13321_2018_324_MOESM6_ESM.sdf (BioTransformer hold-out, 59 compounds)
- Predictions from competitor tools: FAME2, FAME3, GLORY, GLORYx, SmartCyp, RS-Predictor, XenoSite, RD-Metabolizer, ADMET Predictor, BioTransformer, Meteor Nexus
- Mapping of the nine target isoforms: CYP1A2, CYP2A6, CYP2B6, CYP2C8, CYP2C9, CYP2C19, CYP2D6, CYP2E1, CYP3A4

## Outputs

- A tools × isoforms matrix of Jaccard scores TP/(TP+FN+FP), each cell in [0, 1]
- Per-tool average Jaccard across the nine isoforms for SoM prediction
- Relative improvement percentages of CypBoM/CyProduct over each competitor (e.g., 25% over FAME2, 36% over FAME3)
- Analogous tabulation for byproduct (metabolite) prediction to compare against CyProduct's SoM margin

## How to apply

Load the tool prediction outputs from the PredictedResults folder of the CyProduct repository for each evaluation dataset (EBoMD2/60Reactants.sdf subset and 13321_2018_324_MOESM6_ESM.sdf BioTransformer hold-out). For each of the nine CYP450 isoforms (1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4), align each tool's predictions against ground-truth metabolite annotations and compute the Jaccard score TP/(TP+FN+FP) per tool–enzyme pair. Score SoM-predicting tools (SmartCyp, FAME2, FAME3, RS-Predictor, XenoSite, RD-Metabolizer) directly on atom sites, but first convert product-predicting tools' (ADMET Predictor, BioTransformer, GLORY, GLORYx, CyProduct) predicted metabolites back into SoM annotations so all tools are on a common scale. Tabulate the scores into a matrix with tools as rows and isoforms as columns, compute each tool's average Jaccard across enzymes, and derive relative improvements — expecting CypBoM to be ~25% better than FAME2 and ~36% better than FAME3 on SoMs, consistent with the paper's ~30% average SoM gain. Validate that every matrix cell is a valid Jaccard value in [0, 1] and that all nine isoform columns and all twelve tools are present.

## Related tools

- **CyProduct** (Metabolism predictor whose per-isoform predictions on EBoMD2 and the BioTransformer hold-out are tabulated against competitor tools) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **CypBoM** (CyProduct's SoM/BoM prediction module whose Jaccard scores are compared against FAME2 and FAME3 (Table 3))
- **FAME2** (SoM-predicting competitor benchmark; its Jaccard scores serve as the baseline for the 25% improvement claim)
- **FAME3** (SoM-predicting competitor benchmark; its Jaccard scores serve as the baseline for the 36% improvement claim)
- **BioTransformer** (Competitor metabolite predictor evaluated on its 59-compound hold-out dataset (13321_2018_324_MOESM6_ESM.sdf))
- **GLORY** (Competitor metabolite predictor evaluated on HDFAME (60Reactants.sdf); note MaxEfficiency/MaxCoverage modes predicted 871/2376 metabolites vs 90 true ones)
- **GLORYx** (Competitor metabolite predictor included in the tools × isoform Jaccard matrix)
- **SmartCyp** (SoM-predicting competitor scored directly on atom sites in the Jaccard matrix)

## Examples

```
java -jar CyProduct.jar CC(=O)Nc1ccc(O)cc1 1A2,2A6,2B6,2C8,2C9,2C19,2D6,2E1,3A4 E:\Users\cyproduct\
```

## Evaluation signals

- Every cell of the tools × isoforms Jaccard matrix is in [0, 1]
- All nine isoform columns (CYP1A2, CYP2A6, CYP2B6, CYP2C8, CYP2C9, CYP2C19, CYP2D6, CYP2E1, CYP3A4) and all twelve tools are present in the matrix
- Computed average improvement of CypBoM over the best-performing SoM competitor approximates the paper's reported ~30% (specifically 25% over FAME2 and 36% over FAME3)
- Predicted products from metabolite-level tools (ADMET Predictor, BioTransformer, GLORY, GLORYx, CyProduct) are converted back to SoM annotations before scoring, so Jaccard values are comparable across tool types
- Tabulation reproduces the paper's Table 3 relationships: CypBoM > FAME3 > FAME2 in Jaccard score on SoMs

## Limitations

- Product-predicting tools require a conversion step back to SoM annotations before Jaccard scoring, which can introduce ambiguity in atom mapping
- GLORY's output volume differs drastically by mode (871 metabolites in MaxEfficiency vs 2376 in MaxCoverage vs 90 true metabolites on HDFAME), strongly affecting FP counts and Jaccard scores
- The manuscript's reported results combine different subsets of the PredictedResults folders, so exact reproduction requires knowing which folder subsets correspond to which table
- Comparisons on HDFAME use only the 60Reactants/30NonReactants subsets of EBoMD2, not the full dataset

## Evidence
