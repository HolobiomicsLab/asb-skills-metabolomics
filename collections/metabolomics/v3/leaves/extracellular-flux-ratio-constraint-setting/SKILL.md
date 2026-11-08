---
name: extracellular-flux-ratio-constraint-setting
description: Reference for imposing extracellular flux ratio constraints from ysi_ratio.csv
  (e.g. lactate/glucose, lactate/glutamine, glutamate/glutamine) onto constraint-based
  models during rasIntegration, together with medium.csv nutrient bounds and RAS-derived
  reaction bounds.
when_to_use_negative:
- When no extracellular flux ratio measurements (e.g. lactate/glucose, lactate/glutamine,
  glutamate/glutamine from exo-metabolomics) are available for the cell lines under
  study — set imposeYSI='N' rather than fabricating ratios.
- When the goal is to generate models from transcriptomics data alone without exo-metabolomics
  constraints (imposeMedium='N' and imposeYSI='N'), since this skill specifically
  requires ysi_ratio.csv and medium.csv inputs.
- When the input model is not a constraint-based stoichiometric metabolic model in
  SBML format compatible with the pipeline's GLPK-solver flux-bound manipulation (e.g.
  already irreversible .mat format from Step 5).
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_0602
tools:
- name: rasIntegration
  role: Pipeline script (Step 4) that integrates RAS scores, nutrient-availability
    bounds (medium.csv), and extracellular flux ratio constraints (ysi_ratio.csv)
    into the input generic model to generate cell relative models.
  repo: https://github.com/qLSLab/integrate
- name: getNormalizedRAS
  role: Upstream Step 3 that produces ENGRO2_wNormalizedRAS.csv, whose normalized
    RAS scores are used to constrain reaction bounds proportionally to transcriptomic
    regulation.
  repo: https://github.com/qLSLab/integrate
- name: GLPK
  role: Linear programming solver used for constraint-based modeling to translate
    the imposed ratio, medium, and RAS constraints into consistent flux bounds.
  repo: https://www.gnu.org/software/glpk/
- name: randomSampling
  role: Downstream Step 6 that samples the feasible flux region of each cell relative
    model produced by this skill, re-imposing the same ysi_ratio.csv ratio constraints
    per replica.
  repo: https://github.com/qLSLab/integrate
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1009337
    title: 'INTEGRATE: Model-based multi-omics data integration to characterize multi-level
      metabolic regulation'
  source_content_hash: sha256:228ca51e9ab5b3bb05ca6359dd0905055b083075dcae51404bb5bb3c1595ce67
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qLSLab/integrate
  tools:
  - GLPK
  - INTEGRATE
  - Python
  - getNormalizedRAS
  - randomSampling
  - rasIntegration
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# extracellular-flux-ratio-constraint-setting

## Summary

Impose experimentally measured extracellular metabolite flux ratios (e.g. lactate/glucose, lactate/glutamine, glutamate/glutamine from the ysi_ratio.csv file) as constraints on a constraint-based metabolic model, alongside nutrient-availability bounds and RAS-derived bounds, to generate cell-line-specific models (e.g. ENGRO2_MCF102A.xml, ENGRO2_MDAMB231.xml).

## When to use

Reference for imposing extracellular flux ratio constraints from ysi_ratio.csv (e.g. lactate/glucose, lactate/glutamine, glutamate/glutamine) onto constraint-based models during rasIntegration, together with medium.csv nutrient bounds and RAS-derived reaction bounds.

## When NOT to use

- When no extracellular flux ratio measurements (e.g. lactate/glucose, lactate/glutamine, glutamate/glutamine from exo-metabolomics) are available for the cell lines under study — set imposeYSI='N' rather than fabricating ratios.
- When the goal is to generate models from transcriptomics data alone without exo-metabolomics constraints (imposeMedium='N' and imposeYSI='N'), since this skill specifically requires ysi_ratio.csv and medium.csv inputs.
- When the input model is not a constraint-based stoichiometric metabolic model in SBML format compatible with the pipeline's GLPK-solver flux-bound manipulation (e.g. already irreversible .mat format from Step 5).

## Inputs

- ysi_ratio.csv - extracellular flux ratio dataset with columns: 'Ratio' listing the extracellular flux ratios to impose, plus n columns for each cell line listing the corresponding value in each replica (e.g. _A, _B)
- medium.csv - nutrients availability dataset with columns: 'Rxn' listing nutrients uptaken within the model, plus two columns per cell line listing lower bound (cell line + _LB) and upper bound (cell line + _UB)
- ENGRO2_wNormalizedRAS.csv - normalized RAS scores per cell line (mean_XXX and norm_XXX columns) from Step 3 getNormalizedRAS
- ENGRO2 model - unconstrained SBML model (modelId='ENGRO2')
- lcellLines - list of input sample names (default ['MCF102A', 'MDAMB231', 'SKBR3', 'MCF7', 'MDAMB361'])
- lReplicas - list of sample replicas (default ['_A', '_B'])

## Outputs

- One SBML model per input cell line named modelId + '_' + cellLineName + '.xml' (e.g. ENGRO2_MCF102A.xml, ENGRO2_MDAMB231.xml, ENGRO2_SKBR3.xml, ENGRO2_MCF7.xml, ENGRO2_MDAMB361.xml) with extracellular flux ratio, nutrient-availability, and RAS-derived constraints imposed

## How to apply

