---
name: smiles-substrate-batch-prediction
description: Use when you need metabolite/byproduct predictions for one or more query
  compounds against specific human CYP450 isoforms, e.g. replicating benchmark runs
  on 60Reactants.sdf or 30NonReactants.sdf.
when_to_use_negative:
- The query compounds are not plausible human CYP450 substrates — the CypReact filter
  will gate out non-reactants, so expect empty/near-zero metabolite output (as seen
  on 30NonReactants.sdf)
- You need metabolism prediction for enzymes outside the nine major human isoforms
  (1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4) — the tool and its trained CypBoM
  models (EBoMD/EBoMD2) cover only these
- Enzyme names are given with the 'CYP' prefix — the README explicitly requires '1A2'
  not 'CYP1A2', so input must be reformatted first
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3370
- http://edamontology.org/topic_0209
tools:
- name: CyProduct
  role: The three-module predictor (CypReact filter + CypBoM SoM/bond predictor +
    MetaboGen metabolite generator) invoked via CyProduct.jar CLI
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: CypReact
  role: 'Machine-learned filter that gates the pipeline: proceeds only if the compound
    is classified as a reactant (R) for the given CYP450 enzyme'
- name: CypBoM
  role: Machine-learned module predicting reactive η-η bonds (BoMX-Y pairs) with scores
    for a given isoform and molecule
- name: MetaboGen
  role: Generates metabolites from predicted BoMX-Ys using SMIRKS transformation rules
    from Reaction_rules.csv (Dealkylation, EpOxidation, Reduction, Deamination, RingRearrangement)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:765698a8dfa6db9fc3aaf62991f599c6ea1079e863bef19dc79a5e2c3796d79c
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
  - http://edamontology.org/topic_3370
  - http://edamontology.org/topic_0209
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6842105263157895
n_claims: 19
---

# smiles-substrate-batch-prediction

## Summary

Run the CyProduct.jar command-line tool on a SMILES string or SDF file to batch-predict CYP450 metabolites across any subset of the nine major human isoforms (1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4), writing results to a designated output folder.

## When to use

Use when you need metabolite/byproduct predictions for one or more query compounds against specific human CYP450 isoforms, e.g. replicating benchmark runs on 60Reactants.sdf or 30NonReactants.sdf.

## When NOT to use

- The query compounds are not plausible human CYP450 substrates — the CypReact filter will gate out non-reactants, so expect empty/near-zero metabolite output (as seen on 30NonReactants.sdf)
- You need metabolism prediction for enzymes outside the nine major human isoforms (1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4) — the tool and its trained CypBoM models (EBoMD/EBoMD2) cover only these
- Enzyme names are given with the 'CYP' prefix — the README explicitly requires '1A2' not 'CYP1A2', so input must be reformatted first

## Inputs

- Query molecule as SMILES string (format: SMILES=yourSmilesString) or path to an .sdf file (e.g. 60Reactants.sdf, 30NonReactants.sdf)
- Comma-separated list of CYP450 enzyme names without 'CYP' prefix (e.g. 1A2,2A6,2B6,2C8,2C9,2C19,2D6,2E1,3A4)
- Output folder path for predicted metabolites

## Outputs

- Predicted metabolites for each query compound/enzyme combination written to the specified output folder (e.g. PredictedResults)
- Per-Site-of-Metabolism scores computed as Score = (prob-threshold)/(1-threshold)

## How to apply

Invoke the jar as 'java -jar CyProduct.jar QueryMolecule enzymeList OutputFolderPath', where QueryMolecule is either a SMILES string prefixed with 'SMILES=' or a path to an .sdf file. Supply the enzyme list as comma-separated isoform names WITHOUT the 'CYP' prefix (e.g. '1A2,3A4'); predictions cover metabolites catalyzed by any enzyme in the list (union). CyProduct internally applies the CypReact reactant filter before CypBoM predicts reactive bonds (BoMX-Y pairs) and MetaboGen applies SMIRKS transformation rules from Reaction_rules.csv to generate metabolites, scoring each site with Score = (prob-threshold)/(1-threshold) where threshold = 1/(1+beta). For batch benchmarking, run on 60Reactants.sdf and on 30NonReactants.sdf to confirm near-zero predictions for non-reactants, then compare against reference metabolite lists or published ADMET Predictor, BioTransformer and GLORY results using the Jaccard score TP/(TP+FN+FP).

## Related tools

- **CyProduct** (The three-module predictor (CypReact filter + CypBoM SoM/bond predictor + MetaboGen metabolite generator) invoked via CyProduct.jar CLI) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **CypReact** (Machine-learned filter that gates the pipeline: proceeds only if the compound is classified as a reactant (R) for the given CYP450 enzyme)
- **CypBoM** (Machine-learned module predicting reactive η-η bonds (BoMX-Y pairs) with scores for a given isoform and molecule)
- **MetaboGen** (Generates metabolites from predicted BoMX-Ys using SMIRKS transformation rules from Reaction_rules.csv (Dealkylation, EpOxidation, Reduction, Deamination, RingRearrangement))

## Examples

```
java -jar CyProduct.jar CC(=O)Nc1ccc(O)cc1 1A2,2A6,2B6,2C8,2C9,2C19,2D6,2E1,3A4 E:\Users\cyproduct\
```

## Evaluation signals

- Output folder contains predicted metabolite files for every requested enzyme in the enzymeList
- On the 30NonReactants.sdf control set, predicted metabolite count is zero or near-zero
- On HDFAME's 60Reactants.sdf, Jaccard score TP/(TP+FN+FP) computed against the 90 true metabolites (union of CYP2C9/2D6/3A4) can be reproduced and matches the paper's ~200% average improvement over ADMET Predictor, BioTransformer and GLORY
- Computed Jaccard scores for CypBoM exceed FAME2 by ~25% and FAME3 by ~36% in SoM terms (Table 3)
- GLORY reference counts on HDFAME are 871 metabolites (MaxEfficiency) and 2376 (MaxCoverage), consistent with the paper

## Limitations

- Enzyme input must omit the 'CYP' prefix (use '1A2', not 'CYP1A2') or the tool may not interpret the request
- If the CypReact filter misclassifies a true substrate as non-reactant, no metabolites will be generated for that compound/isoform; the BioTransformerAPI exposes useCypReact=false to disable this filter
- No software version numbers are stated for CDK, WEKA, or the external comparison tools; users must contact <maintainer e-mail, see the repository README> regarding the repository
- Scoring depends on the cost-matrix parameter beta (threshold = 1/(1+beta)), so score interpretation requires knowing the trained model's beta

## Evidence

- [methods] MetaboGen SMIRKS generation: [evidence span withheld — anchor status: not_found]
