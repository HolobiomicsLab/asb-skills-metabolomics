---
name: nutrient-availability-constraint-annotation
description: Use when you need to impose nutrient-availability constraints on a stoichiometric
  model (e.g. ENGRO2) from a per-cell-line medium file listing 'Rxn' plus _LB/_UB
  bounds for each cell line, prior to integrating RAS-derived flux limits.
when_to_use_negative:
- When the goal is to compute or normalize RAS scores from transcriptomics (use getRASscore
  / getNormalizedRAS instead); this skill only imposes already-derived bounds.
- When you only need to sample the feasible flux region of an already-constrained
  model — use randomSampling, not constraint annotation.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3316
tools:
- name: rasIntegration
  role: Python pipeline step that reads medium.csv and imposes nutrient-availability
    lower/upper bounds on the input model alongside RAS-derived flux constraints,
    producing per-cell-line SBML models.
  repo: https://github.com/qLSLab/integrate
- name: Python
  role: Runtime environment in which the rasIntegration script executes; install via
    conda (integrate_env.txt) or pip.
  repo: https://github.com/qLSLab/integrate
- name: GLPK
  role: Linear-programming solver used to evaluate the constraint-based model under
    the imposed nutrient-availability bounds.
  repo: https://www.gnu.org/software/glpk/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1009337
    title: 'INTEGRATE: Model-based multi-omics data integration to characterize multi-level
      metabolic regulation'
  source_content_hash: sha256:a492082327499bf1b15dbf4a94ed3c1e1ec4b83804760a8011cf5ef1abbcaabb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qLSLab/integrate
  tools:
  - GLPK
  - INTEGRATE
  - Python
  - rasIntegration
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 18
---

# nutrient-availability-constraint-annotation

## Summary

Annotate nutrient uptake reactions of a constraint-based metabolic model with per-cell-line lower/upper flux bounds derived from an external nutrients-availability dataset (medium.csv), so that extracellular exchange is constrained before RAS integration.

## When to use

Use when you need to impose nutrient-availability constraints on a stoichiometric model (e.g. ENGRO2) from a per-cell-line medium file listing 'Rxn' plus _LB/_UB bounds for each cell line, prior to integrating RAS-derived flux limits.

## When NOT to use

- When the goal is to compute or normalize RAS scores from transcriptomics (use getRASscore / getNormalizedRAS instead); this skill only imposes already-derived bounds.
- When you only need to sample the feasible flux region of an already-constrained model — use randomSampling, not constraint annotation.

## Inputs

- Unconstrained ENGRO2 SBML model (modelId='ENGRO2')
- Nutrients-availability CSV (mediumFileName='medium.csv') with columns: 'Rxn', cellLine_LB, cellLine_UB for each of the five cell lines
- Normalized RAS table ENGRO2_wNormalizedRAS.csv (output of getNormalizedRAS) if RAS constraints are co-imposed
- Extracellular flux-ratio CSV (ysiFileName='ysi_ratio.csv') with a 'Ratio' column and per-cell-line per-replica (_A,_B) values if imposeYSI='Y'

## Outputs

- One SBML model per cell line named modelId + '_' + cellLineName + '.xml' (e.g. ENGRO2_MCF102A.xml)
- Five cell-relative constraint-based models: ENGRO2_MCF102A.xml, ENGRO2_MDAMB231.xml, ENGRO2_SKBR3.xml, ENGRO2_MCF7.xml, ENGRO2_MDAMB361.xml

## How to apply

Load the unconstrained ENGRO2 SBML model and the nutrients-availability CSV whose 'Rxn' column lists uptake reactions and whose per-cell-line columns give lower (cellLine_LB) and upper (cellLine_UB) bounds. Run python pipeline/rasIntegration.py with imposeMedium='Y' (together with imposeRasConstraints='Y' and imposeYSI='Y' if desired), modelId='ENGRO2', biomassRxn='Biomass', lactate/glucose/glutamine/glutamate exchanges EX_lac__L_e, EX_glc__D_e, EX_gln__L_e, EX_gluOUT__L_e, and lcellLines=['MCF102A','MDAMB231','SKBR3','MCF7','MDAMB361']. The script reads mediumFileName='medium.csv' and, using the GLPK solver, constrains each nutrient exchange reaction's bounds proportionally to the cell-line-specific normalized RAS so that transcriptional regulation is translated into flux bounds; extracellular flux-ratio constraints from ysi_ratio.csv (replicas _A,_B) may be imposed in parallel. Write one SBML per sample named modelId + '_' + cellLineName + '.xml'; validity is checked by parsing each XML as an SBML model.

