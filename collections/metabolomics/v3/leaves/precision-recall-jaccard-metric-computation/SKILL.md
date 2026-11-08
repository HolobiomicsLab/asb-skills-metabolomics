---
name: precision-recall-jaccard-metric-computation
description: Use when evaluating or reproducing performance ranges of CYP450 metabolite
  prediction tools against a curated ground-truth metabolite dataset (e.g. EBoMD2
  hold-out compounds in 60Reactants.sdf).
when_to_use_negative:
- The goal is site-of-metabolism (SoM) prediction evaluation, where Jaccard is computed
  over atoms/bonds rather than whole metabolite structures
- The tool under test is CyProduct itself with its CypReact filter applied differently
  (reactant vs non-reactant splits require separate handling per the PredictedResults
  folder structure)
- No curated ground-truth metabolite set exists for the query compounds
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3370
tools:
- name: BioTransformer
  role: Run with CYP450 module enabled to generate predicted metabolites for the evaluation
    compounds
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: GLORY
  role: Run in MaxEfficiency and MaxCoverage modes to produce baseline predicted metabolite
    sets (871 and 2376 metabolites vs 90 true)
- name: ADMET Predictor
  role: Commercial tool (Simulations Plus) run with its CYP450 metabolism module to
    generate a predicted metabolite set
- name: CyProduct
  role: ML-based predictor under evaluation; its predicted results per dataset are
    provided in the PredictedResults folder of the repository
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:ad8225c1bf1dbd655d12e0cf31131b7d7195b6f776529b43e852a13031ea74b7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ADMET Predictor
  - BioTransformer
  - CyProduct
  - GLORY
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3370
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# precision-recall-jaccard-metric-computation

## Summary

Benchmarking protocol for in silico metabolism predictors: run competing tools (BioTransformer, GLORY/GLORYx, ADMET Predictor) on a curated test set, match predicted against true metabolites, and compute precision, recall and Jaccard scores per tool.

## When to use

Use when evaluating or reproducing performance ranges of CYP450 metabolite prediction tools against a curated ground-truth metabolite dataset (e.g. EBoMD2 hold-out compounds in 60Reactants.sdf).

## When NOT to use

- The goal is site-of-metabolism (SoM) prediction evaluation, where Jaccard is computed over atoms/bonds rather than whole metabolite structures
- The tool under test is CyProduct itself with its CypReact filter applied differently (reactant vs non-reactant splits require separate handling per the PredictedResults folder structure)
- No curated ground-truth metabolite set exists for the query compounds

## Inputs

- Evaluation test-set compounds as an SDF file (60Reactants.sdf, subset of the EBoMD2 hold-out dataset)
- Curated true metabolites from EBoMD2 for structure matching
- Predicted metabolite sets from each tool (BioTransformer, GLORY MaxEfficiency, GLORY MaxCoverage, GLORYx, ADMET Predictor)

## Outputs

- Per-tool precision, recall and Jaccard scores (Jaccard = TP/(TP+FN+FP))
- Per-tool predicted metabolite counts (e.g. GLORY MaxEfficiency 871 vs MaxCoverage 2376 vs 90 true metabolites)
- Per-isoform metric table for comparison against article baseline ranges

## How to apply

Load the evaluation compounds from 60Reactants.sdf (a 60-compound subset of the 98-compound EBoMD2 hold-out set) and run each predictor on them: BioTransformer with its CYP450 module enabled, GLORY in both MaxEfficiency and MaxCoverage modes, GLORYx, and ADMET Predictor (Simulations Plus) with its CYP450 metabolism module. For context on scale, GLORY's two modes predicted 871 and 2376 metabolites respectively against 90 true metabolites in the article's HDFame comparison. Match each tool's predicted metabolite set against the curated true metabolites using standard structure comparison (exact structure matching), then compute precision = TP/(TP+FP), recall = TP/(TP+FN), and Jaccard = TP/(TP+FN+FP). Tabulate per-tool and per-isoform metrics; the article's reported baseline ranges are precision 0.08-0.46, recall 0.08-0.84 and Jaccard 0.04-0.38, so computed values should fall within (or be compared against) those ranges.

## Related tools

- **BioTransformer** (Run with CYP450 module enabled to generate predicted metabolites for the evaluation compounds) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **GLORY** (Run in MaxEfficiency and MaxCoverage modes to produce baseline predicted metabolite sets (871 and 2376 metabolites vs 90 true))
- **ADMET Predictor** (Commercial tool (Simulations Plus) run with its CYP450 metabolism module to generate a predicted metabolite set)
- **CyProduct** (ML-based predictor under evaluation; its predicted results per dataset are provided in the PredictedResults folder of the repository) — https://bitbucket.org/wishartlab/cyproduct/src/master/

## Examples

```
java -jar CyProduct.jar 60Reactants.sdf 1A2,2A6,2B6,2C8,2C9,2C19,2D6,2E1,3A4 ./PredictedResults/ && python compute_metrics.py --pred PredictedResults/ --truth EBoMD2.sdf --metric jaccard
```

## Evaluation signals

- Computed per-tool metric ranges must reproduce the article's baseline values: precision 0.08-0.46, recall 0.08-0.84, Jaccard 0.04-0.38
- GLORY baseline must yield 871 (MaxEfficiency) and 2376 (MaxCoverage) predicted metabolites against 90 true metabolites on the HDFame comparison set
- TP + FP equals the size of each tool's predicted metabolite set after structure matching
- Jaccard = TP/(TP+FN+FP) must be ≤ both precision and recall for every tool (sanity inequality)
- The evaluation set must contain exactly the 60 reactant compounds from 60Reactants.sdf (subset of the 98-compound EBoMD2 hold-out)

## Limitations

- No software version numbers are stated for the external tools (BioTransformer, GLORY, ADMET Predictor), which may cause reproduction drift
- Metric ranges are highly dependent on the substrate set and reaction types predicted; results do not transfer to other test sets
- ADMET Predictor is commercial (Simulations Plus) and may not be freely reproducible

## Evidence

- [results] Existing in silico metabolism prediction programs achieve a precision of 0.08–0.46, a recall of 0.08–0.84 and a Jaccard score of 0.04–0.38, depending on the substrate molecules and reactions predicted.: [evidence span withheld — anchor status: not_found]
