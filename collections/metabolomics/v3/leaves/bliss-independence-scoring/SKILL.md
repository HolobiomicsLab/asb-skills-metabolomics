---
name: bliss-independence-scoring
description: Use when ranking drug combinations for synergy relative to a metabolite-predictor
  network — e.g. combinations from DrugCombDB whose targets overlap top features,
  to be scored and compared against random background combinations.
when_to_use_negative:
- The random forest models are not directly transferrable to predicting metabolomics
  from other sources due to differences between in vitro and in vivo conditions, cell
  line panels, and limited feature overlap — do not reuse predictor lists from outside
  CCLE-trained models without revalidation.
- When a mechanistic (rather than associational) synergy ranking is required — the
  'guilt by association' strategy may not identify causative mechanisms, and network
  associations are descriptive, not necessarily mechanistic.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0154
tools:
- name: machine learning (random forest regression)
  role: Generates the metabolite predictor lists whose features are mapped to drug
    targets for Bliss scoring
  repo: https://github.com/sriram-lab/Metab8D
- name: scikit-learn
  role: Implements the Random Forest Regressor used to train the metabolite models
    whose top features drive the synergy enrichment analysis
  repo: https://github.com/sriram-lab/Metab8D
- name: Python
  role: Runs the drug-target mapping and Bliss-score contrast workflow; scripts for
    these algorithms are in the Recon8D GitHub repository
  repo: https://github.com/sriram-lab/Metab8D
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:f56d323887601e664c9723987d23725359bd55cd29db624c47ec1c187cdccf9e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - Python
  - machine learning (random forest regression)
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# bliss-independence-scoring

## Summary

Scores drug-combination synergy using the Bliss independence interaction score, applied here to drug combinations whose targets overlap top predictors from Metab8D random forest regression models, contrasted against a randomly selected drug-combination background.

## When to use

Use when ranking drug combinations for synergy relative to a metabolite-predictor network — e.g. combinations from DrugCombDB whose targets overlap top features, to be scored and compared against random background combinations.

## When NOT to use

- The random forest models are not directly transferrable to predicting metabolomics from other sources due to differences between in vitro and in vivo conditions, cell line panels, and limited feature overlap — do not reuse predictor lists from outside CCLE-trained models without revalidation.
- When a mechanistic (rather than associational) synergy ranking is required — the 'guilt by association' strategy may not identify causative mechanisms, and network associations are descriptive, not necessarily mechanistic.

## Inputs

- Top predictor features (feature importances) from Metab8D random forest regression models for each feature set, including transcriptomic top predictors
- DrugBank drug-target database
- DrugCombDB drug combination records with Bliss synergy scores

## Outputs

- drug_combination_bliss_scores.csv — annotated drug-combination synergy table marking nucleotide-metabolite-targeting combinations as synergistic and TCA-cycle-targeting combinations as antagonistic
- Average Bliss interaction score for predictor-targeting combinations vs random background combinations

## How to apply

First extract the top predictor features for each metabolite from the Metab8D random forest regression models (Metab8D_network.xlsx lists the top 20 features per metabolite across nine feature sets, with confidence scores). Map those predictor genes to known drug targets using the DrugBank drug-target database, then intersect with drug combination records from DrugCombDB to retain combinations whose targets overlap the top predictors. Compute the average Bliss independence interaction score for these predictor-targeting combinations, and separately compute the average Bliss score for randomly selected drug combinations as a negative-control background. Validate by confirming the contrast matches the reported pattern: average Bliss score > +4 for combinations targeting transcriptomic top predictors and < -5 for random combinations.

## Related tools

- **machine learning (random forest regression)** (Generates the metabolite predictor lists whose features are mapped to drug targets for Bliss scoring) — https://github.com/sriram-lab/Metab8D
- **scikit-learn** (Implements the Random Forest Regressor used to train the metabolite models whose top features drive the synergy enrichment analysis) — https://github.com/sriram-lab/Metab8D
- **Python** (Runs the drug-target mapping and Bliss-score contrast workflow; scripts for these algorithms are in the Recon8D GitHub repository) — https://github.com/sriram-lab/Metab8D

## Examples

```
python map_predictors_to_drugs.py --model Metab8D_network.xlsx --drugbank drugbank.csv --drugcomb drugcombdb.csv --bliss-threshold 4 --out drug_combination_bliss_scores.csv
```

## Evaluation signals

- Average Bliss interaction score for combinations targeting transcriptomic top predictors exceeds +4
- Average Bliss score for randomly selected drug combinations is below -5
- Each retained drug combination in the output table is traceable to a top predictor from a Metab8D RF model via a DrugBank target overlap with DrugCombDB records
- Output CSV distinguishes nucleotide-metabolite-targeting combinations as synergistic from TCA-cycle-targeting combinations as antagonistic

## Limitations

- Models are not directly transferrable to predicting metabolomics from other sources due to differences between in vitro and in vivo conditions, cell line panels, and limited feature overlap
- Analysis was done in cancer cell lines in culture and may not reflect metabolism in vivo
- Network associations from systems-level data are descriptive and not necessarily mechanistic
- The metabolite panel of 225 metabolites is only a fraction of metabolites that exist within a cell across compartments

## Evidence

- [full_text] Drug combinations that target top predictors in the transcriptomic model were highly likely to be synergistic, with an average bliss interaction score greater than +4: [evidence span withheld — anchor status: not_found]
- [intro] Our discovery of synthetic lethal interactions among top metabolite predictors can help design combination therapies by targeting compensatory mechanisms: "our discovery of synthetic lethal interactions among top metabolite predictors can help design combination therapies by targeting compensatory mechanisms"
- [readme] Metab8D network generation involves training ML models, assessing their accuracy, and obtaining feature importances thereof: "Metab8D network generation involves training ML models, assessing their accuracy, and obtaining feature importances thereof. This process is repeated for each individual feature set."
- [readme] Top 20 features for 225 RF metabolite models from nine feature sets along with confidence scores: "Top 20 features for 225 RF metabolite models from nine feature sets (excluding genomic mutations) along with confidence scores"
- [discussion] such models are not directly transferrable in predicting metabolomics from other sources: "such models are not directly transferrable in predicting metabolomics from other sources due to several factors, including differences between in vitro and in vivo conditions"
