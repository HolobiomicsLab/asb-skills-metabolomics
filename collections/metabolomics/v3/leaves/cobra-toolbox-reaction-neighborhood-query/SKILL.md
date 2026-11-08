---
name: cobra-toolbox-reaction-neighborhood-query
description: Use when you need to verify whether predictor features (genes/enzymes)
  from ML models are topologically proximal to a metabolite's reactions in a genome-scale
  metabolic model such as Recon3D.
when_to_use_negative:
- When a direct mechanistic gene-metabolite relationship already exists in curated
  databases — this skill tests topological proximity in Recon3D, not causality (the
  paper notes its 'guilt by association' strategy may not identify causative mechanisms)
- When features fall outside Recon3D — the paper filters input datasets for genes
  present in the Recon3D model, so features absent from Recon3D cannot be mapped
- When assessing models trained on in vivo or non-CCLE data — the authors caution
  models are not directly transferrable due to differences between in vitro and in
  vivo conditions
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0203
tools:
- name: COBRA toolbox
  role: Provides functions to identify all Recon3D reactions associated with relevant
    metabolites and to support reaction-neighborhood extraction
- name: Python
  role: Runs the recon_mapping scripts that extract genes from reactions involving
    metabolites of interest and match them with top feature lists
  repo: https://github.com/sriram-lab/Metab8D
- name: Recon3D genome-scale metabolic model
  role: Reference metabolic network whose reactions and first-degree neighbors define
    the topological target for overlap computation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:883ca8f16ebc3812e03ccbdf6e26c3114f3583cbbef2de7f2ba7e28cd35251a0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - COBRA toolbox
  - Python
  - Recon3D genome-scale metabolic model
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# cobra-toolbox-reaction-neighborhood-query

## Summary

Uses COBRA toolbox functions on the Recon3D genome-scale metabolic model to extract all reactions associated with a metabolite of interest and to map ML-identified top predictor features onto those reactions, enabling quantitative topological-overlap checks between predictors and metabolites.

## When to use

Use when you need to verify whether predictor features (genes/enzymes) from ML models are topologically proximal to a metabolite's reactions in a genome-scale metabolic model such as Recon3D.

## When NOT to use

- When a direct mechanistic gene-metabolite relationship already exists in curated databases — this skill tests topological proximity in Recon3D, not causality (the paper notes its 'guilt by association' strategy may not identify causative mechanisms)
- When features fall outside Recon3D — the paper filters input datasets for genes present in the Recon3D model, so features absent from Recon3D cannot be mapped
- When assessing models trained on in vivo or non-CCLE data — the authors caution models are not directly transferrable due to differences between in vitro and in vivo conditions

## Inputs

- Top predictor feature lists per metabolite from random-forest metabolomic models (e.g., Metab8D_network.xlsx top-20 features with confidence scores)
- Recon3D genome-scale metabolic model
- recon_mapping scripts (MATLAB/Python) and common gene name to BiggID translation csv (recon_mapping csv, human_1_mapping.csv)

## Outputs

- Per-metabolite set of associated Recon3D reactions and their first-degree reaction neighbors
- Mapped feature-to-reaction assignments after cofactor removal
- Overlap statistics (fraction of top features within one step of the metabolite's reactions; e.g., overlap_statistics.csv)

## How to apply

Load Recon3D and use COBRA toolbox functions to identify all reactions associated with each relevant metabolite (e.g., querying reactions containing the metabolite's BiggID via the recon_mapping scripts and the common-gene-name-to-BiggID translations in the repository). Map each top ML feature (from CNV, DNA methylation, transcriptomics, or proteomics models) to its corresponding Recon3D reaction. Remove common metabolic cofactors (ATP, ADP, water, hydrogen, phosphate, diphosphate) so that neighboring reactions are not overrepresented. Then compute the fraction of mapped features that overlap the metabolite's directly associated reactions or their first-degree reaction neighbors; the paper reports 92-96% overlap within one step for the top 5 features per metabolite across the four omic layers.

## Related tools

- **COBRA toolbox** (Provides functions to identify all Recon3D reactions associated with relevant metabolites and to support reaction-neighborhood extraction)
- **Python** (Runs the recon_mapping scripts that extract genes from reactions involving metabolites of interest and match them with top feature lists) — https://github.com/sriram-lab/Metab8D
- **Recon3D genome-scale metabolic model** (Reference metabolic network whose reactions and first-degree neighbors define the topological target for overlap computation)

## Examples

```
python recon_mapping/map_features.py --metabolite <BiggID> --features Metab8D_network.xlsx --model Recon3D
```

## Evaluation signals

- Cofactor-removal check: confirm ATP, ADP, water, hydrogen, phosphate, and diphosphate are excluded before computing neighbor sets, as their inclusion overrepresents neighboring reactions
- Overlap invariance: top 5 features per metabolite should show 92-96% overlap with the metabolite's directly associated reactions or first-degree neighbors across the four omic layers
- Stratification consistency: overlap should reproduce contrasts such as central vs peripheral metabolites and signaling-enzyme vs other predictor types (e.g., ~50% vs ~85%, ~31% vs ~12.5% in the source card)
- Mapping completeness: every feature mapped must resolve to a valid Recon3D BiggID via the common-gene-name-to-BiggID translation csv

## Limitations

- Network associations from systems-level data are descriptive and not necessarily mechanistic; the 'guilt by association' strategy may not identify causative mechanisms
- The analysis was done in cancer cell lines in culture and may not reflect metabolism in vivo
- Features must be genes present in Recon3D; input datasets were filtered for Recon3D genes, so unmappable features are excluded

## Evidence
