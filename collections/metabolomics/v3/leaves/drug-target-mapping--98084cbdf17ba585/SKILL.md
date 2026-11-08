---
name: drug-target-mapping
description: Use when you have ranked top predictor features from trained metabolite
  models (e.g. Metab8D_network.xlsx top-20 features) and need to convert them into
  drug-target / drug-combination hypotheses with synergy scores.
when_to_use_negative:
- When predictor features are not genes or gene products (e.g. histone PTM, methylation,
  or splicing features) that cannot be directly mapped to DrugBank targets without
  an intermediate gene-level translation
- When the goal is predicting metabolite levels from omics rather than therapeutic
  interpretation of trained models — use the Metab8D ML workflow instead
- When models will be transferred to non-CCLE data sources, since the article notes
  models are not directly transferrable due to in vitro vs in vivo differences and
  limited feature overlap
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3336
tools:
- name: COBRA toolbox
  role: Identify all reactions associated with the relevant metabolites when linking
    predictor genes to metabolic targets
- name: Python
  role: Runs the recon_mapping scripts that match top feature lists with metabolite-reaction
    genes and writes the synergy table
  repo: https://github.com/sriram-lab/Metab8D
- name: scikit-learn
  role: RandomForestRegressor used upstream to produce the feature importances that
    are mapped to drug targets
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:5a1f516892a187bf06fd95aecfe08e9c22c5e7c3dee70ee8eeedddc35a3f59fc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - COBRA toolbox
  - Python
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# drug-target-mapping

## Summary

Maps Metab8D random-forest top predictor genes onto known drug targets (DrugBank) and drug-combination records (DrugCombDB) to test whether synergistic combinations are enriched among therapies targeting top transcriptomic predictors of metabolites in Recon8D.

## When to use

Use when you have ranked top predictor features from trained metabolite models (e.g. Metab8D_network.xlsx top-20 features) and need to convert them into drug-target / drug-combination hypotheses with synergy scores.

## When NOT to use

- When predictor features are not genes or gene products (e.g. histone PTM, methylation, or splicing features) that cannot be directly mapped to DrugBank targets without an intermediate gene-level translation
- When the goal is predicting metabolite levels from omics rather than therapeutic interpretation of trained models — use the Metab8D ML workflow instead
- When models will be transferred to non-CCLE data sources, since the article notes models are not directly transferrable due to in vitro vs in vivo differences and limited feature overlap

## Inputs

- Top-20 predictor features per metabolite from Metab8D_network.xlsx (nine feature sets, confidence scores 0-8)
- DrugBank drug-target database records
- DrugCombDB drug combination records with Bliss interaction scores
- recon_mapping gene-name-to-BiggID translation CSV
- Recon3D reaction-gene associations for metabolites of interest

## Outputs

- drug_combination_bliss_scores.csv — annotated drug-combination synergy table
- Average Bliss synergy score for combinations targeting transcriptomic top predictors
- Average Bliss score for randomly selected combinations (negative control)

## How to apply

First load the top predictor features (feature importances) from the Metab8D random forest regression models for each feature set, prioritizing the transcriptomic top predictors, which emerged as the strongest omic layer for metabolite prediction. Map predictor genes to known drug targets using the DrugBank drug-target database, and retrieve drug-combination records from DrugCombDB, retaining only combinations whose targets overlap the top predictors. For gene-to-reaction mapping, use the recon_mapping MATLAB/Python scripts and the common-gene-name-to-BiggID CSV shipped with Metab8D, with COBRA toolbox functions to identify all reactions associated with the relevant metabolites. Compute the average Bliss interaction score for combinations targeting transcriptomic top predictors and contrast it against randomly selected drug combinations as a negative-control background; annotate nucleotide-metabolite-targeting combinations as synergistic and TCA-cycle-targeting combinations as antagonistic.

## Related tools

- **COBRA toolbox** (Identify all reactions associated with the relevant metabolites when linking predictor genes to metabolic targets)
- **Python** (Runs the recon_mapping scripts that match top feature lists with metabolite-reaction genes and writes the synergy table) — https://github.com/sriram-lab/Metab8D
- **scikit-learn** (RandomForestRegressor used upstream to produce the feature importances that are mapped to drug targets)

## Evaluation signals

- Average Bliss interaction score for combinations targeting transcriptomic top predictors exceeds +4
- Average Bliss score for randomly selected drug combinations is below -5 (negative-control contrast reproduces the reported pattern)
- Every retained DrugCombDB combination has at least one target overlapping the top predictor list for a metabolite model
- Nucleotide-metabolite-targeting combinations are annotated as synergistic and TCA-cycle-targeting combinations as antagonistic in drug_combination_bliss_scores.csv
- Feature-importance lists used for mapping are the confidence-scored top-20 features from Metab8D_network.xlsx (mutations excluded as a feature set)

## Limitations

- Top predictors were enriched for synergistic combination therapies that target compensatory metabolic modulators, but the 'guilt by association' network inference strategy may not identify causative mechanisms
- Analysis was done in cancer cell lines in culture and may not reflect metabolism in vivo
- Experimentally validating proposed interactions, even for a small percentage of interactions, would require an immense effort

## Evidence
