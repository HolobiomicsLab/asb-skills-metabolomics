---
name: metabolite-predictor-annotation
description: Use when you have ranked feature importances (e.g., random forest top
  predictors) for metabolites and want to link them to known drug targets and evaluate
  enrichment of synergistic drug combinations against a random background.
when_to_use_negative:
- Input is raw omics data rather than ranked feature importances — run Metab8D random
  forest training and Shapley/feature-importance extraction first (and exclude genomic
  mutations, which were dropped from RF results due to their binary structure)
- Predictors cannot be mapped to gene identifiers or lack DrugBank drug-target entries
  — the drug-combination overlap step will be empty
- Goal is de novo discovery of synergistic drug pairs rather than annotation/enrichment
  testing of already-identified predictor targets
edam_operation: http://edamontology.org/operation_3715
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3407
- http://edamontology.org/topic_0154
tools:
- name: scikit-learn
  role: RandomForestRegressor used to train metabolite models and extract the top
    predictor features that are then drug-annotated
  repo: https://github.com/sriram-lab/Metab8D
- name: Python
  role: Scripting language for the ML workflow and the drug-combination synergy annotation
    script
  repo: https://github.com/sriram-lab/Metab8D
- name: COBRA toolbox
  role: Identifies all reactions associated with relevant metabolites to link predictors
    to metabolic pathway context
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:01f505f9595a21129432b959d9ef5fe742d7395d381c194349647e435876266b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - COBRA toolbox
  - Python
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3715
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3407
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# metabolite-predictor-annotation

## Summary

Annotates top ML predictor features of metabolites with drug-target and drug-combination information (DrugBank, DrugCombDB) to test whether synergistic drug combinations are enriched among compounds targeting those predictors, quantified via Bliss synergy scores.

## When to use

Use when you have ranked feature importances (e.g., random forest top predictors) for metabolites and want to link them to known drug targets and evaluate enrichment of synergistic drug combinations against a random background.

## When NOT to use

- Input is raw omics data rather than ranked feature importances — run Metab8D random forest training and Shapley/feature-importance extraction first (and exclude genomic mutations, which were dropped from RF results due to their binary structure)
- Predictors cannot be mapped to gene identifiers or lack DrugBank drug-target entries — the drug-combination overlap step will be empty
- Goal is de novo discovery of synergistic drug pairs rather than annotation/enrichment testing of already-identified predictor targets

## Inputs

- Top predictor feature list per metabolite (e.g., Metab8D_network.xlsx: top 20 features for 225 RF metabolite models from nine feature sets with confidence scores 0-8)
- DrugBank drug-target database (gene-to-drug mappings)
- DrugCombDB drug combination records with Bliss synergy scores
- Gene name mapping (recon_mapping csv with common gene name to BiggID translations)

## Outputs

- drug_combination_bliss_scores.csv: annotated drug-combination synergy table flagging nucleotide-metabolite-targeting combinations as synergistic and TCA-cycle-targeting combinations as antagonistic
- Per-metabolite drug-target annotation of top predictors

## How to apply

Load the top predictor features (e.g., top 20 per metabolite from Metab8D_network.xlsx, the random forest feature importances for 225 metabolites across nine feature sets, including transcriptomic coding transcripts). Map predictor genes to known drug targets using the DrugBank drug-target database, then retrieve drug combination records from DrugCombDB and retain combinations whose targets overlap the top predictors. Compute the average Bliss synergy score for combinations targeting the transcriptomic top predictors and contrast it with the average Bliss score of randomly selected drug combinations as a negative control. In Recon8D this yielded an average Bliss score > +4 for transcriptomic top-predictor-targeting combinations (annotated synergistic) versus < -5 for random combinations (antagonistic), with the annotated table written to drug_combination_bliss_scores.csv. Validation for select metabolites used Shapley analysis to quantify feature importances before annotation.

## Related tools

- **scikit-learn** (RandomForestRegressor used to train metabolite models and extract the top predictor features that are then drug-annotated) — https://github.com/sriram-lab/Metab8D
- **Python** (Scripting language for the ML workflow and the drug-combination synergy annotation script) — https://github.com/sriram-lab/Metab8D
- **COBRA toolbox** (Identifies all reactions associated with relevant metabolites to link predictors to metabolic pathway context)

## Examples

```
python ML_function.ipynb --feature-set transcriptomics --top-n 20; python annotate_drug_targets.py --features Metab8D_network.xlsx --drugbank drugbank_targets.csv --drugcombdb drugcombdb.csv --out drug_combination_bliss_scores.csv
```

## Evaluation signals

- Average Bliss interaction score for combinations targeting transcriptomic top predictors exceeds +4 (reproduces reported synergistic pattern)
- Average Bliss score for randomly selected drug combinations is below -5 (negative-control background)
- Output drug_combination_bliss_scores.csv contains both synergy group assignments: nucleotide-metabolite-targeting = synergistic, TCA-cycle-targeting = antagonistic
- Predictor genes successfully map to DrugBank targets with BiggID translations via recon_mapping
- Feature importances of annotated predictors are supported by Shapley analysis and confidence scores (0-8 controls) in Metab8D_network.xlsx

## Limitations

- Guilt-by-association network inference may not identify causative mechanisms; associations from systems-level data are descriptive and not necessarily mechanistic
- Analysis was done in cancer cell lines in culture (CCLE) and may not reflect metabolism in vivo; models are not directly transferrable to other sources due to differences between in vitro and in vivo conditions and limited feature overlap
- The metabolite panel of 225 metabolites is only a fraction of those that exist within a cell across compartments, limiting coverage of druggable metabolic targets

## Evidence

- [results] Drug combinations targeting top predictors highly synergistic: [evidence span withheld — anchor status: not_found]
