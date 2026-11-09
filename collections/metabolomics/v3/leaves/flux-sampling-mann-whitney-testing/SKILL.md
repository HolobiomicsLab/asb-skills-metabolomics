---
name: flux-sampling-mann-whitney-testing
description: Use when you need a nonparametric, distribution-free classification of
  per-reaction flux differences between two biological conditions, starting from constraint-based
  models whose feasible regions have been constrained by RAS scores, extracellular
  flux ratios, and nutrient-availability bounds.
when_to_use_negative:
- When the difference you are testing is on transcriptomics-derived RAS scores rather
  than sampled fluxes — INTEGRATE uses a parametric t-test (rasTtest) for that layer,
  not the Mann-Whitney U test
- When the input model has not been constrained by RAS, extracellular flux ratios,
  and medium bounds — sampling the unconstrained feasible region would not reflect
  the cell-relative flux states the pipeline is designed to compare
- When you lack enough sampled solutions per condition for the Mann-Whitney U test
  to be informative (e.g. nSamples too small relative to the feasible-region complexity)
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2259
tools:
- name: INTEGRATE
  role: Pipeline that packages randomSampling and mannWhitneyUTest as Steps 6–7 of
    the FFD-flux classification workflow
  repo: https://github.com/qLSLab/integrate
- name: randomSampling
  role: Python script that samples the feasible flux region of each cell-relative
    model in nSamples-per-nBatches fashion with epsilon = 1e-4 lower bound on Biomass
  repo: https://github.com/qLSLab/integrate
- name: mannWhitneyUTest
  role: Python script that runs the pairwise nonparametric test on sampled fluxes
    for every cell-line pair
  repo: https://github.com/qLSLab/integrate
- name: rasIntegration
  role: Upstream step that builds the cell-relative models whose constrained feasible
    regions are then sampled
  repo: https://github.com/qLSLab/integrate
- name: GLPK
  role: Solver used by the pipeline when the models were tested
  repo: https://www.gnu.org/software/glpk/
provenance:
  source_task_ids:
  - task_005
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1009337
    title: 'INTEGRATE: Model-based multi-omics data integration to characterize multi-level
      metabolic regulation'
  source_content_hash: sha256:dff75e55d2b4f5cb729eebd3f2387ce14ff329f97c43a1ae003954ceebb62bb1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qLSLab/integrate
  tools:
  - GLPK
  - INTEGRATE
  - MATLAB
  - Python
  - mannWhitneyUTest
  - randomSampling
  - rasIntegration
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# flux-sampling-mann-whitney-testing

## Summary

Randomly sample the feasible flux region of each cell-relative constraint-based model and run a pairwise Mann-Whitney U test on the sampled flux distributions to classify whether each reaction's flux differs between cell lines (the FFD / flux fold-difference layer of the INTEGRATE pipeline).

## When to use

Use when you need a nonparametric, distribution-free classification of per-reaction flux differences between two biological conditions, starting from constraint-based models whose feasible regions have been constrained by RAS scores, extracellular flux ratios, and nutrient-availability bounds.

## When NOT to use

- When the difference you are testing is on transcriptomics-derived RAS scores rather than sampled fluxes — INTEGRATE uses a parametric t-test (rasTtest) for that layer, not the Mann-Whitney U test
- When the input model has not been constrained by RAS, extracellular flux ratios, and medium bounds — sampling the unconstrained feasible region would not reflect the cell-relative flux states the pipeline is designed to compare
- When you lack enough sampled solutions per condition for the Mann-Whitney U test to be informative (e.g. nSamples too small relative to the feasible-region complexity)

## Inputs

- Irreversible-format mat files of the cell-relative models (e.g. ENGRO2_MCF102A_wIrrRxns) exported from SBML after Step 5
- ysi_ratio.csv: extracellular flux ratio dataset with a 'Ratio' column plus per-cell-line/per-replica values
- medium.csv: nutrient availability dataset with 'Rxn' column plus per-cell-line _LB/_UB columns
- ENGRO2_wNormalizedRAS.csv: normalized RAS scores per reaction and cell line (from Step 3)
- Parameters: nSamples and nBatches for randomSampling; timestamp and nSamples for mannWhitneyUTest; lcellLines ['MCF102A','MDAMB231','SKBR3','MCF7','MDAMB361']; modelId (default 'ENGRO2'); biomassRxn 'Biomass'; lacRxn EX_lac__L_e; glcRxn EX_glc__D_e; glnRxn EX_gln__L_e; gluRxn EX_gluOUT__L_e; lReplicas ['_A','_B']; epsilon = 1e-4

## Outputs

- Per-cell-line sampled flux solution files named 'randomSampling_' + modelId + '_nSol_' + str(nSamples) + '_' + cellLine + '_' + timeStamp + '.csv' containing the nSamples sampled solutions
- For each of the 10 cell-line pairs (c_1 vs c_2), a Mann-Whitney U test output file (per-reaction significance of flux difference)
- Downstream FFD (flux fold-difference) classifications per reaction per pair, consumed by concordanceAnalysis for RPS-vs-FFD and RPS-vs-RAS Cohen's kappa scoring

## How to apply

