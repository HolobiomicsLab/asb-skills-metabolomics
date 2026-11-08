---
name: genome-scale-metabolic-model-media-perturbation
description: Use when a constraint-based model pipeline (e.g., SARTRE shadow-price
  features) must be shown to be insensitive to medium composition — alternate carbon
  sources or limiting carbon/nitrogen/phosphorus uptake rates — relative to a glucose
  base model.
when_to_use_negative:
- The research question concerns prediction performance on other tasks (shared-MPI
  exclusion, subsystem exclusion, or permutation tests) rather than media-composition
  sensitivity.
- The target organism/model is S. cerevisiae (Yeast-GEM) and no medium-perturbation
  design for that GEM has been defined — this skill's workflow is grounded in iJO1366
  media manipulations.
- The dataset lacks a glucose base model as the reference against which alternate-media
  metrics are compared.
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_2258
tools:
- name: COBRAToolbox
  role: Manipulate the curated iJO1366 GEM in MATLAB to set alternate carbon sources
    and limiting/non-limiting C/N/P uptake bounds
  repo: https://github.com/opencobra/cobratoolbox
- name: MATLAB
  role: Environment for running COBRAToolbox GEM curation and media-condition model
    manipulation
- name: GUROBI
  role: Optimizer behind the shadow-price recomputation for each manipulated medium
    condition
- name: Python
  role: Interface for GUROBI optimization and for running the SARTRE evaluation scripts
    per medium setting
  repo: https://github.com/fayazsoleymani/SARTRE
- name: scikit-learn
  role: Provides the random forest classifier and accuracy/AUC/F1 metrics used for
    each media configuration
- name: SARTRE
  role: Overall framework whose shadow-price-based MPI prediction performance is being
    tested for media robustness
  repo: https://github.com/fayazsoleymani/SARTRE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad098
    title: Prediction of metabolite–protein interactions based on integration of machine
      learning and constraint-based modeling
  source_content_hash: sha256:38b4d6c7271785021efd9c2adec221b3a06c729425c165c7fbfc5bedae1b04f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opencobra/cobratoolbox
  tools:
  - COBRAToolbox
  - GUROBI
  - MATLAB
  - Python
  - SARTRE
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# genome-scale-metabolic-model-media-perturbation

## Summary

Robustness-testing skill for SARTRE: perturb the media composition of the curated iJO1366 GEM (alternate carbon sources acetate, fructose, glycerol, mannose, succinate, plus limiting vs non-limiting C/N/P uptake bounds), recompute shadow prices per condition, and compare random-forest classification metrics (accuracy, AUC, F1) against the glucose baseline.

## When to use

Use when a constraint-based model pipeline (e.g., SARTRE shadow-price features) must be shown to be insensitive to medium composition — alternate carbon sources or limiting carbon/nitrogen/phosphorus uptake rates — relative to a glucose base model.

## When NOT to use

- The research question concerns prediction performance on other tasks (shared-MPI exclusion, subsystem exclusion, or permutation tests) rather than media-composition sensitivity.
- The target organism/model is S. cerevisiae (Yeast-GEM) and no medium-perturbation design for that GEM has been defined — this skill's workflow is grounded in iJO1366 media manipulations.
- The dataset lacks a glucose base model as the reference against which alternate-media metrics are compared.

## Inputs

- Curated iJO1366 GEM (iJO1366.mat / ModelIrrevOpt90.mat from MATLAB/COBRAToolbox curation)
- Media-condition specifications: alternate carbon sources (acetate, fructose, glycerol, mannose, succinate) and limiting/non-limiting carbon, nitrogen, phosphorus uptake bounds
- STITCH-derived E. coli gold standard at medium confidence score (400)
- Trivial metabolite-protein pairs derived from the GEM (to be excluded)

## Outputs

- Per-condition shadow-price optimization results saved per reaction number in the opt directory
- Per-condition accuracy, AUC, and F1 metrics compared against the glucose base model (compiled e.g. in media_composition_metrics.csv)
- Manipulated model files, optimization results, and metabolite/protein features per media directory

## How to apply

Manipulate the curated iJO1366 model (produced by COBRAToolbox in MATLAB as ModelIrrevOpt90.mat) with COBRAToolbox to swap the carbon source from glucose to acetate, fructose, glycerol, mannose, or succinate, and to set limiting vs non-limiting uptake bounds for carbon, nitrogen, and phosphorus sources. For each manipulated model, rerun the GUROBI-based optimization via Python (optimization.py) to recompute per-reaction shadow prices, saved per reaction number in the opt directory. Reconstruct the evaluation dataset from the STITCH-derived E. coli gold standard at medium confidence score (400) with trivial GEM-derived MPI pairs removed, run cleaning.py preprocessing, then train and evaluate the scikit-learn random forest classifier by running evaluation.py with the setting name in media_composition. Tabulate accuracy, AUC, and F1 for every medium configuration side-by-side with the glucose base-model metrics; robustness is demonstrated by stability of these metrics across conditions.

## Related tools

- **COBRAToolbox** (Manipulate the curated iJO1366 GEM in MATLAB to set alternate carbon sources and limiting/non-limiting C/N/P uptake bounds) — https://github.com/opencobra/cobratoolbox
- **MATLAB** (Environment for running COBRAToolbox GEM curation and media-condition model manipulation)
- **GUROBI** (Optimizer behind the shadow-price recomputation for each manipulated medium condition)
- **Python** (Interface for GUROBI optimization and for running the SARTRE evaluation scripts per medium setting) — https://github.com/fayazsoleymani/SARTRE
- **scikit-learn** (Provides the random forest classifier and accuracy/AUC/F1 metrics used for each media configuration)
- **SARTRE** (Overall framework whose shadow-price-based MPI prediction performance is being tested for media robustness) — https://github.com/fayazsoleymani/SARTRE

## Examples

```
cd media_composition; python3 evaluation.py acetate
```

## Evaluation signals

- Accuracy, AUC, and F1 under each alternate carbon source (acetate, fructose, glycerol, mannose, succinate) remain stable relative to the glucose baseline, consistent with the paper's reported AUC of ~0.86 for E. coli.
- The same stability holds for limiting vs non-limiting uptake rates of carbon, nitrogen, and phosphorus (corresponding to Supplementary Tables S4–S7).
- Per-condition optimization results exist for every reaction in the opt directory and each medium setting was run with the correct setting-name argument to evaluation.py.
- Each condition's dataset is built from the STITCH E. coli gold standard at confidence score 400 with trivial GEM-derived MPI pairs removed, matching the base-model dataset construction.

## Limitations

- Robustness is demonstrated for iJO1366 (E. coli) and the specified carbon sources and nutrient limitations; it does not automatically transfer to other GEMs such as Yeast-GEM.
- GUROBI requires installation and an active license before the optimization step can be run for any medium condition.
- Archives with provided optimization results must be unzipped before the pipeline runs; the README notes 'MAKE sure to unzip archives'.

## Evidence

- [methods] methods: "We utilized models with different carbon sources as acetate, fructose, glycerol, mannose, and succinate, and compared metrics with the base model, which was in glucose media composition"
- [readme] readme-dataset: "In this part SARTRE framework is applied to a dataset, which is constructed from the GEM of iJO1366 and the gold standard from STITCH with a medium (400) confidence score."
- [readme] readme-execution: "For executing the codes, first, change current directory, then run the script with desired setting name, e.g. acetate as below"
- [methods] methods-optimization: "The optimization process has been implemented by [GUROBI](https://www.gurobi.com/) optimizer and its python interface"
