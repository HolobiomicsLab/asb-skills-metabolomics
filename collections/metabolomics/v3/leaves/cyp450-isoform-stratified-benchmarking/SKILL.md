---
name: cyp450-isoform-stratified-benchmarking
description: Use when multiple in silico CYP450 SoM or metabolite predictors must
  be compared quantitatively per enzyme (1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1,
  3A4) on a common annotated test set, to quantify relative improvement in Jaccard
  score.
when_to_use_negative:
- The task is metabolite/byproduct prediction itself rather than head-to-head tool
  benchmarking — use the CyProduct tool directly instead (java -jar CyProduct.jar
  with an enzyme list).
- 'Benchmarking non-human or non-major CYP450 isoforms: this protocol is defined only
  over the nine major human enzymes (1A2–3A4) and datasets (EBoMD2, 60Reactants.sdf,
  BioTransformer hold-out) annotated for those isoforms.'
- Comparators lack SoM or metabolite predictions on the shared test compounds — the
  Jaccard computation requires alignable predictions per tool–enzyme pair on a common
  annotated set.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3334
tools:
- name: CyProduct
  role: Subject of the benchmark; its CypBoM module's Jaccard SoM scores are compared
    against competitor tools across nine CYP450 isoforms
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: CypBoM
  role: Machine-learned SoM/BoM predictor whose Table 3 Jaccard scores provide the
    benchmark reference values (+25% over FAME2, +36% over FAME3)
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: FAME2
  role: SoM-predicting comparator scored directly on the HDFAME 60Reactants subset;
    benchmark baseline for the 25% improvement claim
- name: FAME3
  role: SoM-predicting comparator scored directly on the HDFAME 60Reactants subset;
    benchmark baseline for the 36% improvement claim
- name: GLORY
  role: Metabolite-predicting comparator evaluated on HDFAME; its MaxEfficiency (871)
    and MaxCoverage (2376) predictions vs 90 true metabolites illustrate overprediction
    effects on Jaccard
- name: BioTransformer
  role: Metabolite-predicting comparator evaluated on its own hold-out set (13321_2018_324_MOESM6_ESM.sdf,
    59 compounds)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:7839a002786c4567f22ea8de74cbcd30ef6ae2985e37afd249cc8ce1e6d5191d
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
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 24
---

# cyp450-isoform-stratified-benchmarking

## Summary

A benchmarking protocol that compares CYP450 metabolism-prediction tools (CyProduct/CypBoM vs FAME2, FAME3, GLORY, GLORYx, BioTransformer, ADMET Predictor, SmartCyp, RS-Predictor, XenoSite, RD-Metabolizer) isoform-by-isoform across the nine major human CYP450s using the Jaccard score TP/(TP+FN+FP) on the EBoMD2/60Reactants.sdf and BioTransformer hold-out sets.

## When to use

Use when multiple in silico CYP450 SoM or metabolite predictors must be compared quantitatively per enzyme (1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4) on a common annotated test set, to quantify relative improvement in Jaccard score.

## When NOT to use

- The task is metabolite/byproduct prediction itself rather than head-to-head tool benchmarking — use the CyProduct tool directly instead (java -jar CyProduct.jar with an enzyme list).
- Benchmarking non-human or non-major CYP450 isoforms: this protocol is defined only over the nine major human enzymes (1A2–3A4) and datasets (EBoMD2, 60Reactants.sdf, BioTransformer hold-out) annotated for those isoforms.
- Comparators lack SoM or metabolite predictions on the shared test compounds — the Jaccard computation requires alignable predictions per tool–enzyme pair on a common annotated set.

## Inputs

- EBoMD2.sdf hold-out dataset (98 compounds with ground-truth BoM annotations)
- HDFAME 60Reactants.sdf subset of EBoMD2 and 30NonReactants.sdf
- BioTransformer hold-out dataset 13321_2018_324_MOESM6_ESM.sdf (59 compounds scored)
- Predicted results per tool from the CyProduct repository's PredictedResults folder
- Predicted SoM or metabolite outputs from competitor tools (FAME2, FAME3, GLORY, GLORYx, BioTransformer, ADMET Predictor, SmartCyp, RS-Predictor, XenoSite, RD-Metabolizer)

## Outputs

- Tools-by-isoform matrix of Jaccard scores TP/(TP+FN+FP) for the nine CYP450 enzymes
- Average Jaccard score per tool across the nine isoforms
- Relative improvement percentages of CyProduct/CypBoM over each competitor (e.g., +25% vs FAME2, +36% vs FAME3, ~30% average SoM gain)

