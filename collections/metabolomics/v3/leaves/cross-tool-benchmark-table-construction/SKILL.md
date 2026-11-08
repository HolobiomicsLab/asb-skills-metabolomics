---
name: cross-tool-benchmark-table-construction
description: Use when evaluating a prediction tool against published baselines on
  a shared benchmark set, needing a per-tool/per-condition metric table with verifiable
  reference counts.
when_to_use_negative:
- The task is a single-tool hold-out evaluation (e.g. CypBoM vs FAME2 SoM Jaccard
  on EBoMD2), not a multi-tool benchmark table
- No shared ground-truth metabolite list or published baseline predictions exist for
  the comparison set
- The benchmark set lacks a non-reactant negative-control subset, so false-positive
  behavior cannot be assessed
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3370
tools:
- name: CyProduct
  role: Target tool whose metabolite predictions are benchmarked against baselines;
    run via CyProduct.jar CLI
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: ADMET Predictor
  role: Baseline in silico metabolism predictor included in the Jaccard comparison
- name: BioTransformer
  role: Baseline ML-based metabolism predictor included in the Jaccard comparison
- name: GLORY
  role: Baseline tool evaluated in MaxEfficiency (871 predicted metabolites) and MaxCoverage
    (2376 predicted metabolites) modes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:813bc50e9aa0da7bcc7f9e335c35483fd830e89569ca264504b4fe610a5eef77
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3370
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# cross-tool-benchmark-table-construction

## Summary

Constructs a reproducible benchmark table comparing a target tool's metabolite predictions against competing in silico predictors using a shared test set and a common metric (Jaccard score = TP/(TP+FN+FP)). In the CyProduct paper this quantified an average ~200% Jaccard improvement over ADMET Predictor, BioTransformer and GLORY.

## When to use

Use when evaluating a prediction tool against published baselines on a shared benchmark set, needing a per-tool/per-condition metric table with verifiable reference counts.

## When NOT to use

- The task is a single-tool hold-out evaluation (e.g. CypBoM vs FAME2 SoM Jaccard on EBoMD2), not a multi-tool benchmark table
- No shared ground-truth metabolite list or published baseline predictions exist for the comparison set
- The benchmark set lacks a non-reactant negative-control subset, so false-positive behavior cannot be assessed

## Inputs

- 60Reactants.sdf — subset of EBoMD2 containing 60 known CYP450 substrates with 90 true metabolites
- 30NonReactants.sdf — 30 non-reactant compounds used to confirm near-zero false positives
- Baseline predictions for ADMET Predictor, BioTransformer and GLORY (MaxEfficiency/MaxCoverage) from the paper and PredictedResults folder
- True metabolite reference lists per CYP450 isoform

## Outputs

- Per-tool, per-CYP450-isoform Jaccard score table (TP/(TP+FN+FP))
- Average percentage improvement of the target tool over each baseline
- Negative-control check showing no (or near-zero) metabolites predicted for the 30NonReactants.sdf set

## How to apply

Assemble a benchmark set that contains both positives and negatives — here HDFAME's 60Reactants.sdf (60 known CYP450 substrates, 90 true metabolites) and 30NonReactants.sdf — and run the target tool over all nine CYP450 isoforms (1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4), writing predictions to a PredictedResults folder. Pull baseline predictions (ADMET Predictor, BioTransformer, GLORY MaxEfficiency and MaxCoverage modes) from the paper and its PredictedResults repository folder rather than re-running them. Compute Jaccard = TP/(TP+FN+FP) per tool against the true metabolite list, aggregating over the union of CYP2C9, 2D6 and 3A4 predictions, then average the percentage improvement of the target tool across enzymes. Validate the table against paper-reported anchors: GLORY MaxEfficiency predicted 871 metabolites and MaxCoverage 2376 versus 90 true metabolites, and the average improvement should reproduce the reported ~200%.

## Related tools

- **CyProduct** (Target tool whose metabolite predictions are benchmarked against baselines; run via CyProduct.jar CLI) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **ADMET Predictor** (Baseline in silico metabolism predictor included in the Jaccard comparison)
- **BioTransformer** (Baseline ML-based metabolism predictor included in the Jaccard comparison)
- **GLORY** (Baseline tool evaluated in MaxEfficiency (871 predicted metabolites) and MaxCoverage (2376 predicted metabolites) modes)

## Examples

```
java -jar CyProduct.jar 60Reactants.sdf 2C9,2D6,3A4 ./PredictedResults/CyProductResult_ForCYP2C9_2D6_3A4
```

## Evaluation signals

- Computed Jaccard improvement reproduces the paper's reported ~200% average advantage of CyProduct over existing tools
- GLORY reference counts match: 871 metabolites in MaxEfficiency mode and 2376 in MaxCoverage mode versus 90 true metabolites
- Running CyProduct on 30NonReactants.sdf yields no (or near-zero) predicted metabolites, confirming the negative control
- Jaccard formula applied consistently as TP/(TP+FN+FP) across every tool and enzyme column in the table
- Predicted output folder structure matches the repository's PredictedResults layout (CyProductResult_ForCYP2C9_2D6_3A4 for the 3-CYP union)

## Limitations

- Benchmark results are produced by combining subsets of the repository's PredictedResults files, so table construction depends on picking the correct predicted-metabolite combination per comparison
- The paper's stated average improvement (200%) refers specifically to metabolite (byproduct) prediction; SoM-only comparison shows a smaller ~30% advantage
- GLORY's MaxCoverage mode predicts 2376 metabolites versus 90 true metabolites, indicating baseline outputs are heavily inflated and scores depend on mode choice

## Evidence

- [results] Jaccard metric definition used across tools: [evidence span withheld — anchor status: not_found]