## Related tools

- **rasIntegration** (Python pipeline step that reads medium.csv and imposes nutrient-availability lower/upper bounds on the input model alongside RAS-derived flux constraints, producing per-cell-line SBML models.) — https://github.com/qLSLab/integrate
- **Python** (Runtime environment in which the rasIntegration script executes; install via conda (integrate_env.txt) or pip.) — https://github.com/qLSLab/integrate
- **GLPK** (Linear-programming solver used to evaluate the constraint-based model under the imposed nutrient-availability bounds.) — https://www.gnu.org/software/glpk/

## Examples

```
python pipeline/rasIntegration.py imposeMedium='Y' imposeRasConstraints='Y' imposeYSI='Y' modelId='ENGRO2' biomassRxn='Biomass' lacRxn='EX_lac__L_e' glcRxn='EX_glc__D_e' glnRxn='EX_gln__L_e' gluRxn='EX_gluOUT__L_e' lcellLines=['MCF102A','MDAMB231','SKBR3','MCF7','MDAMB361']
```

## Evaluation signals

- Five valid SBML XML files (ENGRO2_MCF102A.xml, ENGRO2_MDAMB231.xml, ENGRO2_SKBR3.xml, ENGRO2_MCF7.xml, ENGRO2_MDAMB361.xml) are produced and each parses as an SBML model.
- Each output file name matches modelId + '_' + cellLineName + '.xml' for every cell line in lcellLines.
- Imposed bounds for nutrient exchange reactions equal the cellLine_LB / cellLine_UB values listed in medium.csv.
- Solution is feasible under GLPK given the combined nutrient-availability, YSI-ratio and RAS constraints (non-empty feasible flux region).

## Limitations

- If imposeMedium is set to 'N', nutrient-availability constraints are omitted and the model may produce fluxes inconsistent with the actual medium composition.
- The medium file must match the model's reaction IDs ('Rxn') exactly; mismatched identifiers will silently drop constraints.
- Only the five default cell lines (MCF102A, MDAMB231, SKBR3, MCF7, MDAMB361) are preconfigured; other samples require editing lcellLines and the medium/YSI files.

## Evidence

- [readme] mediumFileName: the input nutrients availability dataset. This file consists of the following columns: "Rxn" listing nutrients uptaken within the model; two columns for each cell line listing for each nutrient the corresponding lower bound (cell line + _LB) and upper bound (cell line + _UB).: "mediumFileName: the input nutrients availability dataset. This file consists of the following columns: "Rxn" listing nutrients uptaken within the model; two columns for each cell line listing for"
- [readme] imposeMedium: 'Y' (yes) or 'N' (no) according to whether nutrients availability constraints have to be integrated. Default value: 'Y'.: "imposeMedium: 'Y' (yes) or 'N' (no) according to whether nutrients availability constraints have to be integrated. Default value: 'Y'."
- [methods] Constrain reaction bounds proportionally to the normalized RAS scores of each cell line so that transcriptional regulation is translated into flux bounds via constraint-based modeling (GLPK solver).: [evidence span withheld — anchor status: not_found]
- [readme] Output:
  * A SBML model for each input cell line named as modelId + '_' + cellLineName + '.xml': "Output:
  * A SBML model for each input cell line named as modelId + '_' + cellLineName + '.xml'"
- [readme] The Pipeline was tested using GLPK solver (https://www.gnu.org/software/glpk/).: "The Pipeline was tested using GLPK solver (https://www.gnu.org/software/glpk/)."