## How to apply

Load each tool's predictions from the CyProduct repository's PredictedResults folder on the common evaluation sets: EBoMD2.sdf hold-out, the HDFAME 60Reactants.sdf subset (used when comparing with FAME2 and GLORY), and the BioTransformer hold-out 13321_2018_324_MOESM6_ESM.sdf. For each tool–enzyme pair among the nine CYP450 isoforms, align predicted sites of metabolism or byproducts against ground-truth annotations and compute the Jaccard score TP/(TP+FN+FP); for metabolite-predicting tools (ADMET Predictor, BioTransformer, GLORY, GLORYx, CyProduct) first convert predicted products back into SoM annotations, while for SoM-predicting tools (FAME2, FAME3, SmartCyp, RS-Predictor, XenoSite, RD-Metabolizer) score SoM predictions directly. Tabulate per-tool/per-enzyme Jaccard scores into a tools-by-isoform matrix, average across the nine isoforms, and report each tool's relative improvement — the paper verifies CypBoM's Jaccard is 25% better than FAME2's and 36% better than FAME3's on Table 3, supporting the ~30% average SoM gain claim. Decision point: choose which PredictedResults subset matches each comparison — e.g., the union of CYP2C9/2D6/3A4 results for 60Reactants.sdf mirror the FAME comparisons, and CyProduct_Results_For59BiotransformerMolecules covers the BioTransformer hold-out.

## Related tools

- **CyProduct** (Subject of the benchmark; its CypBoM module's Jaccard SoM scores are compared against competitor tools across nine CYP450 isoforms) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **CypBoM** (Machine-learned SoM/BoM predictor whose Table 3 Jaccard scores provide the benchmark reference values (+25% over FAME2, +36% over FAME3)) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **FAME2** (SoM-predicting comparator scored directly on the HDFAME 60Reactants subset; benchmark baseline for the 25% improvement claim)
- **FAME3** (SoM-predicting comparator scored directly on the HDFAME 60Reactants subset; benchmark baseline for the 36% improvement claim)
- **GLORY** (Metabolite-predicting comparator evaluated on HDFAME; its MaxEfficiency (871) and MaxCoverage (2376) predictions vs 90 true metabolites illustrate overprediction effects on Jaccard)
- **BioTransformer** (Metabolite-predicting comparator evaluated on its own hold-out set (13321_2018_324_MOESM6_ESM.sdf, 59 compounds))

## Examples

```
java -jar CyProduct.jar CC(=O)Nc1ccc(O)cc1 1A2,2A6,2B6,2C8,2C9,2C19,2D6,2E1,3A4 E:\Users\cyproduct\
```

## Evaluation signals

- Every cell of the tools-by-isoform Jaccard matrix must lie in [0, 1], with all nine isoform columns (1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4) and all twelve tools present.
- The computed average Jaccard improvement of CypBoM over FAME2 should approximate the reported 25%, and over FAME3 the reported 36%.
- The average SoM improvement across tools should approximate the paper's ~30% claim (CyProduct outperforms existing tools 'by an average of 30% (with regard to SoMs)').
- Predicted-product tools (ADMET Predictor, BioTransformer, GLORY, GLORYx, CyProduct) must be converted back to SoM annotations before scoring; scoring them directly on products would invalidate the comparison.
- On the HDFAME set, sanity-check class imbalance effects: GLORY's MaxEfficiency/MaxCoverage modes predicted 871/2376 metabolites against only 90 true metabolites, which should depress Jaccard via false positives.

## Limitations

- Different comparisons use different dataset subsets (60Reactants.sdf for FAME2/GLORY, 30NonReactants.sdf, EBoMD2, and the 59-molecule BioTransformer hold-out), so per-tool scores are not computed on an identical compound set across all tools.
- No software version numbers are stated for CDK, WEKA, or the external comparator tools, limiting exact reproducibility.
- CyProduct's reported score depends on the CypBoM cost-matrix threshold, Score = (prob-threshold)/(1-threshold) with threshold = 1/(1+beta); reproducing scores requires matching the learned beta.
- GLORY's MaxCoverage mode massively overpredicts (2376 vs 90 true metabolites), so Jaccard comparisons are sensitive to each tool's precision/recall trade-off defaults.

## Evidence
