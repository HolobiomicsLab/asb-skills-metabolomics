---
name: mass-action-law-flux-prediction-from-metabolomics
description: Use when you have intracellular metabolomics data for paired biological
  samples (e.g. breast cell lines) and need to infer which reactions are metabolically
  controlled, without relying on transcriptomics.
when_to_use_negative:
- When the goal is transcriptional regulation only and metabolomics data are unavailable
  (use getRASscore/rasIntegration instead).
- When extracellular metabolomics (ysi ratios) are the sole available data; exo-metabolomics
  in INTEGRATE is used to set flux constraints on the model, not to build RPS.
- When metabolites are not uniquely mappable to the model's metabolite ID space (no
  conversion dictionary available), as the mapping step is required before the mass-action
  inference.
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0602
tools:
- name: INTEGRATE
  role: pipeline hosting the createMetabolicDataset and concordanceAnalysis steps
    that implement the mass-action-based flux prediction from metabolomics
  repo: github.com/QLSLab/integrate
- name: Python
  role: language in which the createMetabolicDataset.py and downstream concordance
    scripts are implemented
- name: GLPK
  role: LP solver used by the pipeline for the constraint-based steps adjacent to
    (and downstream of) the metabolomics-only prediction
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1009337
    title: 'INTEGRATE: Model-based multi-omics data integration to characterize multi-level
      metabolic regulation'
  source_content_hash: sha256:23b17f8b40cd9e956f4fe5d607160d3f0e94d4081ee4f95c558f8af48935929b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GLPK
  - INTEGRATE
  - MATLAB
  - Python
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 18
---

# Mass action law flux prediction from metabolomics

## Summary

Predicts reaction-wise relative flux differences between cell lines from intracellular metabolomics alone, by applying a mass action law formulation (flux directionality from substrate/product log2 ratios) and a metabolic threshold (mean ratio > 1.2) to build RPS classes that are later intersected with transcriptomics-derived flux predictions.

## When to use

Use when you have intracellular metabolomics data for paired biological samples (e.g. breast cell lines) and need to infer which reactions are metabolically controlled, without relying on transcriptomics.

## When NOT to use

- When the goal is transcriptional regulation only and metabolomics data are unavailable (use getRASscore/rasIntegration instead).
- When extracellular metabolomics (ysi ratios) are the sole available data; exo-metabolomics in INTEGRATE is used to set flux constraints on the model, not to build RPS.
- When metabolites are not uniquely mappable to the model's metabolite ID space (no conversion dictionary available), as the mapping step is required before the mass-action inference.

## Inputs

- Intracellular metabolomics dataset (e.g. metabolomics_LM.csv)
- Metabolic model in irreversible SBML format (e.g. ENGRO2_irrev.xml)
- Conversion file between metabolomics IDs and model metabolite IDs (e.g. metsEngroVsMetabolomics.csv)
- List of input cell-line names (e.g. MCF102A, SKBR3, MCF7, MDAMB231, MDAMB361)

## Outputs

- Per-pair metabolomic t-test output files (e.g. resultsMetabolomic*.csv)
- Log2 ratio between means per metabolite pair (e.g. medie_Met.csv)
- Reaction Propensity Score (RPS) classification per reaction per cell-line pair

## How to apply

Start from the intracellular metabolomics dataset (e.g. metabolomics_LM.csv) mapped onto the metabolic model's metabolite IDs via the conversion table (metsEngroVsMetabolomics.csv). For each cell-line pair, compute the ratio of means per metabolite; a metabolite is deemed statistically different when the mean ratio exceeds valLog (default 1.2), which is the pipeline's empirical threshold for metabolomic significance. Apply the mass action law formulation: if the substrate side of a reaction shows a monotonic increase relative to the product side across the pair, the reaction is predicted to have increased flux in that direction (metabolic regulation only, neglecting enzymatic activity). Use createMetabolicDataset.py with data_quality_filter=1 to retain only high-quality measured metabolites, producing per-pair t-test output files and a log2 ratio file (medie_Met.csv) that feed the downstream concordanceAnalysis.py step. The resulting Reaction Propensity Score (RPS) classes are compared with RAS and FFD classifications to classify reactions as metabolically or transcriptionally regulated.

## Related tools

- **INTEGRATE** (pipeline hosting the createMetabolicDataset and concordanceAnalysis steps that implement the mass-action-based flux prediction from metabolomics) — github.com/QLSLab/integrate
- **Python** (language in which the createMetabolicDataset.py and downstream concordance scripts are implemented)
- **GLPK** (LP solver used by the pipeline for the constraint-based steps adjacent to (and downstream of) the metabolomics-only prediction)

## Examples

```
python pipeline/createMetabolicDataset.py  # with metabolic_data='metabolomics_LM.csv', metabolic_model='ENGRO2_irrev.xml', dict_to_convert_metnames='metsEngroVsMetabolomics.csv', valLog=1.2, data_quality_filter=1
```

## Evaluation signals

- The log2 ratio file medie_Met.csv must contain finite log2 values for every metabolite pair that passed data_quality_filter=1 (eps=0.00001 is used to avoid undefined ratios).
- Each per-pair t-test output file should exist for all requested cell-line pairs (c_1 vs c_2).
- Metabolites with mean ratio above valLog=1.2 should be flagged as statistically different in the t-test output.
- Downstream concordanceAnalysis should yield non-degenerate RPS vs RAS and RPS vs FFD Cohen kappa values (not all equal to 0) for the breast cell-line dataset.
- The RPE reaction (pentose phosphate pathway) should be classified as transcriptionally and metabolically regulated in both ENGRO2 and the Recon3D-derived model, serving as a sanity anchor for the mass-action inference.

## Limitations

- The mass action law formulation neglects enzymatic activity; it assumes flux differences are driven only by substrate availability.
- The threshold valLog=1.2 is an empirical choice for metabolomic significance; changing it changes which reactions are called metabolically regulated.
- Flux prediction is restricted to metabolites that pass the data_quality_filter and map onto the model; missing or low-quality measurements reduce coverage.

## Evidence

- [methods] metabolic threshold definition: "valLog: value above which the ratio between the means of two cell lines are considered statistically different. Default value: 1.2"
- [intro] mass-action formulation rationale: [evidence span withheld — anchor status: not_found]
- [readme] quality filter applied to metabolomics: "data_quality_filter: quality filter. Default value: 1"
- [readme] conversion file requirement: "dict_to_convert_metnames: Conversion file between ID of metabolites in metabolomics dataset and ID of metabolites in the input model. Default value: 'metsEngroVsMetabolomics.csv'"
- [abstract] intersection discriminates regulation class: "We discriminate fluxes regulated at the metabolic and/or gene expression level by intersecting these two output datasets"
- [readme] avoid undefined ratios: "eps: a very close to 0 value to avoid undefined ratios. Default value: 0.00001."
