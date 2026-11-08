---
name: metabolite-structure-matching-against-reference-sets
description: Use when you must reproduce or compare precision/recall/Jaccard performance
  of CYP450 metabolite-prediction tools against a curated true-metabolite reference
  set like EBoMD2 or the BioTransformer hold-out dataset.
when_to_use_negative:
- Input compound is predicted to be a non-reactant (30NonReactants.sdf-type set) —
  the CypReact filter gates the CyProduct prediction pipeline itself rather than structure
  matching
- The goal is SoM (site-of-metabolism) bond-level evaluation against FAME2/FAME3 —
  that uses CypBoM Jaccard on BoMX−Y pairs (Table 3), not whole-metabolite structure
  matching
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: GLORY
  role: Run in MaxEfficiency and MaxCoverage modes to generate predicted metabolite
    sets that are matched against the 90 true HDFAME metabolites
- name: BioTransformer
  role: CYP450 module run on each test compound to generate predicted metabolites
    for structure matching
- name: ADMET Predictor
  role: Commercial CYP450 metabolism module producing a predicted metabolite set for
    the same compounds
- name: CyProduct
  role: ML-based predictor whose PredictedResults folder outputs are matched against
    reference sets to compute its metrics
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:e51ae9c1b62c244e8c21a1d83fefb1c09ac7fbbfea7118708817cb11fd9194ae
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ADMET Predictor
  - BioTransformer
  - CyProduct
  - GLORY
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# metabolite-structure-matching-against-reference-sets

## Summary

Benchmark workflow for in silico CYP450 metabolism predictors: run tools such as BioTransformer, GLORY/GLORYx and ADMET Predictor on the EBoMD2-derived 60Reactants.sdf test set, then match each tool's predicted metabolite set against the curated true metabolites to compute precision, recall and Jaccard (TP/(TP+FN+FP)).

## When to use

Use when you must reproduce or compare precision/recall/Jaccard performance of CYP450 metabolite-prediction tools against a curated true-metabolite reference set like EBoMD2 or the BioTransformer hold-out dataset.

## When NOT to use

- Input compound is predicted to be a non-reactant (30NonReactants.sdf-type set) — the CypReact filter gates the CyProduct prediction pipeline itself rather than structure matching
- The goal is SoM (site-of-metabolism) bond-level evaluation against FAME2/FAME3 — that uses CypBoM Jaccard on BoMX−Y pairs (Table 3), not whole-metabolite structure matching

## Inputs

- 60Reactants.sdf evaluation test-set compounds (subset of EBoMD2, from the HDFAME folder)
- Curated true metabolites from EBoMD2 (90 true metabolites on the HDFAME subset)
- Predicted metabolite sets from BioTransformer (CYP450 module), GLORY (MaxEfficiency and MaxCoverage), GLORYx, ADMET Predictor (CYP450 module) and CyProduct

## Outputs

- Per-tool and per-isoform precision, recall and Jaccard (TP/(TP+FN+FP)) scores
- Counts of predicted metabolites per tool and mode (e.g. 871 vs 2376 vs 90 true)
- Reproduced baseline metric ranges: precision 0.08–0.46, recall 0.08–0.84, Jaccard 0.04–0.38

## How to apply

Load the evaluation compounds from 60Reactants.sdf (a subset of the 98-compound EBoMD2 hold-out dataset) and run each predictor — BioTransformer with its CYP450 module, GLORY in both MaxEfficiency and MaxCoverage modes, GLORYx, and ADMET Predictor's CYP450 metabolism module — to produce per-tool predicted metabolite sets. For each tool, match predicted metabolites against the curated true metabolites in EBoMD2 using standard structure comparison, treating a matched prediction as TP and computing precision, recall and Jaccard as TP/(TP+FN+FP). Tabulate per-tool and per-isoform metrics; a correct reproduction yields the article-reported ranges (precision 0.08–0.46, recall 0.08–0.84, Jaccard 0.04–0.38). Rationale: predicted-set size strongly affects these metrics — GLORY MaxEfficiency predicted 871 metabolites and MaxCoverage 2376 versus only 90 true metabolites on HDFAME — so the matching step must be applied per configuration to expose precision/coverage trade-offs.

## Related tools

- **GLORY** (Run in MaxEfficiency and MaxCoverage modes to generate predicted metabolite sets that are matched against the 90 true HDFAME metabolites)
- **BioTransformer** (CYP450 module run on each test compound to generate predicted metabolites for structure matching)
- **ADMET Predictor** (Commercial CYP450 metabolism module producing a predicted metabolite set for the same compounds)
- **CyProduct** (ML-based predictor whose PredictedResults folder outputs are matched against reference sets to compute its metrics) — https://bitbucket.org/wishartlab/cyproduct/src/master/

## Examples

```
java -jar CyProduct.jar CC(=O)Nc1ccc(O)cc1 1A2,2A6,2B6,2C8,2C9,2C19,2D6,2E1,3A4 ./predicted/  # then match ./predicted/*.sdf metabolites against EBoMD2 truths and compute Jaccard = TP/(TP+FN+FP)
```

## Evaluation signals

- Computed baseline ranges must match the article-reported values: precision 0.08–0.46, recall 0.08–0.84, Jaccard 0.04–0.38
- On HDFAME (60Reactants.sdf), GLORY MaxEfficiency must reproduce 871 predicted metabolites and MaxCoverage 2376 against 90 true metabolites
- Jaccard is computed exactly as TP/(TP+FN+FP) with structure-identity matching against curated EBoMD2 metabolites
- Per-tool, per-isoform metrics must be tabulated across the nine CYP450 isoforms (1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4)
- Manuscript-level results come from specific combinations/subsets of the PredictedResults folders, so matched outputs must be reconciled against those folder contents

## Limitations

- Very small true-metabolite denominators (90 true metabolites on HDFAME) make precision/recall/Jaccard highly sensitive to individual matches
- η+η-forming reactions are extremely rare (4 of 828 reactions in EBoMD, all Cyclization), so rare-reaction metrics are unstable
- No software version numbers are stated for CDK, WEKA or external tools, and the Discussion section content was unavailable, limiting reproducibility detail

## Evidence
