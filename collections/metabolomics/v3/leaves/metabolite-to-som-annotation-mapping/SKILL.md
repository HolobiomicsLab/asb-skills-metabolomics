---
name: metabolite-to-som-annotation-mapping
description: Use when comparing a metabolite/Byproduct-predicting tool against SoM-only
  predictors on a common benchmark, so predicted products must be converted to atom-level
  SoM labels (e.g., TP/(TP+FN+FP) Jaccard over CYP450 isoforms).
when_to_use_negative:
- When the tool already outputs SoM atom annotations directly (e.g. FAME2, FAME3,
  XenoSite) — mapping products back to SoMs is unnecessary and can introduce alignment
  errors
- When the evaluation target is byproduct (product-level) accuracy rather than atom-level
  SoM agreement — in that case score whole metabolite structures instead
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0209
tools:
- name: CyProduct
  role: Metabolite-predicting tool whose byproduct outputs are converted to SoM annotations
    for Jaccard comparison; run via CyProduct.jar over the nine CYP450 enzymes
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: CypBoM
  role: CyProduct's bond-of-metabolism module whose reactive <X,Y> bond predictions
    directly yield substrate atom SoM labels
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: MetaboGen
  role: Generates metabolites from predicted BoM<X−Y>s using SMIRKS transformation
    rules; the applied SMIRKS edits define which substrate atoms count as sites of
    metabolism
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: FAME2
  role: SoM-only comparator; its atom-level predictions are scored directly without
    product-to-SoM conversion (CypBoM is 25% better on SoMs)
- name: FAME3
  role: SoM-only comparator; scored directly at atom level (CypBoM is 36% better on
    SoMs)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:b19f2d783db5dfde5042759b6de7c10a15ef7f8d04eeea477c162bc3439c472d
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
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0209
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8636363636363636
n_claims: 22
---

# metabolite-to-som-annotation-mapping

## Summary

Maps predicted metabolite structures (byproducts) back onto the parent substrate to recover site-of-metabolism (SoM) atom annotations, enabling head-to-head Jaccard comparison between metabolite-predicting tools (CyProduct, GLORY, BioTransformer) and SoM-predicting tools (FAME2, FAME3) on CYP450 data.

## When to use

Use when comparing a metabolite/Byproduct-predicting tool against SoM-only predictors on a common benchmark, so predicted products must be converted to atom-level SoM labels (e.g., TP/(TP+FN+FP) Jaccard over CYP450 isoforms).

## When NOT to use

- When the tool already outputs SoM atom annotations directly (e.g. FAME2, FAME3, XenoSite) — mapping products back to SoMs is unnecessary and can introduce alignment errors
- When the evaluation target is byproduct (product-level) accuracy rather than atom-level SoM agreement — in that case score whole metabolite structures instead

## Inputs

- Predicted metabolite structures (SDF or SMILES) from metabolite-predicting tools, e.g. CyProduct output in the PredictedResults folder of the repository
- Parent substrate structures with ground-truth SoM annotations from EBoMD2.sdf or 60Reactants.sdf
- Predicted reactant/bond labels from SoM-predicting tools (FAME2, FAME3) on the same substrates

## Outputs

- Per-substrate, per-isoform binary SoM annotation vectors derived from predicted metabolites
- Tool-by-isoform Jaccard score matrix (rows = 12 tools, columns = 9 CYP450 isoforms)
- Relative improvement percentages of CyProduct/CypBoM over competitors (e.g. 25% vs FAME2, 36% vs FAME3, ~30% average SoM gain)

## How to apply

Run each metabolite-predicting tool (CyProduct via CyProduct.jar, plus ADMET Predictor, BioTransformer, GLORY, GLORYx) over the common evaluation set, e.g. the EBoMD2 / 60Reactants.sdf subset or BioTransformer's 13321_2018_324_MOESM6_ESM.sdf hold-out. For each predicted product, perform a maximal common substructure match between the parent substrate and the product; atoms of the substrate that are altered (bond broken/formed or element changed by the reaction, e.g. via the SMIRKS transformations used by MetaboGen) are labeled as sites of metabolism. For SoM-only tools (SmartCyp, FAME2, FAME3, RS-Predictor, XenoSite, RD-Metabolizer) use the atom annotations directly — no structure conversion is needed. Convert product-level predictions into binary atom labels per substrate per CYP450 isoform, then compute the Jaccard score TP/(TP+FN+FP) per tool–enzyme pair across the nine isoforms (CYP1A2, CYP2A6, CYP2B6, CYP2C8, CYP2C9, CYP2C19, CYP2D6, CYP2E1, CYP3A4) so that both tool families occupy the same scoring space. Note CyProduct's score formula (Score = (prob−threshold)/(1−threshold), threshold = 1/(1+beta)) if per-atom probabilities must be thresholded before labeling.

## Related tools

- **CyProduct** (Metabolite-predicting tool whose byproduct outputs are converted to SoM annotations for Jaccard comparison; run via CyProduct.jar over the nine CYP450 enzymes) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **CypBoM** (CyProduct's bond-of-metabolism module whose reactive <X,Y> bond predictions directly yield substrate atom SoM labels) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **MetaboGen** (Generates metabolites from predicted BoM<X−Y>s using SMIRKS transformation rules; the applied SMIRKS edits define which substrate atoms count as sites of metabolism) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **FAME2** (SoM-only comparator; its atom-level predictions are scored directly without product-to-SoM conversion (CypBoM is 25% better on SoMs))
- **FAME3** (SoM-only comparator; scored directly at atom level (CypBoM is 36% better on SoMs))

## Examples

```
java -jar CyProduct.jar CC(=O)Nc1ccc(O)cc1 1A2,2A6,2B6,2C8,2C9,2C19,2D6,2E1,3A4 E:/Users/cyproduct/
```

## Evaluation signals

- Every Jaccard cell must lie in [0, 1] and the matrix must contain all 9 isoform columns and all 12 tools
- The average SoM improvement of CyProduct over the best-performing competitor should approximate the paper's reported ~30% value
- Sub-matrix checks: CypBoM's Jaccard should come out 25% better than FAME2's and 36% better than FAME3's (Table 3 left/right)
- Back-mapped SoM labels must be atoms of the parent substrate (valid atom indices), and any substrate atom labeled as a site must appear in at least one predicted product as modified

## Limitations

- Converting products back to SoMs depends on substrate–product alignment; metabolites modified at multiple positions or through rare rules (e.g. the 4 of 828 Cyclization η+η reactions in EBoMD) may map ambiguously
- Reported results are produced from different combinations/subsets of the PredictedResults folders (60Reactants, 30NonReactants, 59 BioTransformer molecules), so the exact benchmark composition must be checked per comparison
- Per-atom scores depend on the cost-sensitive threshold threshold = 1/(1+beta); choosing beta changes which atoms exceed the SoM cutoff

## Evidence

- [methods] MetaboGen applies SMIRKS transformations to generate metabolites: [evidence span withheld — anchor status: not_found]
