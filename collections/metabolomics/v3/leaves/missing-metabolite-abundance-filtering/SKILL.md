---
name: missing-metabolite-abundance-filtering
description: Use when mapping metabolite identifiers to model identifiers (e.g. via
  metsEngroVsMetabolomics.csv) before computing mass-action-law substrate-product
  propensity scores, and you must decide which reactions can be scored given missing
  abundance measurements.
when_to_use_negative:
- When the analysis targets transcriptomics-derived Reaction Activity Scores (RAS)
  from GPR rules, where no metabolite abundance completeness check applies.
- When at least one substrate of the reaction of interest has no abundance measurement
  in metabolomics_LM.csv — that reaction must be omitted rather than imputed or scored.
- When extracellular exo-metabolomics ratios (ysi_ratio.csv) are being used to constrain
  uptake/secretion bounds instead of intracellular substrate-based propensity scoring.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: INTEGRATE
  role: Pipeline implementing the createMetabolicDataset step that performs the metabolite
    ID mapping and missing-substrate filtering before RPS computation
  repo: github.com/qLSLab/integrate
- name: Python
  role: Runtime for running pipeline/createMetabolicDataset.py which applies the filter
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1009337
    title: 'INTEGRATE: Model-based multi-omics data integration to characterize multi-level
      metabolic regulation'
  source_content_hash: sha256:e6bad926d2852ed727e18c7d32665ba84eb8414457d6d939d507387c19d7d13b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - INTEGRATE
  - Python
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# missing-metabolite-abundance-filtering

## Summary

Filters reactions for the Reaction Propensity Score (RPS) computation so that only reactions whose substrates are all quantified in the intracellular metabolomics dataset (metabolomics_LM.csv) are retained, dropping any reaction with at least one missing substrate concentration. This is the completeness gate applied by INTEGRATE's createMetabolicDataset step before mass-action-law scoring.

## When to use

Use when mapping metabolite identifiers to model identifiers (e.g. via metsEngroVsMetabolomics.csv) before computing mass-action-law substrate-product propensity scores, and you must decide which reactions can be scored given missing abundance measurements.

## When NOT to use

- When the analysis targets transcriptomics-derived Reaction Activity Scores (RAS) from GPR rules, where no metabolite abundance completeness check applies.
- When at least one substrate of the reaction of interest has no abundance measurement in metabolomics_LM.csv — that reaction must be omitted rather than imputed or scored.
- When extracellular exo-metabolomics ratios (ysi_ratio.csv) are being used to constrain uptake/secretion bounds instead of intracellular substrate-based propensity scoring.

## Inputs

- metabolomics_LM.csv intracellular metabolomics dataset
- metsEngroVsMetabolomics.csv metabolite ID conversion dictionary
- Stoichiometric coefficients per reaction from the input metabolic model (e.g. ENGRO2_irrev.xml)

## Outputs

- Set of reactions whose substrates are all quantified (eligible for RPS computation, 81 reactions in the article)
- List of reactions excluded because at least one substrate abundance is missing

## How to apply

Load the stoichiometric coefficients of every substrate for each reaction from the irreversible model (e.g. ENGRO2_irrev.xml), then map metabolomics identifiers to model identifiers using the conversion dictionary metsEngroVsMetabolomics.csv. For each reaction, check whether ALL of its substrates have quantified abundances in metabolomics_LM.csv across the cell lines of interest (MCF102A, SKBR3, MCF7, MDAMB231, MDAMB361). Retain a reaction only if every substrate is quantified; omit any reaction with at least one missing substrate. The rationale, per the mass action law used by INTEGRATE, is that the propensity is the product of substrate concentrations raised to their stoichiometric coefficients, so a single missing term makes the score undefined. The output of this filtering feeds directly into computing RPS for the eligible reactions and normalizing per-cell-line values.

## Related tools

- **INTEGRATE** (Pipeline implementing the createMetabolicDataset step that performs the metabolite ID mapping and missing-substrate filtering before RPS computation) — github.com/qLSLab/integrate
- **Python** (Runtime for running pipeline/createMetabolicDataset.py which applies the filter)

## Examples

```
python pipeline/createMetabolicDataset.py --metabolic_model ENGRO2_irrev.xml --metabolic_data metabolomics_LM.csv --dict_to_convert_metnames metsEngroVsMetabolomics.csv
```

## Evaluation signals

- Count of retained reactions equals the number of reactions whose substrates are all quantified in the metabolomics data (article reports 81 eligible reactions for ENGRO2).
- No reaction with a missing substrate abundance appears in the RPS output dataset.
- Every retained reaction's substrates map successfully to metabolomics identifiers via metsEngroVsMetabolomics.csv.
- RPS values computed for retained reactions follow the mass action law: product of substrate concentrations raised to their stoichiometric coefficients.

## Limitations

- Reactions with any missing substrate abundance cannot be scored and are omitted entirely, reducing coverage of the metabolic model.
- The filter depends on the accuracy of the metabolite identifier conversion dictionary; unmapped metabolites cause substrates to be treated as missing.
- RPS is computed by neglecting enzymatic activity, so the score reflects substrate availability only, not catalytic regulation.

## Evidence

- [methods] Filter reactions, retaining only those whose substrates are all quantified in the metabolomics data and omitting any reaction with at least one missing substrate.: [evidence span withheld — anchor status: not_found]
- [methods] Mass action law definition of RPS: [evidence span withheld — anchor status: not_found]
- [methods] Mapping metabolomics IDs to model IDs: "dict_to_convert_metnames: Conversion file between ID of metabolites in metabolomics dataset and ID of metabolites in the input model. Default value: 'metsEngroVsMetabolomics.csv'"
- [readme] createMetabolicDataset usage from README: "Step 9: Create metabolomic statistical test dataset * Aim: create the metabolomic dataset to perform concordance analysis * Usage: `python pipeline/createMetabolicDataset.py`"
- [readme] Metabolomics dataset file name default: "metabolic_data: dataset of metabolites. Default value: 'metabolomics_LM.csv'"
- [intro] Mass action law neglecting enzymatic activity: "INTEGRATE uses intracellular metabolomics datasets and the mass action law formulation to predict how differences in substrate availability translate into differences in metabolic fluxes (metabolic"
