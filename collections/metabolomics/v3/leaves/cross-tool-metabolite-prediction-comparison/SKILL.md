---
name: cross-tool-metabolite-prediction-comparison
description: 'Decision support for choosing among CYP450 SoM or byproduct predictors:
  when a practitioner has a common evaluation test set (e.g., EBoMD2, the BioTransformer
  hold-out 13321_2018_324_MOESM6_ESM.sdf, or the HDFAME 60Reactants.'
when_to_use_negative:
- If no ground-truth SoM/metabolite annotation is available for the test compounds
  — Jaccard scoring requires labeled positives to compute TP, FN, and FP.
- If the goal is to evaluate one tool's internal performance rather than cross-tool
  comparison — a single-tool evaluation does not need a per-tool/per-isoform Jaccard
  matrix.
- If the evaluation set contains compounds that some tools (via CypReact filter) classify
  as non-reactants and you do not wish to penalize filtered-out predictions — this
  skill assumes common treatment of reactant/non-reactant classifications across tools.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3336
tools:
- name: CyProduct
  role: ML-based predictor (CypReact + CypBoM + MetaboGen) whose outputs are scored
    against competitors; invoked via CyProduct.jar with SMILES or SDF query and a
    CYP450 enzyme list.
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: CypBoM
  role: The SoM-prediction module of CyProduct whose Jaccard advantage over FAME2
    (25%) and FAME3 (36%) is the key benchmark result being reproduced by this skill.
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: FAME2
  role: SoM-predicting competitor tool scored on the common test set to compute the
    relative Jaccard improvement of CypBoM.
- name: FAME3
  role: SoM-predicting competitor tool scored on the common test set; 36% Jaccard
    advantage attributed to CypBoM on the SoM task.
- name: GLORY
  role: Metabolite-predicting competitor whose outputs are converted back to SoM annotations
    before Jaccard scoring.
- name: GLORYx
  role: Metabolite-predicting competitor scored on the same common test set; also
    used for the HDFAME comparison with 60Reactants.sdf.
- name: ADMET Predictor
  role: Metabolite-predicting commercial tool scored on the common test set; its predicted
    products are converted back to SoM labels for Jaccard computation.
- name: BioTransformer
  role: Metabolite-predicting tool scored on its own hold-out dataset (13321_2018_324_MOESM6_ESM.sdf)
    as part of the cross-tool comparison.
- name: SmartCyp
  role: SoM-predicting competitor tool scored directly on SoM labels without conversion.
- name: RS-Predictor
  role: SoM-predicting competitor tool scored directly on SoM labels.
- name: XenoSite
  role: SoM-predicting competitor tool scored directly on SoM labels.
- name: RD-Metabolizer
  role: SoM-predicting competitor tool scored directly on SoM labels.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:40390fc523c558afa95af3e302b160018a8244d628e19ba22919fa71a24224c8
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
  - RD-Metabolizer
  - RS-Predictor
  - SmartCyp
  - XenoSite
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# cross-tool-metabolite-prediction-comparison

## Summary

Benchmark multiple in silico CYP450 metabolism predictors by scoring their outputs on common datasets against the Jaccard similarity TP/(TP+FN+FP), then tabulate per-tool/per-isoform performance to quantify relative advantages (e.g., CypBoM's Jaccard being 25% better than FAME2's and 36% better than FAME3's for SoMs).

## When to use

Decision support for choosing among CYP450 SoM or byproduct predictors: when a practitioner has a common evaluation test set (e.g., EBoMD2, the BioTransformer hold-out 13321_2018_324_MOESM6_ESM.sdf, or the HDFAME 60Reactants.sdf subset) and needs head-to-head Jaccard-score comparisons across knowledge-based and ML-based tools.

## When NOT to use

- If no ground-truth SoM/metabolite annotation is available for the test compounds — Jaccard scoring requires labeled positives to compute TP, FN, and FP.
- If the goal is to evaluate one tool's internal performance rather than cross-tool comparison — a single-tool evaluation does not need a per-tool/per-isoform Jaccard matrix.
- If the evaluation set contains compounds that some tools (via CypReact filter) classify as non-reactants and you do not wish to penalize filtered-out predictions — this skill assumes common treatment of reactant/non-reactant classifications across tools.

## Inputs

- Common evaluation test set (SDF/SMILES), e.g. EBoMD2.sdf, 60Reactants.sdf, or 13321_2018_324_MOESM6_ESM.sdf
- Ground-truth metabolite/SoM annotations for the test set
- Output predictions from each competing tool (predicted metabolite structures or predicted SoM labels with scores)
- Enzyme list, e.g. 1A2,2A6,2B6,2C8,2C9,2C19,2D6,2E1,3A4

## Outputs

- Per-tool/per-isoform Jaccard score matrix (rows = tools, columns = 9 CYP450 isoforms)
- Average Jaccard across isoforms per tool
- Relative-improvement percentages (e.g. CyProduct vs FAME2, vs FAME3, vs GLORY)

## How to apply

Run each tool (CyProduct, ADMET Predictor, BioTransformer, GLORY/GLORYx, SmartCyp, FAME2/FAME3, RS-Predictor, XenoSite, RD-Metabolizer, etc.) on the same evaluation set, e.g. EBoMD2 with the nine CYP450 isoforms 1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4. Align each tool's predictions (products for ADMET Predictor/BioTransformer/GLORY/GLORYx/CyProduct, SoMs for SmartCyp/FAME2/FAME3/RS-Predictor/XenoSite/RD-Metabolizer) against the ground-truth metabolite/SoM annotations and compute the Jaccard score TP/(TP+FN+FP) per tool-enzyme pair. Assemble a matrix with tools as rows and the nine isoforms as columns; average across isoforms per tool to derive relative improvement, verifying that CyProduct's average SoM gain approximates the paper's reported ~30%. For CyProduct specifically, invoke via the CyProduct.jar CLI and, where appropriate, disable the CypReact filter by passing useCypReact=false to the BioTransformerAPI, since non-reactant filtering can alter which compounds are scored. Validate that every matrix cell lies in [0,1], that all nine isoform columns and twelve tools are present, and that computed average improvement over the best competitor is consistent with the paper's ~30% claim.

## Related tools

- **CyProduct** (ML-based predictor (CypReact + CypBoM + MetaboGen) whose outputs are scored against competitors; invoked via CyProduct.jar with SMILES or SDF query and a CYP450 enzyme list.) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **CypBoM** (The SoM-prediction module of CyProduct whose Jaccard advantage over FAME2 (25%) and FAME3 (36%) is the key benchmark result being reproduced by this skill.) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **FAME2** (SoM-predicting competitor tool scored on the common test set to compute the relative Jaccard improvement of CypBoM.)
- **FAME3** (SoM-predicting competitor tool scored on the common test set; 36% Jaccard advantage attributed to CypBoM on the SoM task.)
- **GLORY** (Metabolite-predicting competitor whose outputs are converted back to SoM annotations before Jaccard scoring.)
- **GLORYx** (Metabolite-predicting competitor scored on the same common test set; also used for the HDFAME comparison with 60Reactants.sdf.)
- **ADMET Predictor** (Metabolite-predicting commercial tool scored on the common test set; its predicted products are converted back to SoM labels for Jaccard computation.)
- **BioTransformer** (Metabolite-predicting tool scored on its own hold-out dataset (13321_2018_324_MOESM6_ESM.sdf) as part of the cross-tool comparison.)
- **SmartCyp** (SoM-predicting competitor tool scored directly on SoM labels without conversion.)
- **RS-Predictor** (SoM-predicting competitor tool scored directly on SoM labels.)
- **XenoSite** (SoM-predicting competitor tool scored directly on SoM labels.)
- **RD-Metabolizer** (SoM-predicting competitor tool scored directly on SoM labels.)

## Examples

```
java -jar CyProduct.jar CC(=O)Nc1ccc(O)cc1 1A2,2A6,2B6,2C8,2C9,2C19,2D6,2E1,3A4 E:\Users\cyproduct\
```

## Evaluation signals

- Each Jaccard matrix cell lies within [0, 1]; a value > 1 or < 0 indicates an alignment or counting error.
- The matrix contains all twelve tools and all nine CYP450 isoform columns (1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4) with no missing cells.
- Average per-tool Jaccard across the nine isoforms reproduces the paper's relative improvement: CyProduct/CypBoM ~30% better on average for SoMs and a comparable margin for byproduct prediction.
- CypBoM's computed Jaccard on the common set exceeds FAME2's by roughly 25% and FAME3's by roughly 36% — failing this check indicates inconsistent treatment of SoM-vs-metabolite annotation conversion between tools.
- For metabolite-predicting tools (ADMET Predictor, BioTransformer, GLORY, GLORYx, CyProduct), converted SoM labels are used; using raw product sets instead would systematically inflate FPs and depress Jaccard.

## Limitations

- The comparison depends on a shared evaluation set; tools differ in which subsets they were run on (e.g. EBoMD2 vs the HDFAME 60Reactants/30NonReactants subsets vs the BioTransformer hold-out), and results in the manuscript 'are produced by using different combinations of the predicted metabolites mentioned above (or their subsets).'
- Some tools are only SoM predictors while others output full metabolite structures; converting products back to SoM annotations introduces ambiguity that affects Jaccard comparability.
- CyProduct's CypReact filter can be disabled (useCypReact=false) via the BioTransformerAPI; results obtained with the filter off are not directly comparable to those with the filter on.
- Reactive η-η bonds make up only ~5% of the EBoMD dataset, so Jaccard on rare positives is sensitive to small counts of TP/FN/FP.

## Evidence
