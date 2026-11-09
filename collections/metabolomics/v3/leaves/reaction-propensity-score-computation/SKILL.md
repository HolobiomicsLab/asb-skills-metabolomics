---
name: reaction-propensity-score-computation
description: Use when you need to predict how differences in substrate availability
  translate into differences in metabolic fluxes from intracellular metabolomics data
  and an irreversible stoichiometric model, to discriminate metabolically regulated
  reactions in INTEGRATE.
when_to_use_negative:
- When substrates of a reaction are not all quantified in the intracellular metabolomics
  dataset — such reactions must be omitted, and the RPS cannot be computed for them.
- When the goal is to characterize transcriptional regulation rather than metabolic
  regulation — RPS isolates the metabolic layer and neglects enzymatic activity; use
  getGPRsFromModel/getRASscore instead.
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3174
tools:
- name: INTEGRATE
  role: Pipeline that performs the mass action law computation of RPS from intracellular
    metabolomics mapped onto the model reactions
  repo: github.com/qLSLab/integrate
- name: Python
  role: Language used to implement the pipeline scripts (from release version 3.0
    on)
  repo: github.com/qLSLab/integrate
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1009337
    title: 'INTEGRATE: Model-based multi-omics data integration to characterize multi-level
      metabolic regulation'
  source_content_hash: sha256:bd65f6ef3b77fd0e91fc62c2d74153c48fe12e75cf8a736f9e22d407e96e64cc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - INTEGRATE
  - Python
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# reaction-propensity-score-computation

## Summary

Computes the Reaction Propensity Score (RPS) dataset from intracellular metabolomics data for each model reaction and sample by applying the mass action law: the product of quantified substrate concentrations each raised to its stoichiometric coefficient, neglecting enzymatic activity.

## When to use

Use when you need to predict how differences in substrate availability translate into differences in metabolic fluxes from intracellular metabolomics data and an irreversible stoichiometric model, to discriminate metabolically regulated reactions in INTEGRATE.

## When NOT to use

- When substrates of a reaction are not all quantified in the intracellular metabolomics dataset — such reactions must be omitted, and the RPS cannot be computed for them.
- When the goal is to characterize transcriptional regulation rather than metabolic regulation — RPS isolates the metabolic layer and neglects enzymatic activity; use getGPRsFromModel/getRASscore instead.

## Inputs

- Irversible metabolic model in SBML format (default: ENGRO2_irrev.xml) providing substrate stoichiometric coefficients
- Intracellular metabolomics dataset CSV (default: metabolomics_LM.csv)
- Metabolite identifier conversion dictionary (default: metsEngroVsMetabolomics.csv)
- List of input sample names (default: ['MCF102A', 'SKBR3', 'MCF7', 'MDAMB231', 'MDAMB361'])

## Outputs

- RPS dataset with normalized RPS values for each eligible reaction and each cell line
- Means output file (default: medie_Met.csv) including all computed means for downstream concordance analysis

## How to apply

Load the irreversible metabolic model (e.g. ENGRO2_irrev.xml) and extract the stoichiometric coefficient of each substrate for every reaction. Load the intracellular metabolomics dataset (metabolomics_LM.csv) and map metabolite identifiers to model identifiers using the conversion dictionary metsEngroVsMetabolomics.csv. Filter reactions to retain only those whose substrates are all quantified in the metabolomics data, omitting any reaction with at least one missing substrate; in the INTEGRATE proof-of-principle this yielded 81 eligible reactions out of the model. For each retained reaction and each breast cell line (MCF102A, SKBR3, MCF7, MDAMB231, MDAMB361), compute RPS as the product of substrate concentrations raised to their stoichiometric coefficients (mass action law formulation), neglecting enzymatic activity. Normalize the per-cell-line RPS values and assemble the RPS dataset; this RPS is later intersected with RAS and FFD outputs to classify fluxes regulated at the metabolic and/or gene expression level.

## Related tools

- **INTEGRATE** (Pipeline that performs the mass action law computation of RPS from intracellular metabolomics mapped onto the model reactions) — github.com/qLSLab/integrate
- **Python** (Language used to implement the pipeline scripts (from release version 3.0 on)) — github.com/qLSLab/integrate

## Examples

```
python pipeline/createMetabolicDataset.py --metabolic_model ENGRO2_irrev.xml --metabolic_data metabolomics_LM.csv --dict_to_convert_metnames metsEngroVsMetabolomics.csv
```

## Evaluation signals

- Verify the output contains RPS values for exactly the reactions for which quantification of all substrate abundances was available (81 reactions in the proof-of-principle) and none for reactions with any missing substrate.
- Check that the RPS computation matches the mass action law: product of substrate concentrations raised to their stoichiometric coefficients.
- Confirm per-cell-line RPS values are normalized before downstream concordance analysis.
- Cross-check the concordance dataset (Cohen kappa, linearly weighted) between RPS and RAS/FFD outputs to confirm the intersection discriminates metabolically regulated fluxes.

## Limitations

- Reactions with even one missing substrate in the metabolomics data are excluded entirely, reducing the set of reactions with an RPS (81 of the model's reactions in the proof-of-principle).
- The mass action law formulation neglects enzymatic activity, so the RPS captures only the metabolic (substrate-availability) layer of flux regulation, not transcriptional or enzymatic contributions.
- Metabolite identifiers must match the model identifiers; the conversion dictionary metsEngroVsMetabolomics.csv is required to bridge the two naming systems.

## Evidence

- [other] Mass action law with substrate stoichiometric exponents: [evidence span withheld — anchor status: not_found]
- [intro] Metabolic regulation from intracellular metabolomics via mass action law: "INTEGRATE uses intracellular metabolomics datasets and the mass action law formulation to predict how differences in substrate availability translate into differences in metabolic fluxes (metabolic"
- [readme] Data quality filter / difference threshold for metabolomic dataset: "valLog: value above which the ratio between the means of two cell lines are considered statistically different. Default value: 1.2"
- [readme] Identifier conversion dictionary between metabolomics and model IDs: "dict_to_convert_metnames: Conversion file between ID of metabolites in metabolomics dataset and ID of metabolites in the input model. Default value: 'metsEngroVsMetabolomics.csv'"
