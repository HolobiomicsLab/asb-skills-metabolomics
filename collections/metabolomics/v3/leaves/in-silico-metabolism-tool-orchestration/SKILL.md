---
name: in-silico-metabolism-tool-orchestration
description: Use when benchmarking or comparing in silico metabolism prediction software
  on a common substrate set, e.g. to reproduce baseline precision/recall/Jaccard ranges
  before evaluating a new predictor.
when_to_use_negative:
- The goal is SoM (site-of-metabolism) atom-level prediction rather than whole-metabolite
  byproduct prediction — use SoM tools like FAME2/FAME3 or CypBoM directly instead.
- Only a single predictor is being run for production prediction with no comparison
  or benchmarking intent — orchestration overhead and metric computation are unnecessary.
- True metabolite ground truth is unavailable for the substrate set, making precision/recall/Jaccard
  computation impossible.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3370
- http://edamontology.org/topic_0209
tools:
- name: CyProduct
  role: ML-based CYP450 byproduct predictor run via CyProduct.jar on the shared query
    compounds and enzyme list
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: BioTransformer
  role: Baseline ML predictor run with its CYP450 module on the same compounds
- name: GLORY
  role: Baseline rule-based predictor run in MaxEfficiency and MaxCoverage modes to
    produce predicted metabolite sets
- name: GLORYx
  role: Baseline predictor run on the same compounds to generate an additional predicted
    metabolite set
- name: ADMET Predictor
  role: Commercial knowledge-based predictor (Simulations Plus) run with its CYP450
    metabolism module
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:a929bfdc7a53dcc225391920afd9fe8b80545af8995e821c8122551db80966ec
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ADMET Predictor
  - BioTransformer
  - CyProduct
  - GLORY
  - GLORYx
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3370
  - http://edamontology.org/topic_0209
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# in-silico-metabolism-tool-orchestration

## Summary

Runs multiple in silico CYP450 metabolism predictors (CyProduct, BioTransformer, GLORY/GLORYx, ADMET Predictor) on a shared evaluation compound set and scores each tool's predicted metabolites against curated true metabolites using precision, recall and Jaccard metrics.

## When to use

Use when benchmarking or comparing in silico metabolism prediction software on a common substrate set, e.g. to reproduce baseline precision/recall/Jaccard ranges before evaluating a new predictor.

## When NOT to use

- The goal is SoM (site-of-metabolism) atom-level prediction rather than whole-metabolite byproduct prediction — use SoM tools like FAME2/FAME3 or CypBoM directly instead.
- Only a single predictor is being run for production prediction with no comparison or benchmarking intent — orchestration overhead and metric computation are unnecessary.
- True metabolite ground truth is unavailable for the substrate set, making precision/recall/Jaccard computation impossible.

## Inputs

- Evaluation compound set as SDF (e.g. 60Reactants.sdf, subset of EBoMD2 hold-out dataset)
- Curated true metabolite set from EBoMD2 for structure matching
- CYP450 isoform list (e.g. 1A2,2A6,2B6,2C8,2C9,2C19,2D6,2E1,3A4)

## Outputs

- Per-tool predicted metabolite sets (SMILES/SDF structures per substrate and isoform)
- Per-tool and per-isoform precision, recall and Jaccard score table
- Verified baseline metric ranges for comparison against new predictors

## How to apply

Load the evaluation compounds (e.g. 60Reactants.sdf, a 60-compound subset of the 98-compound EBoMD2 hold-out dataset) and run each tool on the identical inputs: BioTransformer with its CYP450 module, GLORY in both MaxEfficiency and MaxCoverage modes plus GLORYx, ADMET Predictor's CYP450 metabolism module, and CyProduct via its CLI with a comma-separated enzyme list (enzymes given without the 'CYP' prefix, e.g. 1A2,2C9,2D6,3A4). Match each tool's predicted metabolite set against the curated true metabolites in EBoMD2 using standard structure comparison, then compute per-tool (and optionally per-isoform) precision = TP/(TP+FP), recall = TP/(TP+FN) and Jaccard = TP/(TP+FN+FP). Tabulate all metrics and check the computed ranges against known baseline values (precision 0.08-0.46, recall 0.08-0.84, Jaccard 0.04-0.38); note that permissive tools can predict hundreds of metabolites against ~90 true ones (GLORY MaxEfficiency predicted 871, MaxCoverage 2376 on HDFame), so precision/Jaccard penalize over-prediction while recall rewards coverage.

## Related tools

- **CyProduct** (ML-based CYP450 byproduct predictor run via CyProduct.jar on the shared query compounds and enzyme list) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **BioTransformer** (Baseline ML predictor run with its CYP450 module on the same compounds)
- **GLORY** (Baseline rule-based predictor run in MaxEfficiency and MaxCoverage modes to produce predicted metabolite sets)
- **GLORYx** (Baseline predictor run on the same compounds to generate an additional predicted metabolite set)
- **ADMET Predictor** (Commercial knowledge-based predictor (Simulations Plus) run with its CYP450 metabolism module)

## Examples

```
java -jar CyProduct.jar 60Reactants.sdf 1A2,2A6,2B6,2C8,2C9,2C19,2D6,2E1,3A4 ./cyproduct_predictions/
```

## Evaluation signals

- Computed baseline metric ranges match the article-reported values: precision 0.08-0.46, recall 0.08-0.84, Jaccard 0.04-0.38 on the evaluation test set.
- Every tool was run on the identical compound set (60Reactants.sdf) so predicted-metabolite counts are directly comparable (e.g. GLORY 871 and 2376 vs 90 true metabolites).
- Jaccard score is reproducible as TP/(TP+FN+FP) from the per-tool confusion-matrix counts.
- Predicted metabolites are reported per CYP450 isoform as well as per tool, enabling per-isoform breakdown.
- Predicted structures matched against true metabolites by standard structure comparison (e.g. InChI/SMILES equivalence), not by name.

## Limitations

- Baseline predictors vary enormously in permissiveness — GLORY MaxCoverage predicted 2376 metabolites vs 90 true ones — so raw recall alone is misleading without precision/Jaccard.
- Metric ranges depend strongly on the substrate set and reaction types predicted; ranges from 60Reactants.sdf will not generalize to other compound collections.
- ADMET Predictor requires a commercial Simulations Plus license; no software version numbers are stated for the external tools, complicating exact reproduction.

## Evidence

- [results] Existing in silico metabolism prediction programs achieve a precision of 0.08–0.46, a recall of 0.08–0.84 and a Jaccard score of 0.04–0.38: [evidence span withheld — anchor status: not_found]
- [methods] Jaccard formula for evaluation: [evidence span withheld — anchor status: not_found]
