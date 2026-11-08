---
name: multiomic-feature-metabolite-overlap-quantification
description: 'Use when validating ML metabolite models: given top predictor features
  per metabolite (e.g. from Metab8D_network.xlsx) and the Recon3D model, to compute
  the fraction of predictors mapping to the metabolite''s reactions or one-step neighbors.'
when_to_use_negative:
- The goal is inferring new feature-metabolite regulatory relationships rather than
  validating topological proximity of existing top predictors — Recon8D notes its
  'guilt by association' network associations are descriptive and not necessarily
  mechanistic
- 'The model or network is not Recon3D-compatible: features must map to genes/reactions
  in Recon3D (input datasets were ''filtered for genes present in the Recon3D genome-scale
  metabolic model'')'
- Predictors are non-gene features (e.g. metabolites or reaction fluxes) that cannot
  be translated to BiggIDs via the gene-name-to-BiggID mapping
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: COBRA toolbox
  role: Identifies all Recon3D reactions associated with each relevant metabolite
    for the overlap computation
- name: Python
  role: Runs the recon_mapping scripts that extract genes from reactions involving
    metabolites of interest and match them with top feature lists
  repo: https://github.com/sriram-lab/Metab8D
- name: Recon3D genome-scale metabolic model
  role: Reference network whose reaction adjacency defines direct and first-degree-neighbor
    sets per metabolite
- name: recon_mapping
  role: MATLAB and Python scripts plus a common-gene-name-to-BiggID csv used to map
    top features to Recon3D reactions
  repo: https://github.com/sriram-lab/Metab8D
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:088fe0332a061b8a437c396f8098959f5bce7ca57fe0abe76aee94534969fdcf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - COBRA toolbox
  - Python
  - Recon3D genome-scale metabolic model
  - recon_mapping
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# multiomic-feature-metabolite-overlap-quantification

## Summary

Quantifies how top ML-identified predictor features for a metabolite overlap with that metabolite's associated reactions (or their first-degree neighbors) in the Recon3D genome-scale metabolic model, testing whether predictors are topologically proximal to the metabolite. Grounded in Recon8D's COBRA-based reaction mapping with cofactor filtering.

## When to use

Use when validating ML metabolite models: given top predictor features per metabolite (e.g. from Metab8D_network.xlsx) and the Recon3D model, to compute the fraction of predictors mapping to the metabolite's reactions or one-step neighbors.

## When NOT to use

- The goal is inferring new feature-metabolite regulatory relationships rather than validating topological proximity of existing top predictors — Recon8D notes its 'guilt by association' network associations are descriptive and not necessarily mechanistic
- The model or network is not Recon3D-compatible: features must map to genes/reactions in Recon3D (input datasets were 'filtered for genes present in the Recon3D genome-scale metabolic model')
- Predictors are non-gene features (e.g. metabolites or reaction fluxes) that cannot be translated to BiggIDs via the gene-name-to-BiggID mapping

## Inputs

- Top predictor feature lists per metabolite per omic layer (e.g. Metab8D_network.xlsx: top 20 features for 225 RF metabolite models from nine feature sets, with 0-8 confidence scores)
- Recon3D genome-scale metabolic model (reactions and metabolite-reaction associations, accessed via COBRA toolbox)
- recon_mapping scripts and csv of common gene name to BiggID translations
- human_1_mapping.csv (Recon3D metabolites mapped to Human1 IDs)

## Outputs

- overlap_statistics.csv: per-metabolite fraction of top features overlapping direct reactions and first-degree reaction neighbors
- Stratified overlap contrasts (central vs peripheral metabolites; signaling-enzyme vs other predictors)
- Aggregate one-step overlap range across omic layers (Recon8D: 92-96%)

## How to apply

Load the top predictor features per metabolite from each omic layer (CNV, DNA methylation, transcriptomics, proteomics; the Metab8D_network.xlsx top-20 lists are suitable inputs). Using COBRA toolbox functions, identify all Recon3D reactions associated with each relevant metabolite, and map each top feature to its corresponding Recon3D reaction via the recon_mapping MATLAB/Python scripts, translating common gene names to BiggIDs with the provided csv. Remove common metabolic cofactors (ATP, ADP, water, hydrogen, phosphate, diphosphate) so neighboring reactions are not overrepresented. Compute the fraction of mapped features overlapping the metabolite's direct reactions and its first-degree reaction neighbors, then stratify by metabolite position (central vs peripheral) and predictor type (signaling enzymes vs other features) to obtain contrasts such as the 50% vs 85% and 31% vs 12.5% splits. In Recon8D, 92-96% of top-5 features per metabolite fell within one step of the metabolite's reactions across the four omic layers.

## Related tools

- **COBRA toolbox** (Identifies all Recon3D reactions associated with each relevant metabolite for the overlap computation)
- **Python** (Runs the recon_mapping scripts that extract genes from reactions involving metabolites of interest and match them with top feature lists) — https://github.com/sriram-lab/Metab8D
- **Recon3D genome-scale metabolic model** (Reference network whose reaction adjacency defines direct and first-degree-neighbor sets per metabolite)
- **recon_mapping** (MATLAB and Python scripts plus a common-gene-name-to-BiggID csv used to map top features to Recon3D reactions) — https://github.com/sriram-lab/Metab8D

## Evaluation signals

- Per-metabolite overlap fractions fall in the 92-96% one-step range across the CNV, DNA methylation, transcriptomics, and proteomics layers
- The cofactor removal list exactly matches ATP, ADP, water, hydrogen, phosphate, and diphosphate before computing neighbor fractions
- Stratified contrasts reproduce the expected pattern: central metabolites ~50% vs peripheral ~85% direct overlap; signaling-enzyme predictors ~31% vs other features ~12.5%
- Every mapped feature resolves to a valid Recon3D BiggID via the provided translation csv (no unmapped gene names left silently dropped)
- Output file overlap_statistics.csv contains both direct-reaction and first-degree-neighbor fraction columns per metabolite

## Limitations

- Cofactor removal choices (e.g. excluding ATP, ADP, water, hydrogen, phosphate, diphosphate) directly inflate or deflate one-step overlap fractions, so results depend on this preprocessing step
- Network associations are descriptive 'guilt by association' and do not establish causative mechanisms
- Applicability is restricted to features representable in Recon3D; results are validated in cancer cell lines (CCLE, NCI60) and may not reflect in vivo metabolism

## Evidence