Run the rasIntegration script (python pipeline/rasIntegration.py) with imposeYSI='Y' so that extracellular flux ratio constraints from ysi_ratio.csv are integrated, alongside imposeMedium='Y' for nutrient-availability bounds from medium.csv (cell line _LB and _UB columns per uptake reaction) and imposeRasConstraints='Y' to translate normalized RAS scores (from ENGRO2_wNormalizedRAS.csv) into flux bounds. The ysi_ratio.csv file lists one column per cell line per extracellular flux ratio, with values for each replica (e.g. _A, _B), and the ratio constraints are applied to the lactate demand reaction EX_lac__L_e, glucose uptake EX_glc__D_e, glutamine uptake EX_gln__L_e, and glutamate demand EX_gluOUT__L_e. Fluxes are then bounded proportionally to the normalized RAS scores so transcriptional regulation is translated into consistent flux bounds via constraint-based modeling (GLPK solver), producing one SBML model per cell line named modelId + '_' + cellLineName + '.xml'. Rationale: exo-metabolomics data improve model predictions by constraining selected extracellular fluxes, narrowing the feasible flux region before downstream random sampling and Mann-Whitney U / t-test comparisons.

## Related tools

- **rasIntegration** (Pipeline script (Step 4) that integrates RAS scores, nutrient-availability bounds (medium.csv), and extracellular flux ratio constraints (ysi_ratio.csv) into the input generic model to generate cell relative models.) — https://github.com/qLSLab/integrate
- **getNormalizedRAS** (Upstream Step 3 that produces ENGRO2_wNormalizedRAS.csv, whose normalized RAS scores are used to constrain reaction bounds proportionally to transcriptomic regulation.) — https://github.com/qLSLab/integrate
- **GLPK** (Linear programming solver used for constraint-based modeling to translate the imposed ratio, medium, and RAS constraints into consistent flux bounds.) — https://www.gnu.org/software/glpk/
- **randomSampling** (Downstream Step 6 that samples the feasible flux region of each cell relative model produced by this skill, re-imposing the same ysi_ratio.csv ratio constraints per replica.) — https://github.com/qLSLab/integrate

## Examples

```
python pipeline/rasIntegration.py imposeYSI=Y imposeMedium=Y imposeRasConstraints=Y rasNormFileName=ENGRO2_wNormalizedRAS.csv ysiFileName=ysi_ratio.csv mediumFileName=medium.csv modelId=ENGRO2 lcellLines=['MCF102A','MDAMB231','SKBR3','MCF7','MDAMB361'] biomassRxn=Biomass lacRxn=EX_lac__L_e glcRxn=EX_glc__D_e glnRxn=EX_gln__L_e gluRxn=EX_gluOUT__L_e lReplicas=['_A','_B']
```

## Evaluation signals

- Exactly five valid SBML XML files are produced (ENGRO2_MCF102A.xml, ENGRO2_MDAMB231.xml, ENGRO2_SKBR3.xml, ENGRO2_MCF7.xml, ENGRO2_MDAMB361.xml) and each parses as an SBML model.
- Each output model's flux bounds reflect the extracellular flux ratios listed in ysi_ratio.csv for that cell line and replica (e.g. _A and _B), plus the cellLine_LB / cellLine_UB nutrient bounds from medium.csv.
- Reaction bounds in each model are proportionally consistent with the norm_XXX normalized RAS columns in ENGRO2_wNormalizedRAS.csv for the corresponding cell line.
- The downstream randomSampling step can sample the feasible flux region of each model (with epsilon lower bound 1e-4 on the Biomass reaction), confirming the constrained models are feasible.
- Mann-Whitney U test outputs (Step 7) between cell-line pairs show non-degenerate flux distributions, indicating the imposed constraints did not over-restrict the feasible region.

## Limitations

- Extracellular flux ratio constraints require per-replica measurements for each cell line in ysi_ratio.csv; missing or unreliable ratio values limit the constraint's validity.
- The default biomass reaction, lactate/glucose/glutamine/glutamate exchange reactions (EX_lac__L_e, EX_glc__D_e, EX_gln__L_e, EX_gluOUT__L_e), and cell-line list must match the input model; other models require adjusting these defaults.
- The pipeline was tested with the GLPK solver only; other solvers may behave differently.
- Extracellular flux ratio constraints constrain only selected extracellular fluxes; intracellular regulation is additionally captured by RAS-derived bounds and mass-action metabolomics predictions in other pipeline steps.

## Evidence

- [readme] ysiFileName: the input extracellular flux ratio dataset. This file consists of the following columns: "Ratio" listing the extracellular flux ratios to impose within the model; n columns for each cell line listing for each extracellular flux ratio the corresponding value in each replica. Default value: 'ysi_ratio.csv'.: "ysiFileName: the input extracellular flux ratio dataset. This file consists of the following columns: "Ratio" listing the extracellular flux ratios to impose within the model; n columns for each cell"
- [supplementary] Impose extracellular flux-ratio constraints from ysi_ratio.csv (e.g. lactate/glucose, lactate/glutamine, glutamate/glutamine ratios), which lists one value per replica (_A, _B) for each of the five cell lines.: [evidence span withheld — anchor status: not_found]
- [intro] INTEGRATE sets constraints also on selected extracellular fluxes, according to exo-metabolomics data.: "INTEGRATE sets constraints also on selected extracellular fluxes, according to exo-metabolomics data."
- [supplementary] Impose nutrient-availability constraints from medium.csv, which provides a lower bound (cellLine_LB) and upper bound (cellLine_UB) for each uptake reaction per cell line.: [evidence span withheld — anchor status: not_found]
- [readme] The Pipeline was tested using GLPK solver (https://www.gnu.org/software/glpk/).: "The Pipeline was tested using GLPK solver (https://www.gnu.org/software/glpk/)."
- [readme] A SBML model for each input cell line named as modelId + '_' + cellLineName + '.xml': "A SBML model for each input cell line named as modelId + '_' + cellLineName + '.xml'"
