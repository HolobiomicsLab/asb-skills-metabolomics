---
name: soy-metabolite-reference-curation
description: 'Use when the goal is to reproduce or extend the paper''s evaluation:
  running CyProduct.jar on known CYP450 substrates and non-reactants and computing
  Jaccard scores (TP/(TP+FN+FP)) against reference metabolites and baseline tools.'
when_to_use_negative:
- The input compounds are not human CYP450 substrates or the question concerns non-CYP
  metabolism — the tool and its test sets cover only the nine major CYP450 isoforms
  (1A2 through 3A4).
- The task is SoM (site-of-metabolism) ranking only rather than byproduct prediction
  — then CypBoM's comparison against FAME2/FAME3 is the relevant evaluation, not the
  full MetaboGen pipeline benchmark.
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0154
tools:
- name: CyProduct
  role: Three-module predictor (CypReact filter → CypBoM bond prediction → MetaboGen
    metabolite generation) run via its CLI on the test SDFs
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: CypReact
  role: Reactant/non-reactant filter module; prediction proceeds only if the compound
    is classified as R for a given enzyme
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: GLORY
  role: Baseline tool benchmarked in MaxEfficiency (871 metabolites predicted) and
    MaxCoverage (2376 metabolites predicted) modes
- name: BioTransformer
  role: Baseline ML-based metabolite predictor; its hold-out dataset (13321_2018_324_MOESM6_ESM.sdf)
    is distributed with CyProduct
- name: ADMET Predictor
  role: Commercial baseline tool compared by Jaccard score
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:5942dc74001b2462a38898c935f394066cf3eef6e9a63cc74e6000514b1b21bb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ADMET Predictor
  - BioTransformer
  - CyProduct
  - CypBoM
  - CypReact
  - GLORY
  - MetaboGen
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# soy-metabolite-reference-curation

## Summary

Benchmark CyProduct's three-module CYP450 metabolite-prediction pipeline (CypReact → CypBoM → MetaboGen) on the HDFAME test sets (60Reactants.sdf / 30NonReactants.sdf) and quantify its Jaccard-score improvement over ADMET Predictor, BioTransformer and GLORY (~200% on average).

## When to use

Use when the goal is to reproduce or extend the paper's evaluation: running CyProduct.jar on known CYP450 substrates and non-reactants and computing Jaccard scores (TP/(TP+FN+FP)) against reference metabolites and baseline tools.

## When NOT to use

- The input compounds are not human CYP450 substrates or the question concerns non-CYP metabolism — the tool and its test sets cover only the nine major CYP450 isoforms (1A2 through 3A4).
- The task is SoM (site-of-metabolism) ranking only rather than byproduct prediction — then CypBoM's comparison against FAME2/FAME3 is the relevant evaluation, not the full MetaboGen pipeline benchmark.

## Inputs

- CyProduct.jar executable from the Bitbucket repository
- 60Reactants.sdf (subset of EBoMD2, 68-substrate test set; contains 90 true metabolites)
- 30NonReactants.sdf (non-CYP450 compound set)
- Comma-separated CYP450 enzyme list (1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4) without the 'CYP' prefix
- Published baseline predictions: ADMET Predictor, BioTransformer, GLORY (MaxEfficiency and MaxCoverage) from the paper's PredictedResults folder

## Outputs

- PredictedResults output folder with metabolite structures per enzyme/union of enzymes
- Per-tool Jaccard scores (TP/(TP+FN+FP)) over the union of CYP2C9, 2D6 and 3A4 predictions
- Average percentage improvement of CyProduct over each baseline tool

## How to apply

Download CyProduct.jar plus the datasets from the Bitbucket repository (CyProduct.jar, HDFAME/60Reactants.sdf, 30NonReactants.sdf). Run `java -jar CyProduct.jar QueryMolecule enzymeList OutputFolderPath`, passing an SDF path (or SMILES=... string) and a comma-separated enzyme list without the 'CYP' prefix (e.g. 1A2,2C9,2D6,3A4); enzyme scores are computed as Score=(prob-threshold)/(1-threshold) with threshold=1/(1+beta). Run the tool on 60Reactants.sdf for the union of CYP2C9/2D6/3A4 and on 30NonReactants.sdf to confirm near-zero metabolite predictions for non-reactants. Compare against baseline predictions — GLORY predicted 871 metabolites in MaxEfficiency mode and 2376 in MaxCoverage mode versus 90 true metabolites — by computing the Jaccard score TP/(TP+FN+FP) per tool and enzyme, then averaging the percentage improvement of CyProduct over each baseline.

## Related tools

- **CyProduct** (Three-module predictor (CypReact filter → CypBoM bond prediction → MetaboGen metabolite generation) run via its CLI on the test SDFs) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **CypReact** (Reactant/non-reactant filter module; prediction proceeds only if the compound is classified as R for a given enzyme) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **GLORY** (Baseline tool benchmarked in MaxEfficiency (871 metabolites predicted) and MaxCoverage (2376 metabolites predicted) modes)
- **BioTransformer** (Baseline ML-based metabolite predictor; its hold-out dataset (13321_2018_324_MOESM6_ESM.sdf) is distributed with CyProduct)
- **ADMET Predictor** (Commercial baseline tool compared by Jaccard score)

## Examples

```
java -jar CyProduct.jar HDFAME/60Reactants.sdf 1A2,2A6,2B6,2C8,2C9,2C19,2D6,2E1,3A4 ./PredictedResults/
```

## Evaluation signals

- GLORY reference counts must be consistent: 871 metabolites in MaxEfficiency mode and 2376 in MaxCoverage mode against 90 true metabolites.
- On 30NonReactants.sdf, CyProduct should predict no (or near-zero) metabolites — the CypReact filter should reject non-reactants.
- The computed average percentage improvement of CyProduct's Jaccard score should match the paper's reported ~200% over ADMET Predictor, BioTransformer and GLORY.
- Enzyme names passed to CyProduct must not use the 'CYP' prefix; predictions for the union of 2C9/2D6/3A4 should be assembled from the CyProductResult_ForCYP2C9_2D6_3A4 results folder.
- Per-enzyme scores follow Score = (prob-threshold)/(1-threshold) with threshold = 1/(1+beta) from the CypBoM cost-sensitive super-learner.

## Limitations

- No software version numbers are stated for CDK, WEKA, or the external baseline tools, so exact-version reproduction is not possible from the paper alone.
- The paper's Discussion section (limitations, reproducibility gaps) was not available in the extracted text.
- Reported results are produced by different combinations/subsets of the predicted-results folders, so a single naive CLI run may not exactly reproduce every published number.
- Support is limited to the nine major human CYP450 isoforms; η+η-forming reactions are very rare (4 of 828 reactions in EBoMD, all Cyclization).

## Evidence
