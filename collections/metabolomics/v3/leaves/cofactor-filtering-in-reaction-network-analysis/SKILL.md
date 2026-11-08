---
name: cofactor-filtering-in-reaction-network-analysis
description: Use when mapping top predictor features (e.g., genes from CNV, methylation,
  transcriptomic, or proteomic ML models) onto a genome-scale metabolic reaction network
  to compute topological overlap with a target metabolite's reactions.
when_to_use_negative:
- When the goal is flux balance or reaction activity analysis rather than topological
  proximity of predictor features to a metabolite's reactions
- When features are already gene-set members of the metabolite's own reactions and
  no first-degree-neighbor inference is intended (cofactor removal mainly guards neighbor-level
  overlap)
tools:
- name: COBRA toolbox
  role: Used to identify all Recon3D reactions associated with each relevant metabolite
    prior to cofactor-filtered overlap computation
  repo: sriram-lab/Metab8D
- name: Python
  role: Runs the recon_mapping scripts that extract genes from metabolite-associated
    reactions and match them against top feature lists after cofactor removal
  repo: sriram-lab/Metab8D
- name: Recon3D genome-scale metabolic model
  role: Provides the reaction network in which metabolite-proximal features are identified
    and cofactor-mediated spurious neighbors are pruned
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:34281281108a64861196d4b9eda58d6c5ecb78907f21ea2e573b504a99025c49
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - COBRA toolbox
  - Python
  - Recon3D genome-scale metabolic model
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Cofactor filtering in reaction network analysis

## Summary

Removes ubiquitous metabolic cofactors (ATP, ADP, water, hydrogen, phosphate, diphosphate) before computing network proximity between ML predictor features and metabolites in Recon3D, so that first-degree-neighbor overlap is not inflated by reactions connected only through common cofactors.

## When to use

Use when mapping top predictor features (e.g., genes from CNV, methylation, transcriptomic, or proteomic ML models) onto a genome-scale metabolic reaction network to compute topological overlap with a target metabolite's reactions.

## When NOT to use

- When the goal is flux balance or reaction activity analysis rather than topological proximity of predictor features to a metabolite's reactions
- When features are already gene-set members of the metabolite's own reactions and no first-degree-neighbor inference is intended (cofactor removal mainly guards neighbor-level overlap)

## Inputs

- Top predictor feature lists per metabolite from the metabolic-gene-only ML models (e.g., Metab8D_network.xlsx top-20 features per model)
- Recon3D genome-scale metabolic model with metabolite-reaction associations
- Common gene name to BiggID translation table (recon_mapping csv)

## Outputs

- Per-metabolite overlap fractions of mapped features with the metabolite's direct reactions and first-degree reaction neighbors (overlap_statistics.csv)
- Overlap stratified by central vs peripheral metabolite position and signaling-enzyme vs other predictor type

## How to apply

First, use COBRA toolbox functions to identify all Recon3D reactions associated with each target metabolite, and map each top predictor feature to its corresponding Recon3D reaction via the recon_mapping scripts (which include common gene name to BiggID translations). Before computing overlap, remove the common metabolic cofactors — ATP, ADP, water, hydrogen, phosphate, and diphosphate — from the network neighborhood; the rationale is that cofactors participate in so many reactions that leaving them in overrepresents neighboring reactions and inflates apparent first-degree proximity. Then compute the fraction of mapped features overlapping the metabolite's associated reactions or their first-degree neighbors. Finally, stratify overlap by metabolite position (central vs peripheral metabolism) and predictor type (signaling enzymes vs other features) to detect differential proximity, as in the 50% vs 85% and 31% vs 12.5% contrasts reported in the source study.

## Related tools

- **COBRA toolbox** (Used to identify all Recon3D reactions associated with each relevant metabolite prior to cofactor-filtered overlap computation) — sriram-lab/Metab8D
- **Python** (Runs the recon_mapping scripts that extract genes from metabolite-associated reactions and match them against top feature lists after cofactor removal) — sriram-lab/Metab8D
- **Recon3D genome-scale metabolic model** (Provides the reaction network in which metabolite-proximal features are identified and cofactor-mediated spurious neighbors are pruned)

## Evaluation signals

- The cofactor exclusion list must contain exactly ATP, ADP, water, hydrogen, phosphate, and diphosphate as defined in the study
- After filtering, overall overlap of top-5 features with the metabolite's direct reactions or first-degree neighbors should fall in the 92-96% range across the four omic layers (CNV, DNA methylation, transcriptomics, proteomics)
- Central metabolites should show higher first-degree-neighbor overlap than peripheral ones (85% vs 50% contrast) and signaling-enzyme predictors higher than other features (31% vs 12.5%)
- Every mapped feature must carry a valid BiggID via the recon_mapping gene-name translation table before overlap is scored

## Limitations

- Feature-gene membership is inferred by guilt-by-association network logic; reactions matched to features are associations, not necessarily mechanistic or causative links
- Recon3D reaction annotations are incomplete, so some metabolite-proximal relationships may be missed or spurious
- The analysis was done in cancer cell lines in culture and may not reflect metabolism in vivo, limiting transferability of mapped networks to other data sources

## Evidence
