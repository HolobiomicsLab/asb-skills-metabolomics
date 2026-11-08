---
name: drug-combination-synergy-analysis
description: Use when you have ranked top-predictor features (e.g., the top 20 features
  per metabolite from Metab8D_network.xlsx with 0-8 confidence scores) and want to
  test whether drugs or drug combinations against those predictors show synergistic
  Bliss interaction scores.
when_to_use_negative:
- Model outputs will be transferred to in vivo settings or other data sources — the
  CCLE-trained models are not directly transferrable due to differences between in
  vitro and in vivo conditions, cell line panels, and limited feature overlap
- The goal is mechanistic explanation of why a drug combination works — the 'guilt
  by association' strategy behind the top predictors may not identify causative mechanisms,
  and network associations are descriptive rather than mechanistic
- Your metabolites of interest fall outside the 225-metabolite panel, which is only
  a fraction of metabolites that exist within a cell across distinct cellular compartments
tools:
- name: scikit-learn
  role: Random Forest Regressor training used to generate the top-predictor features
    whose drug targets are tested for synergy
  repo: https://github.com/sriram-lab/Metab8D
- name: Python
  role: Implementation language for the Metab8D ML workflow and the drug-combination
    synergy contrast script
  repo: https://github.com/sriram-lab/Metab8D
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:b2f49d3c5e0f5cc3e001fbbdc6949ac6e1150d23dc0ba8fa764845c5e94696fe
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - Python
  - scikit-learn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# drug-combination-synergy-analysis

## Summary

Tests whether drug combinations targeting the top transcriptomic predictors from the Metab8D random-forest metabolite models are synergistic, by computing average Bliss interaction scores against a randomly selected negative-control background. Grounds multiomic predictor priorities in therapeutic hypotheses (e.g., nucleotide-metabolism vs TCA-cycle metabolites).

## When to use

Use when you have ranked top-predictor features (e.g., the top 20 features per metabolite from Metab8D_network.xlsx with 0-8 confidence scores) and want to test whether drugs or drug combinations against those predictors show synergistic Bliss interaction scores.

## When NOT to use

- Model outputs will be transferred to in vivo settings or other data sources — the CCLE-trained models are not directly transferrable due to differences between in vitro and in vivo conditions, cell line panels, and limited feature overlap
- The goal is mechanistic explanation of why a drug combination works — the 'guilt by association' strategy behind the top predictors may not identify causative mechanisms, and network associations are descriptive rather than mechanistic
- Your metabolites of interest fall outside the 225-metabolite panel, which is only a fraction of metabolites that exist within a cell across distinct cellular compartments

## Inputs

- Top 20 predictor features per metabolite from Metab8D_network.xlsx (nine feature sets for 225 metabolites, with confidence scores 0-8)
- Feature importances from CCLE-trained random forest models (scikit-learn RandomForestRegressor, 5-fold cross-validation)
- DrugBank drug-target database records mapping predictor genes to drug targets
- DrugCombDB drug combination records with Bliss interaction scores

## Outputs

- drug_combination_bliss_scores.csv — annotated drug-combination synergy table (nucleotide-metabolite-targeting combinations annotated as synergistic, TCA-cycle-targeting as antagonistic)
- Average Bliss interaction score for combinations targeting transcriptomic top predictors
- Average Bliss score for randomly selected drug combinations (negative-control background)

## How to apply

First load the top predictor features (feature importances) from the Metab8D random forest regression models for each feature set, prioritizing the transcriptomic top predictors since coding and non-coding RNAs emerged as top predictors of metabolomic variation. Map predictor genes to known drug targets using the DrugBank drug-target database. Retrieve drug combination records from DrugCombDB and retain combinations whose targets overlap the top predictors. Compute the average Bliss synergy score for drug combinations targeting transcriptomic top predictors, and separately compute the average Bliss score for randomly selected drug combinations as a negative-control background. Contrast the two groups, annotating nucleotide-metabolite-targeting combinations as synergistic and TCA-cycle-targeting combinations as antagonistic, and write the annotated table to drug_combination_bliss_scores.csv.

## Related tools

- **scikit-learn** (Random Forest Regressor training used to generate the top-predictor features whose drug targets are tested for synergy) — https://github.com/sriram-lab/Metab8D
- **Python** (Implementation language for the Metab8D ML workflow and the drug-combination synergy contrast script) — https://github.com/sriram-lab/Metab8D

## Examples

```
import pandas as pd; network = pd.read_excel('Metab8D_network.xlsx')  # top 20 predictors per metabolite, then map to DrugBank targets
```

## Evaluation signals

- Average Bliss interaction score for drug combinations targeting transcriptomic top predictors must exceed +4
- Average Bliss score for randomly selected drug combinations (negative control) must be below -5
- Nucleotide-metabolite-targeting combinations are annotated as synergistic and TCA-cycle-targeting combinations as antagonistic in drug_combination_bliss_scores.csv
- Input top-predictor features match the top 20 features per metabolite in Metab8D_network.xlsx with confidence scores 0-8 based on 8 control experiments
- Predictor genes must map to DrugBank targets and DrugCombDB combinations via overlapping targets before score aggregation

## Limitations

- Analysis was done in cancer cell lines in culture and may not reflect metabolism in vivo
- Experimentally validating proposed interactions from this study, even for a small percentage of interactions, would require an immense effort
- Mutations were excluded from random forest model results due to their binary structure, so mutation-based predictors are absent from the top-predictor lists used for drug targeting
- Findings are associations from systems-level data, not necessarily mechanistic

## Evidence
