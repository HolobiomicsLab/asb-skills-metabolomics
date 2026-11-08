---
name: cyp450-metabolite-jaccard-comparison
description: 'Use when comparing metabolite-prediction tools on human CYP450 enzymes:
  you have an SDF of known substrates with true metabolite lists, a set of non-reactants,
  and baseline tool outputs to score with Jaccard.'
when_to_use_negative:
- Your input compounds are not human CYP450 substrates/non-reactants (e.g. microbiome
  or phase-II metabolism), since the benchmark, datasets (60Reactants.sdf, 30NonReactants.sdf)
  and baselines are all CYP450-specific
- You need site-of-metabolism (SoM) ranking rather than whole-metabolite set comparison
  — the Jaccard score here is computed over predicted metabolite sets, whereas SoM
  performance was evaluated separately (CypBoM vs FAME2/FAME3)
- You only need to run CyProduct to generate predictions for a new query molecule;
  this skill is for benchmarking against reference metabolite lists, not for routine
  prediction
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: CyProduct
  role: Tool under evaluation; run via CLI on 60Reactants.sdf and 30NonReactants.sdf
    for the nine CYP450 isoforms to produce predicted metabolites scored with the
    Jaccard metric
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: CypReact
  role: 'Filter step inside CyProduct: prediction proceeds only if the compound is
    classified as a reactant (R) for the given enzyme'
- name: GLORY
  role: Baseline predictor evaluated in MaxEfficiency and MaxCoverage modes (871 and
    2376 metabolites predicted vs 90 true metabolites)
- name: BioTransformer
  role: Baseline knowledge-based/ML predictor whose hold-out dataset (13321_2018_324_MOESM6_ESM.sdf)
    and published results are used for comparison
- name: ADMET Predictor
  role: Baseline commercial predictor whose published results are compared against
    CyProduct's Jaccard scores
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:c717bc61f453e583712c2bba7759d9045b3b6682ad77ec49ec285337acb20d61
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# cyp450-metabolite-jaccard-comparison

## Summary

A benchmarking procedure that evaluates CYP450 metabolite predictors (CyProduct vs ADMET Predictor, BioTransformer, GLORY) by computing the Jaccard score (TP/(TP+FN+FP)) on a curated test set of 60 known CYP450 substrates (subset of EBoMD2) plus 30 non-reactants, aggregated over the union of CYP2C9, 2D6 and 3A4.

## When to use

Use when comparing metabolite-prediction tools on human CYP450 enzymes: you have an SDF of known substrates with true metabolite lists, a set of non-reactants, and baseline tool outputs to score with Jaccard.

## When NOT to use

- Your input compounds are not human CYP450 substrates/non-reactants (e.g. microbiome or phase-II metabolism), since the benchmark, datasets (60Reactants.sdf, 30NonReactants.sdf) and baselines are all CYP450-specific
- You need site-of-metabolism (SoM) ranking rather than whole-metabolite set comparison — the Jaccard score here is computed over predicted metabolite sets, whereas SoM performance was evaluated separately (CypBoM vs FAME2/FAME3)
- You only need to run CyProduct to generate predictions for a new query molecule; this skill is for benchmarking against reference metabolite lists, not for routine prediction

## Inputs

- 60Reactants.sdf (subset of the EBoMD2 hold-out set, from the HDFAME folder of the Bitbucket repository)
- 30NonReactants.sdf (non-CYP450 compounds used as a negative-control set)
- Reference/true metabolite lists for the 90 true metabolites in the reactant subset
- Published baseline prediction outputs from ADMET Predictor, BioTransformer, and GLORY (MaxEfficiency and MaxCoverage modes)

## Outputs

- Per-enzyme Jaccard scores (TP/(TP+FN+FP)) for each evaluated tool on the union of CYP2C9, 2D6 and 3A4 predictions
- Average percentage improvement of CyProduct's Jaccard score relative to each baseline tool
- Predicted metabolite lists stored in a PredictedResults output folder (e.g. CyProductResult_ForCYP2C9_2D6_3A4)

## How to apply

Run CyProduct.jar via its CLI (java -jar CyProduct.jar QueryMolecule enzymeList OutputFolderPath) on the 60Reactants.sdf from the HDFAME folder for each of the nine CYP450 isoforms (1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4), writing to a PredictedResults output folder; enzyme identifiers must be given without the 'CYP' prefix. Repeat the run on 30NonReactants.sdf to confirm near-zero predictions for non-CYP450 compounds. Collect reference metabolite lists and published outputs of ADMET Predictor, BioTransformer, and GLORY (both MaxEfficiency and MaxCoverage modes, which respectively predicted 871 and 2376 metabolites against 90 true metabolites). For each tool, compute the Jaccard score TP/(TP+FN+FP) on the union of CYP2C9, 2D6 and 3A4 substrate predictions against the true metabolites, then compute the average percentage improvement of CyProduct over each baseline. Cross-check that the computed average improvement matches the paper's reported ~200% average improvement over existing tools.

## Related tools

- **CyProduct** (Tool under evaluation; run via CLI on 60Reactants.sdf and 30NonReactants.sdf for the nine CYP450 isoforms to produce predicted metabolites scored with the Jaccard metric) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **CypReact** (Filter step inside CyProduct: prediction proceeds only if the compound is classified as a reactant (R) for the given enzyme)
- **GLORY** (Baseline predictor evaluated in MaxEfficiency and MaxCoverage modes (871 and 2376 metabolites predicted vs 90 true metabolites))
- **BioTransformer** (Baseline knowledge-based/ML predictor whose hold-out dataset (13321_2018_324_MOESM6_ESM.sdf) and published results are used for comparison)
- **ADMET Predictor** (Baseline commercial predictor whose published results are compared against CyProduct's Jaccard scores)

## Examples

```
java -jar CyProduct.jar Datasets/HDFAME/60Reactants.sdf 1A2,2A6,2B6,2C8,2C9,2C19,2D6,2E1,3A4 PredictedResults/
```

## Evaluation signals

- Jaccard score must be computed as TP / (TP+FN+FP) on the union of CYP2C9, 2D6 and 3A4 predictions against the 90 true metabolites in 60Reactants.sdf
- GLORY reference counts must match the paper: 871 metabolites in MaxEfficiency mode and 2376 in MaxCoverage mode against 90 true metabolites
- Computed average improvement of CyProduct over baselines should reproduce the paper's reported ~200% average improvement in Jaccard score
- Running CyProduct on 30NonReactants.sdf must yield no (or near-zero) predicted metabolites for the non-CYP450 compounds
- Enzyme inputs must be passed without the 'CYP' prefix (e.g. 1A2, not CYP1A2), per the README

## Limitations

- The headline Jaccard comparison is reported on the union of CYP2C9, 2D6 and 3A4 predictions rather than all nine isoforms individually
- GLORY's MaxCoverage mode predicted 2376 metabolites against only 90 true metabolites, so baseline FP counts (and thus Jaccard gaps) can be dominated by over-prediction
- No software version numbers are stated for CDK, WEKA, or the external comparison tools, and the exact 'repository' URL for datasets is not explicit in the methods text (only the Bitbucket link in the abstract)
- CyProduct's internal score is a transformed probability, Score = (prob-threshold)/(1-threshold) with threshold = 1/(1+beta), so raw probabilities are not directly comparable across tools

## Evidence
