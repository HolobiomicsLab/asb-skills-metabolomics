---
name: cyp450-metabolite-prediction-benchmarking
description: Use when a new or existing in silico metabolism predictor must be compared
  against baseline tools on a CYP450 test set, requiring matched predicted-vs-true
  metabolite sets and precision/recall/Jaccard metrics.
when_to_use_negative:
- The goal is site-of-metabolism (SoM) prediction only, without generating whole-metabolite
  structures — CypBoM/FAME2 SoM comparison is a separate task
- The compounds are non-reactants (e.g. 30NonReactants.sdf); the CypReact filter gates
  prediction and reactant-status metrics are evaluated differently
- Only a single tool's absolute metabolite list is needed without a controlled comparison
  against baselines on the same compound set
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3370
tools:
- name: CyProduct
  role: ML-based CYP450 metabolism predictor under evaluation; run via CyProduct.jar
    with a comma-separated enzyme list
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: BioTransformer
  role: Baseline tool run with its CYP450 module enabled to generate predicted metabolites
    for the relevant isoforms
- name: GLORY
  role: Baseline tool run in both MaxEfficiency and MaxCoverage modes to produce predicted
    metabolite sets
- name: GLORYx
  role: Baseline tool run on the same compounds to produce predicted metabolite sets
- name: ADMET Predictor
  role: Commercial baseline (Simulations Plus) run with its CYP450 metabolism module
    to generate its predicted metabolite set
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:0446af45a01ac4c7a4712411cc2d5cddb8077151828369c050edc95edbb9089b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ADMET Predictor
  - BioTransformer
  - CyProduct
  - GLORY
  - GLORYx
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3370
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cyp450-metabolite-prediction-benchmarking

## Summary

A benchmarking protocol for evaluating in silico CYP450 metabolism prediction tools (BioTransformer, GLORY, GLORYx, ADMET Predictor, CyProduct) by computing precision, recall and Jaccard scores of predicted metabolites against curated true metabolites from EBoMD2.

## When to use

Use when a new or existing in silico metabolism predictor must be compared against baseline tools on a CYP450 test set, requiring matched predicted-vs-true metabolite sets and precision/recall/Jaccard metrics.

## When NOT to use

- The goal is site-of-metabolism (SoM) prediction only, without generating whole-metabolite structures — CypBoM/FAME2 SoM comparison is a separate task
- The compounds are non-reactants (e.g. 30NonReactants.sdf); the CypReact filter gates prediction and reactant-status metrics are evaluated differently
- Only a single tool's absolute metabolite list is needed without a controlled comparison against baselines on the same compound set

## Inputs

- Evaluation test-set compounds as SDF (60Reactants.sdf, a subset of the 98-compound EBoMD2 hold-out dataset)
- Curated true metabolite sets from EBoMD2 for matching
- Tool outputs: BioTransformer (CYP450 module), GLORY (MaxEfficiency and MaxCoverage modes), GLORYx, ADMET Predictor (CYP450 module), CyProduct

## Outputs

- Per-tool and per-isoform precision, recall and Jaccard scores (Jaccard = TP / (TP+FN+FP))
- Predicted metabolite sets per tool per compound
- Comparison table verifying reproduction of baseline ranges (precision 0.08–0.46, recall 0.08–0.84, Jaccard 0.04–0.38)

## How to apply

Load the evaluation compounds from 60Reactants.sdf (a subset of the 98-compound EBoMD2 hold-out dataset). Run each tool on the same compounds: BioTransformer with its CYP450 module enabled, GLORY in both MaxEfficiency and MaxCoverage modes, GLORYx, and ADMET Predictor's CYP450 metabolism module (CyProduct via its jar with the nine CYP450 enzymes 1A2,2A6,2B6,2C8,2C9,2C19,2D6,2E1,3A4). Match each tool's predicted metabolites against the curated true metabolites in EBoMD2 using standard structure comparison, then compute precision, recall and Jaccard (TP / (TP+FN+FP)) per tool and per isoform. As a calibration check, the computed baseline ranges should reproduce the article-reported values: precision 0.08–0.46, recall 0.08–0.84, Jaccard 0.04–0.38; on the HDFame subset GLORY MaxEfficiency predicted 871 and MaxCoverage 2376 metabolites versus 90 true metabolites, illustrating the coverage/precision trade-off between GLORY modes.

## Related tools

- **CyProduct** (ML-based CYP450 metabolism predictor under evaluation; run via CyProduct.jar with a comma-separated enzyme list) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **BioTransformer** (Baseline tool run with its CYP450 module enabled to generate predicted metabolites for the relevant isoforms)
- **GLORY** (Baseline tool run in both MaxEfficiency and MaxCoverage modes to produce predicted metabolite sets)
- **GLORYx** (Baseline tool run on the same compounds to produce predicted metabolite sets)
- **ADMET Predictor** (Commercial baseline (Simulations Plus) run with its CYP450 metabolism module to generate its predicted metabolite set)

## Examples

```
java -jar CyProduct.jar CC(=O)Nc1ccc(O)cc1 1A2,2A6,2B6,2C8,2C9,2C19,2D6,2E1,3A4 E:/Users/cyproduct/
```

## Evaluation signals

- Computed baseline metric ranges must match the article-reported ranges: precision 0.08–0.46, recall 0.08–0.84, Jaccard 0.04–0.38 on the evaluation test set
- GLORY MaxEfficiency/MaxCoverage metabolite counts on HDFame should be ~871 and ~2376 against 90 true metabolites
- All tools must be run on the identical compound set (60Reactants.sdf) so metrics are directly comparable per tool and per isoform
- Each predicted metabolite must be matched against the curated EBoMD2 true metabolites using standard structure comparison before TP/FP/FN counting
- Jaccard must be computed as TP / (TP+FN+FP), not as TP/(TP+FP) or TP/(TP+FN)

## Limitations

- Baseline performance depends strongly on the set of substrate molecules and types of reactions predicted (hence the wide reported ranges)
- GLORY MaxCoverage mode predicts vastly more metabolites (2376 vs 90 true), inflating recall at the cost of precision
- No software version numbers are stated for CDK, WEKA, or the external tools; ADMET Predictor requires a commercial (Simulations Plus) license

## Evidence

- [results] Baseline metric ranges: [evidence span withheld — anchor status: not_found]