After building cell-relative SBML models with rasIntegration (imposing ysi_ratio.csv flux ratios, medium.csv nutrient bounds, and transcriptomics-derived RAS constraints with all flags 'Y', biomass reaction 'Biomass', replicas '_A'/'_B'), convert each model to a mat file and switch it to irreversible format via the MATLAB function (Step 5). Run randomSampling (python pipeline/randomSampling.py nSamples nBatches) to draw sampled steady-state flux solutions in batches from each model's feasible region, imposing a lower bound epsilon = 1e-4 on the biomass reaction so all solutions are productive. Then run mannWhitneyUTest (python pipeline/mannWhitneyUTest.py timeStampInput nSamples) on every cell-line pair — 10 pairs for the 5 breast cell lines MCF102A, MDAMB231, SKBR3, MCF7, MDAMB361 — producing, for each pair of input cell lines c_1 and c_2, a file with the Mann-Whitney U test output. The resulting per-reaction significance calls feed the FFD (flux fold-difference) layer that concordanceAnalysis later compares against RPS and RAS classifications using linearly weighted Cohen's kappa. Rationale: fluxes sampled under the constrained feasible region are non-normal and non-linear in scale, so the nonparametric Mann-Whitney U test is used instead of a t-test (which is reserved for the RAS score layer).

## Related tools

- **INTEGRATE** (Pipeline that packages randomSampling and mannWhitneyUTest as Steps 6–7 of the FFD-flux classification workflow) — https://github.com/qLSLab/integrate
- **randomSampling** (Python script that samples the feasible flux region of each cell-relative model in nSamples-per-nBatches fashion with epsilon = 1e-4 lower bound on Biomass) — https://github.com/qLSLab/integrate
- **mannWhitneyUTest** (Python script that runs the pairwise nonparametric test on sampled fluxes for every cell-line pair) — https://github.com/qLSLab/integrate
- **rasIntegration** (Upstream step that builds the cell-relative models whose constrained feasible regions are then sampled) — https://github.com/qLSLab/integrate
- **GLPK** (Solver used by the pipeline when the models were tested) — https://www.gnu.org/software/glpk/

## Examples

```
python pipeline/randomSampling.py 1000 10 && python pipeline/mannWhitneyUTest.py <timeStampInput> 1000
```

## Evaluation signals

- File naming check: sampled solutions appear as 'randomSampling_<modelId>_nSol_<nSamples>_<cellLine>_<timeStamp>.csv' and Mann-Whitney outputs exist for all 10 pairwise combinations of the 5 cell lines
- Biomass floor check: every sampled solution satisfies flux(Biomass) >= epsilon = 1e-4, i.e. no null/degenerate solutions
- Reproduction check: the FFD classifications derived from these tests reproduce the S4/S5 concordance tables when fed into concordanceAnalysis with linearly weighted Cohen's kappa, with significant RPS-vs-FFD concordance (FDR-adjusted) reported for reactions ACONT, ASPTA (forward), RPI, and RPE
- Recon3D sensitivity check: re-running the sampling + Mann-Whitney + concordance chain on Recon3D restricted to ENGRO2-active exchange reactions keeps RPE classified as transcriptionally and metabolically regulated in both models
- Direction/annotation consistency: each sampled flux sign agrees with the irreversible model's reaction directionality after the Step 5 reversible-to-irreversible conversion

## Limitations

- Sampling depends on the quality of the upstream cell-relative models; if rasIntegration flags (imposeYSI, imposeMedium, imposeRasConstraints) are misconfigured or medium/ysi files are noisy, the sampled feasible region and hence FFD calls inherit those errors
- The pipeline was tested with GLPK solver; other solvers may behave differently on the same feasible region
- Mann-Whitney U outputs are per-pair (c_1 vs c_2), so classification is pairwise and must be aggregated across 10 pairs to obtain a per-reaction consensus FFD classification
- Results are sensitive to the input metabolic model, as shown by the Recon3D sensitivity analysis where concordance categories were re-derived but only partially overlapping with ENGRO2

## Evidence

- [methods] randomSampling samples the feasible flux region of each cell-relative model with epsilon = 1e-4 lower bound on biomass, then mannWhitneyUTest computes pairwise nonparametric flux-difference tests: "Step 6: randomSampling * Aim: sample the feasible flux region of each cell relative model"
- [methods] Mann-Whitney U test output files are produced for each pair of input cell lines c_1 and c_2: "Step 7: mannWhitneyUTest * Aim: compute the Mann-Whitney U test"
- [readme] Random sampling step imposes a lower bound epsilon on the biomass reaction to keep sampled solutions productive: "epsilon: lower bound imposed on the biomass synthesis reaction. Default value: 1e-4."
- [readme] Mann-Whitney U test on sampled fluxes distinguishes cell-line pairs; the nonparametric choice contrasts with the t-test used on RAS scores: "Step 7: mannWhitneyUTest
* Aim: compute the Mann-Whitney U test
* Usage: `python pipeline/mannWhitneyUTest.py timeStampInput nSamples`"
- [supplementary] In the source cards the pipeline is described as sampling steady-state fluxes and comparing them pairwise with the Mann-Whitney U test to derive FFD classifications: [evidence span withheld — anchor status: not_found]
- [results] After Benjamini-Hochberg FDR correction, concordance between RPSs and FFDs was statistically significant for reactions ACONT, ASPTA (forward), RPI, and RPE: [evidence span withheld — anchor status: not_found]
- [readme] The pipeline was tested using GLPK solver, which is the solver context for the sampling and testing steps: "The Pipeline was tested using GLPK solver (https://www.gnu.org/software/glpk/)."
- [readme] Model conversion to irreversible format is required before sampling, so reaction directionality is respected: "Each model needs to be converted to a mat file in order to exploit the MATLAB function to convert model from the reversible into the irreversible format"
