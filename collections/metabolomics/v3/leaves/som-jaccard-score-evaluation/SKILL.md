---
name: som-jaccard-score-evaluation
description: Use when comparing SoM or byproduct predictions from multiple CYP450
  metabolism predictors on a common hold-out test set and you need a single metric
  (Jaccard score) plus a percent-improvement margin per competitor.
when_to_use_negative:
- The task is reactant/non-reactant classification for a CYP450 enzyme (CypReact filter
  evaluation) rather than SoM localization scoring
- Predictions and ground truth are not on a common compound set with matched atom-level
  SoM annotations, making TP/FN/FP alignment impossible
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3520
tools:
- name: CyProduct
  role: ML-based predictor under evaluation; its per-isoform SoM/metabolite predictions
    on EBoMD2 form the focal row of the Jaccard matrix
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: CypBoM
  role: CyProduct's SoM-prediction module whose Jaccard score is directly compared
    to FAME2's and FAME3's in Table 3
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: FAME2
  role: Baseline SoM predictor against which the 25% Jaccard improvement is computed
    (scored on 60Reactants.sdf)
- name: FAME3
  role: Baseline SoM predictor against which the 36% Jaccard improvement is computed
- name: GLORY
  role: Metabolite-predicting competitor whose products must be converted back to
    SoM annotations before Jaccard scoring
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:fbdd09e12c8019628b681380be266ca57b6b55474e4924b733dbe378e1e81a16
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 27
---

# som-jaccard-score-evaluation

## Summary

Benchmark workflow for evaluating CYP450 site-of-metabolism (SoM) prediction tools by computing per-tool, per-enzyme Jaccard scores TP/(TP+FN+FP) against ground-truth metabolite annotations (e.g. EBoMD2), then deriving the relative improvement of one predictor (CypBoM/CyProduct) over competitors like FAME2, FAME3 and GLORY.

## When to use

Use when comparing SoM or byproduct predictions from multiple CYP450 metabolism predictors on a common hold-out test set and you need a single metric (Jaccard score) plus a percent-improvement margin per competitor.

## When NOT to use

- The task is reactant/non-reactant classification for a CYP450 enzyme (CypReact filter evaluation) rather than SoM localization scoring
- Predictions and ground truth are not on a common compound set with matched atom-level SoM annotations, making TP/FN/FP alignment impossible

## Inputs

- Predicted results from CyProduct's PredictedResults folder (e.g. CyProductResults_For_9CYPs on EBoMD2) for the nine CYP450 isoforms
- Predicted SoM or metabolite outputs from competitor tools (FAME2, FAME3, GLORY, GLORYx, BioTransformer, ADMET Predictor, SmartCyp, RS-Predictor, XenoSite, RD-Metabolizer)
- Ground-truth annotated hold-out datasets: EBoMD2.sdf, 60Reactants.sdf (subset of EBoMD2), and 13321_2018_324_MOESM6_ESM.sdf (BioTransformer hold-out)

## Outputs

- Matrix of per-tool, per-CYP450-isoform Jaccard scores TP/(TP+FN+FP) (rows = tools, columns = nine isoforms)
- Per-tool average Jaccard across the nine enzymes and per-competitor relative improvement percentages versus CyProduct

## How to apply

Load each tool's predicted outputs on the shared evaluation set — for CyProduct this is the PredictedResults folder containing CyProductResults_For_9CYPs (per-enzyme predictions on EBoMD2), results on 60Reactants.sdf for the FAME2/GLORY comparison, and results on 13321_2018_324_MOESM6_ESM.sdf for the BioTransformer hold-out. Align predicted SoMs or back-converted metabolites against ground-truth annotations for each of the nine CYP450 isoforms (1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4) and compute Jaccard = TP/(TP+FN+FP) per tool–enzyme pair. For metabolite-emitting tools (ADMET Predictor, BioTransformer, GLORY, GLORYx, CyProduct) convert predicted products back to SoM annotations before scoring; for SoM-emitting tools (SmartCyp, FAME2, FAME3, RS-Predictor, XenoSite, RD-Metabolizer) score directly. Tabulate the tool-by-isoform matrix, average across the nine enzymes per tool, and report relative improvement — the paper's benchmark is CypBoM's Jaccard being 25% better than FAME2's and 36% better than FAME3's, consistent with CyProduct's stated ~30% average SoM advantage.

## Related tools

- **CyProduct** (ML-based predictor under evaluation; its per-isoform SoM/metabolite predictions on EBoMD2 form the focal row of the Jaccard matrix) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **CypBoM** (CyProduct's SoM-prediction module whose Jaccard score is directly compared to FAME2's and FAME3's in Table 3) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **FAME2** (Baseline SoM predictor against which the 25% Jaccard improvement is computed (scored on 60Reactants.sdf))
- **FAME3** (Baseline SoM predictor against which the 36% Jaccard improvement is computed)
- **GLORY** (Metabolite-predicting competitor whose products must be converted back to SoM annotations before Jaccard scoring)

## Examples

```
java -jar CyProduct.jar CC(=O)Nc1ccc(O)cc1 1A2,2A6,2B6,2C8,2C9,2C19,2D6,2E1,3A4 E:\Users\cyproduct\
```

## Evaluation signals

- Every cell of the tool-by-isoform matrix is a valid Jaccard value in [0, 1], with all nine isoform columns and all competitor tools present
- The computed average SoM improvement of CyProduct over the best-performing competitor approximates the paper's reported ~30% value
- The pair-level margins reproduce the published figures: 25% over FAME2 and 36% over FAME3 on Table 3 (left/right)
- Metabolite-format predictions (GLORY, GLORYx, BioTransformer, ADMET Predictor, CyProduct) are converted to SoM annotations before scoring — mixing formats inflates/deflates TP counts
- On the HDFAME comparison, competitor overprediction is checkable: GLORY MaxEfficiency predicted 871 and MaxCoverage 2376 metabolites against 90 true metabolites

## Limitations

- The Jaccard margin depends on the chosen hold-out set (EBoMD2 / 60Reactants.sdf vs BioTransformer's hold-out); results reported in the manuscript combine different subsets of PredictedResults, so scores are not directly transferable across test sets
- No software version numbers are stated for competitor tools or for CDK/WEKA, limiting exact reproducibility of baseline scores
- η+η-forming reactions are extremely rare (4 of 828 reactions in EBoMD), so metrics are dominated by η−η bond cases

## Evidence
