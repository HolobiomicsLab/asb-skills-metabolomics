---
name: sdf-molecular-structure-loading
description: Use when the input compounds are provided as .sdf files (e.g. 60Reactants.sdf,
  30NonReactants.sdf, EBoMD2.sdf) and must be read as molecular structures before
  running metabolite predictors or computing precision/recall/Jaccard metrics.
when_to_use_negative:
- The input is already a parsed molecule object, SMILES string, or feature table rather
  than an SDF file — no SDF loading step is needed.
- The task requires reaction rules or SMIRKS transformations rather than substrate
  structures — use Reaction_rules.csv instead of the SDF datasets.
- The compounds come from a non-structure format (e.g. plain text identifiers without
  connection tables); SDF loading cannot recover bond/atom information from IDs alone.
edam_operation: http://edamontology.org/operation_2406
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3332
tools:
- name: CyProduct
  role: Accepts an SDF file path (or SMILES string) directly as the QueryMolecule
    input to run CYP450 metabolite prediction on the loaded structures
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:448df8e93efc48650915401a15a27109360dc43a58648fa6165e3e4fd2b2808b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ADMET Predictor
  - BioTransformer
  - CyProduct
  - GLORY
  edam_operation: http://edamontology.org/operation_2406
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# sdf-molecular-structure-loading

## Summary

Load evaluation or query compounds from Structure-Data File (SDF) resources (e.g. 60Reactants.sdf, EBoMD2.sdf, 13321_2018_324_MOESM6_ESM.sdf) as the substrate input for CYP450 metabolism prediction and benchmarking workflows.

## When to use

Use when the input compounds are provided as .sdf files (e.g. 60Reactants.sdf, 30NonReactants.sdf, EBoMD2.sdf) and must be read as molecular structures before running metabolite predictors or computing precision/recall/Jaccard metrics.

## When NOT to use

- The input is already a parsed molecule object, SMILES string, or feature table rather than an SDF file — no SDF loading step is needed.
- The task requires reaction rules or SMIRKS transformations rather than substrate structures — use Reaction_rules.csv instead of the SDF datasets.
- The compounds come from a non-structure format (e.g. plain text identifiers without connection tables); SDF loading cannot recover bond/atom information from IDs alone.

## Inputs

- SDF file with molecular structure records (e.g. Datasets/60Reactants.sdf, Datasets/30NonReactants.sdf, Datasets/EBoMD2.sdf, Datasets/13321_2018_324_MOESM6_ESM.sdf)
- SMILES string (alternative query format accepted by CyProduct.jar via SMILES=<string>)

## Outputs

- In-memory list of molecular structures (one per SDF record) ready for metabolite prediction tools
- Per-compound identifiers/fields usable to match predicted metabolites against curated true metabolites for precision/recall/Jaccard computation

## How to apply

Parse the SDF file to extract one molecule record per compound, retaining any per-record fields needed to join against curated true metabolites (e.g. the EBoMD2 hold-out labels). Use the loaded structures either as direct tool input — CyProduct.jar accepts an SDF path as its QueryMolecule argument — or convert each record to SMILES for tools that require string input. For the CyProduct benchmark workflow, load 60Reactants.sdf (the 60-compound subset of the 98-compound EBoMD2 hold-out set) rather than the full EBoMD2.sdf when replicating the FAME2/GLORY comparison on HDFame. Verify each record loads as a valid structure (non-empty connection table) before passing it downstream, since tool metrics are computed only over successfully predicted structures (e.g. 59 of the BioTransformer hold-out compounds produced CyProduct predictions).

## Related tools

- **CyProduct** (Accepts an SDF file path (or SMILES string) directly as the QueryMolecule input to run CYP450 metabolite prediction on the loaded structures) — https://bitbucket.org/wishartlab/cyproduct/src/master/

## Examples

```
java -jar CyProduct.jar Datasets/60Reactants.sdf 1A2,2A6,2B6,2C8,2C9,2C19,2D6,2E1,3A4 ./output/
```

## Evaluation signals

- The number of parsed molecule records matches the expected dataset size (e.g. 60 compounds from 60Reactants.sdf, 98 from EBoMD2.sdf, 679-derived compounds from EBoMD.sdf).
- Downstream prediction runs succeed for essentially all loaded records (e.g. CyProduct produced predictions for 59 of the BioTransformer hold-out molecules); unparseable records are flagged.
- When the loaded set is 60Reactants.sdf, the downstream benchmark reproduces the baseline counts (GLORY MaxEfficiency 871 and MaxCoverage 2376 predicted metabolites vs 90 true metabolites) and metric ranges (precision 0.08–0.46, recall 0.08–84→0.08–0.84, Jaccard 0.04–0.38).
- Structures carry valid connection tables (atoms/bonds present), enabling later matching of predicted metabolites against EBoMD2 true metabolites via standard structure comparison.

## Limitations

- When the query is a SMILES string it must use the explicit format SMILES=<yourSmilesString>; a bare SMILES passed as the QueryMolecule argument is not accepted.
- Missing software version numbers for CDK and related tooling mean exact SDF parsing behaviour (e.g. handling of unusual valences or query features) is not pinned by the article.
- The results reported in the manuscript use different combinations/subsets of the PredictedResults folders, so a given SDF load alone does not map 1:1 to a single published table.

## Evidence
