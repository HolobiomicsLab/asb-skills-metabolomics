---
name: predicted-metabolite-set-matching
description: Use when a structure-based metabolism predictor (e.g. CyProduct, GLORY,
  BioTransformer, ADMET Predictor) has produced predicted metabolite lists and you
  need a quantitative comparison against known true metabolites.
when_to_use_negative:
- When the evaluation target is site-of-metabolism (SoM) prediction rather than whole-molecule
  byproducts — the paper used a separate Jaccard evaluation for SoMs (CypBoM vs FAME2/FAME3).
- When no curated reference metabolite set exists for the input substrates, since
  the Jaccard score requires a complete true-metabolite list to compute FN.
- When evaluating a tool on non-CYP450 biotransformations — CyProduct and its benchmark
  cover only the nine major human CYP450 enzymes (1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6,
  2E1, 3A4).
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0209
tools:
- name: CyProduct
  role: ML-based metabolism predictor whose Jaccard-score output is being benchmarked;
    run via CyProduct.jar on QueryMolecule/enzymeList
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: GLORY
  role: Baseline metabolite predictor compared in MaxEfficiency (871 metabolites)
    and MaxCoverage (2376 metabolites) modes against 90 true metabolites
- name: BioTransformer
  role: Baseline metabolism predictor included in the Jaccard-score comparison
- name: ADMET Predictor
  role: Commercial baseline metabolism predictor included in the Jaccard-score comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:547c1832f215f4be9d6898d6b5a505097ad97ca502d784410be2601c80eac77b
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
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0209
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# predicted-metabolite-set-matching

## Summary

Evaluates a metabolite-prediction tool's output by matching its predicted metabolite set against a reference set of true metabolites using the Jaccard score (TP/(TP+FN+FP)), as done to benchmark CyProduct against ADMET Predictor, BioTransformer and GLORY.

## When to use

Use when a structure-based metabolism predictor (e.g. CyProduct, GLORY, BioTransformer, ADMET Predictor) has produced predicted metabolite lists and you need a quantitative comparison against known true metabolites.

## When NOT to use

- When the evaluation target is site-of-metabolism (SoM) prediction rather than whole-molecule byproducts — the paper used a separate Jaccard evaluation for SoMs (CypBoM vs FAME2/FAME3).
- When no curated reference metabolite set exists for the input substrates, since the Jaccard score requires a complete true-metabolite list to compute FN.
- When evaluating a tool on non-CYP450 biotransformations — CyProduct and its benchmark cover only the nine major human CYP450 enzymes (1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4).

## Inputs

- Predicted metabolite structure lists (e.g. SDF/output folder like PredictedResults) from each tool under comparison
- Reference (ground-truth) metabolite list, e.g. the 90 true metabolites of the 60Reactants.sdf subset
- Input substrate set (e.g. 60Reactants.sdf, 68 known CYP450 substrates) and optional non-reactant set (30NonReactants.sdf)

## Outputs

- Per-enzyme (or per-enzyme-union) Jaccard scores (TP/(TP+FN+FP)) for each tool
- Average percentage improvement of one tool's Jaccard score over each baseline tool
- Count of predicted metabolites per tool (e.g. GLORY MaxEfficiency 871 vs MaxCoverage 2376 against 90 true metabolites)

## How to apply

Collect the tool's predicted metabolite structures (e.g. from the PredictedResults output folder) for the same input substrates, and assemble the ground-truth metabolite list (e.g. the 90 true metabolites associated with the 60Reactants.sdf subset of EBoMD2). For each tool and each enzyme or enzyme union (CyProduct's benchmark used the union of CYP2C9, 2D6 and 3A4), compute the Jaccard score as TP/(TP+FN+FP), where TP is the number of correctly predicted true metabolites, FN the missed true metabolites and FP the false predictions. Compute the score per enzyme (or per enzyme union) and then average the relative improvement of one tool's Jaccard score over the baselines across the evaluated enzymes — CyProduct's reported benchmark was a ~200% average improvement. Optionally, also run the predictor on a non-reactant set (30NonReactants.sdf) to confirm near-zero metabolite predictions, which checks false-positive behavior on compounds that should not be metabolized by the given CYP450 isoforms.

## Related tools

- **CyProduct** (ML-based metabolism predictor whose Jaccard-score output is being benchmarked; run via CyProduct.jar on QueryMolecule/enzymeList) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **GLORY** (Baseline metabolite predictor compared in MaxEfficiency (871 metabolites) and MaxCoverage (2376 metabolites) modes against 90 true metabolites)
- **BioTransformer** (Baseline metabolism predictor included in the Jaccard-score comparison)
- **ADMET Predictor** (Commercial baseline metabolism predictor included in the Jaccard-score comparison)

## Examples

```
java -jar CyProduct.jar 60Reactants.sdf 2C9,2D6,3A4 PredictedResults/
```

## Evaluation signals

- Computed Jaccard score for CyProduct on the union of CYP2C9, 2D6 and 3A4 over 60Reactants.sdf should exceed baseline scores by roughly the reported ~200% average improvement.
- GLORY reference counts must reproduce the paper's values: 871 metabolites in MaxEfficiency mode and 2376 in MaxCoverage mode versus 90 true metabolites.
- Running on 30NonReactants.sdf should yield zero (or near-zero) predicted metabolites, confirming CypReact-filter false-positive control.
- TP+FN must equal the size of the reference metabolite set (e.g. 90 for the HDFAME union test), acting as a bookkeeping check on the matching step.

## Limitations

- Score is computed for a union of enzymes (CYP2C9/2D6/3A4 in the benchmark); results for the full nine-isoform union are reported separately and may differ.
- High-FP predictors are penalized heavily: GLORY MaxCoverage predicted 2376 metabolites against only 90 true metabolites, so Jaccard favors precision over coverage.
- No software version numbers are stated for the external tools (CDK, WEKA, BioTransformer, GLORY), limiting exact reproducibility.

## Evidence

- [intro] Jaccard score definition used for benchmarking: [evidence span withheld — anchor status: not_found]
