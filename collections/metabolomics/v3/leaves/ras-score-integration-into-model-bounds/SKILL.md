---
name: ras-score-integration-into-model-bounds
description: Use when per-cell-line FPKM transcriptomics have been converted to normalized
  RAS scores and you must constrain a stoichiometric model's reaction bounds to build
  cell-relative models for flux sampling.
when_to_use_negative:
- Raw RAS scores are not yet normalized — run getNormalizedRAS first
- Only metabolomics data are available and no transcriptomics-derived RAS table exists
- The goal is generic FBA on a published model with no cell-line-specific expression
  constraints
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_0203
tools:
- name: INTEGRATE
  role: Pipeline containing rasIntegration that imposes RAS-proportional bounds on
    the model
  repo: https://github.com/qLSLab/integrate
- name: Python
  role: Language used to run the rasIntegration script
- name: GLPK
  role: LP solver used by constraint-based modeling to solve the bounded model
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1009337
    title: 'INTEGRATE: Model-based multi-omics data integration to characterize multi-level
      metabolic regulation'
  source_content_hash: sha256:93d975e1f1685e506e5bfbab6412efca4b2fa0e503ea605b4edcec4513148307
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qLSLab/integrate
  tools:
  - GLPK
  - INTEGRATE
  - Python
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# ras-score-integration-into-model-bounds

## Summary

Translates normalized Reaction Activity Scores (RAS) derived from transcriptomics GPR rules into proportional reaction bounds within a constraint-based model, producing cell-relative SBML models (e.g. ENGRO2 per breast cancer cell line).

## When to use

Use when per-cell-line FPKM transcriptomics have been converted to normalized RAS scores and you must constrain a stoichiometric model's reaction bounds to build cell-relative models for flux sampling.

## When NOT to use

- Raw RAS scores are not yet normalized — run getNormalizedRAS first
- Only metabolomics data are available and no transcriptomics-derived RAS table exists
- The goal is generic FBA on a published model with no cell-line-specific expression constraints

## Inputs

- Unconstrained ENGRO2 SBML model (modelId='ENGRO2')
- Normalized RAS table ENGRO2_wNormalizedRAS.csv (norm_XXX columns per cell line)
- Nutrient availability dataset medium.csv (Rxn + cellLine_LB/cellLine_UB columns)
- Extracellular flux ratio dataset ysi_ratio.csv (Ratio + per-replica values)
- Cell line list ['MCF102A','MDAMB231','SKBR3','MCF7','MDAMB361'] and replica list ['_A','_B']

## Outputs

- One cell-relative SBML model per sample, named modelId + '_' + cellLineName + '.xml' (e.g. ENGRO2_MCF7.xml, ENGRO2_MDAMB361.xml)

## How to apply

Run the rasIntegration step after getGPRsFromModel, getRASscore and getNormalizedRAS: load the unconstrained ENGRO2 SBML model plus ENGRO2_wNormalizedRAS.csv, then run `python pipeline/rasIntegration.py` with imposeRasConstraints='Y', imposeMedium='Y' (nutrient bounds from medium.csv with per-cell-line _LB/_UB columns), and imposeYSI='Y' (extracellular flux ratios from ysi_ratio.csv with per-replica columns like _A/_B). Proportionally constrain each internal reaction's lower/upper flux boundaries as a function of the cell's normalized RAS so differential enzyme expression translates into consistent flux differences, using GLPK as the solver. Set biomassRxn='Biomass' and exchange reactions EX_lac__L_e, EX_glc__D_e, EX_gln__L_e, EX_gluOUT__L_e; supply lcellLines=['MCF102A','MDAMB231','SKBR3','MCF7','MDAMB361'] and lReplicas=['_A','_B']. The rationale is to exploit constraint-based modeling to predict how global relative expression differences translate into flux-level differences, one model per sample.

## Related tools

- **INTEGRATE** (Pipeline containing rasIntegration that imposes RAS-proportional bounds on the model) — https://github.com/qLSLab/integrate
- **Python** (Language used to run the rasIntegration script)
- **GLPK** (LP solver used by constraint-based modeling to solve the bounded model)

## Examples

```
python pipeline/rasIntegration.py
```

## Evaluation signals

- Each output must be a valid SBML XML parseable as a model
- Exactly five files named ENGRO2_{MCF102A,MDAMB231,SKBR3,MCF7,MDAMB361}.xml must be produced
- Reaction bounds must vary proportionally with the normalized RAS (norm_XXX column) of each cell line
- Nutrient uptake bounds must match medium.csv _LB/_UB values per cell line
- Extracellular flux ratios (lac/glc, lac/gln, glu/gln) must equal the ysi_ratio.csv values per replica

## Limitations

- Flux capability is capped by nutrient availability and extracellular flux-ratio constraints, so RAS alone does not fully determine bounds
- Only exchange reactions explicitly named (lactate, glucose, glutamine, glutamate) carry YSI constraints; others are unaffected by this flag
- The step depends on upstream getGPRsFromModel/getRASscore/getNormalizedRAS outputs; changes in the model ID or GPR regex propagate

## Evidence

- [readme] Impose transcriptomics-derived flux bounds: "imposeRasConstraints: 'Y' (yes) or 'N' (no) according to whether transcriptomics derived constraints have to be integrated. Default value: 'Y'."
- [readme] Nutrient availability constraints: "mediumFileName: the input nutrients availability dataset. This file consists of the following columns: "Rxn" listing nutrients uptaken within the model"
- [readme] YSI flux ratio constraints: "ysiFileName: the input extracellular flux ratio dataset. This file consists of the following columns: "Ratio" listing the extracellular flux ratios to impose within the model"
- [readme] Output models per cell line: "A SBML model for each input cell line named as modelId + '_' + cellLineName + '.xml'"
- [readme] Default cell lines: "lcellLines: list of input sample names: Default value: ['MCF102A',  'MDAMB231','SKBR3', 'MCF7', 'MDAMB361']"
- [readme] Normalizing RAS upstream: "File named modelId + 'wNormalizedRAS.csv' containing for each reaction (column *Rxn*) the mean (*mean_XXX* column) and normalized (*norm_XXX* column) RAS"
- [intro] Constraint-based rationale: "Then, INTEGRATE exploits constraint-based modeling to predict how the global relative differences in expression are expected to translate into consistent differences in metabolic fluxes."
- [methods] GLPK solver tested: "The Pipeline was tested using GLPK solver (https://www.gnu.org/software/glpk/)."
