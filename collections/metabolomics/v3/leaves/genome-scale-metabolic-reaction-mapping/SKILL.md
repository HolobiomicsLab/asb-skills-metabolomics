---
name: genome-scale-metabolic-reaction-mapping
description: Use when top ML predictor features (e.g. genes from CNV, methylation,
  transcriptomics, proteomics models) must be anchored to specific reactions in a
  genome-scale metabolic network like Recon3D to test whether predictors sit near
  the metabolite they predict.
when_to_use_negative:
- The input features are signaling or non-metabolic molecules that cannot be translated
  to Recon3D/Human1 BiggIDs via the gene-name mapping
- The goal is to transfer a trained ML model to a new dataset for prediction, not
  to interpret which reactions the predictors sit near — the article notes such models
  are not directly transferrable across sources due to in vitro/in vivo differences
  and limited feature overlap
- The metabolite panel or model covers only a fraction of cellular metabolites (e.g.
  the 225-metabolite targeted panel) and the user needs coverage across all compartments
  rather than topological placement of top predictors
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0602
tools:
- name: COBRA toolbox
  role: Identifies all reactions associated with each relevant metabolite in Recon3D
- name: recon_mapping (MATLAB and Python scripts)
  role: Extracts genes from reactions involving metabolites of interest and matches
    them with top feature lists
  repo: https://github.com/sriram-lab/Metab8D
- name: Python
  role: Runs the overlap-computation and feature-to-reaction mapping scripts
  repo: https://github.com/sriram-lab/Metab8D
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:476dac1943853a401e190c5ec90e827a85a60a58e8fdb4c77b1d4a3f709c3eef
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - COBRA toolbox
  - Python
  - Recon3D genome-scale metabolic model
  - recon_mapping (MATLAB and Python scripts)
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# genome-scale-metabolic-reaction-mapping

## Summary

Maps machine-learning predictor features to reactions and metabolites of a genome-scale metabolic model (Recon3D/Human1) using COBRA toolbox functions, then computes topological overlap statistics between top features and a metabolite's reactions.

## When to use

Use when top ML predictor features (e.g. genes from CNV, methylation, transcriptomics, proteomics models) must be anchored to specific reactions in a genome-scale metabolic network like Recon3D to test whether predictors sit near the metabolite they predict.

## When NOT to use

- The input features are signaling or non-metabolic molecules that cannot be translated to Recon3D/Human1 BiggIDs via the gene-name mapping
- The goal is to transfer a trained ML model to a new dataset for prediction, not to interpret which reactions the predictors sit near — the article notes such models are not directly transferrable across sources due to in vitro/in vivo differences and limited feature overlap
- The metabolite panel or model covers only a fraction of cellular metabolites (e.g. the 225-metabolite targeted panel) and the user needs coverage across all compartments rather than topological placement of top predictors

## Inputs

- Top predictor feature list per metabolite (e.g. Metab8D_network.xlsx top-20 features with confidence scores)
- Recon3D genome-scale metabolic model / Human1 model
- Common gene name to BiggID translation csv (recon_mapping)
- List of metabolites from Recon3D with mapped Human1 IDs (human_1_mapping.csv)

## Outputs

- Per-metabolite overlap statistics: fraction of top features mapping to directly associated reactions vs first-degree neighbors (e.g. 92-96% overlap range)
- Stratified overlap contrasts by metabolite position (central vs peripheral) and predictor type

## How to apply

Load the top predictor features from the trained random-forest metabolite models (e.g. the top 20 features per metabolite in Metab8D_network.xlsx), and for each metabolite of interest use COBRA toolbox functions to extract all Recon3D reactions in which that metabolite participates. Translate each predictor feature to its BiggID using the recon_mapping scripts' common-gene-name-to-BiggID csv, then assign every feature to its corresponding Recon3D reaction. Before computing overlap, remove common metabolic cofactors (ATP, ADP, water, hydrogen, phosphate, diphosphate) so that hub reactions do not artificially inflate first-degree-neighbor counts. Compute the fraction of mapped features that fall on the metabolite's directly associated reactions versus their first-degree reaction neighbors, then stratify results by metabolite position in central versus peripheral metabolism and by predictor type (e.g. signaling enzymes like YAP1-related features versus other features).

## Related tools

- **COBRA toolbox** (Identifies all reactions associated with each relevant metabolite in Recon3D)
- **recon_mapping (MATLAB and Python scripts)** (Extracts genes from reactions involving metabolites of interest and matches them with top feature lists) — https://github.com/sriram-lab/Metab8D
- **Python** (Runs the overlap-computation and feature-to-reaction mapping scripts) — https://github.com/sriram-lab/Metab8D

## Examples

```
python recon_mapping/match_features_to_reactions.py --features Metab8D_network.xlsx --mapping recon_mapping/common_gene_to_biggid.csv --cofactors ATP,ADP,H2O,H,Phosphate,Diphosphate --out overlap_statistics.csv
```

## Evaluation signals

- Every predictor feature that is translated to a BiggID resolves to a Recon3D reaction present in the loaded model
- Common cofactors (ATP, ADP, water, hydrogen, phosphate, diphosphate) are excluded from the neighbor-expansion step; removing them should lower first-degree-neighbor counts versus an unfiltered run
- Overall overlap between top features and directly-associated-or-first-degree reactions falls in the article's reported 92-96% range
- Stratified contrasts reproduce the article's central-vs-peripheral and signaling-vs-other splits (e.g. ~50% vs 85% and 31% vs 12.5%)
- Output overlap_statistics.csv is non-empty for every metabolite that had at least one resolvable BiggID

## Limitations

- Feature-to-BiggID translation via common gene names may fail for genes absent from Recon3D, since input datasets were filtered for genes present in the Recon3D genome-scale metabolic model
- The 'guilt by association' mapping is descriptive: network associations driven by systems-level data are not necessarily mechanistic and may not identify causative mechanisms
- Mapping relies on the 225-metabolite targeted panel and 90 targeted phosphoproteins, so coverage across the full cellular array of metabolites and phosphoproteins is incomplete
- Only top-N features (e.g. top 20 per metabolite in Metab8D_network.xlsx) are mapped; lower-ranked features with plausible mechanistic roles are ignored

## Evidence
