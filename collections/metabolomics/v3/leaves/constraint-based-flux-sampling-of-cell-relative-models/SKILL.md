---
name: constraint-based-flux-sampling-of-cell-relative-models
description: Use when you have cell-relative metabolic models (built from a generic
  model plus RAS and extracellular-flux constraints) and need per-reaction, pairwise
  flux-difference classifications to feed FFD-vs-RPS concordance scoring.
when_to_use_negative:
- When the input is a single generic model without cell-specific RAS/extracellular
  constraints — the sampled flux region would not reflect cell-relative phenotypes.
- When the goal is differential expression itself rather than flux differences; use
  getRASscore/rasTtest instead.
- When metabolomics-only classification is needed (RPS) — use createMetabolicDataset/concordanceAnalysis
  without flux sampling.
edam_operation: http://edamontology.org/operation_0242
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_2258
tools:
- name: randomSampling
  role: Samples the feasible flux region of each cell-relative model, in nSamples
    × nBatches, with an epsilon lower bound on biomass
  repo: https://github.com/qLSLab/integrate
- name: mannWhitneyUTest
  role: Compares sampled flux distributions between cell-line pairs to produce FFD
    classifications
  repo: https://github.com/qLSLab/integrate
- name: MATLAB
  role: Used to convert each cell-relative model to a mat file and split reversible
    reactions into irreversible format before sampling
- name: rasIntegration
  role: Upstream step producing the cell-relative SBML models that are the sampling
    input
  repo: https://github.com/qLSLab/integrate
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1009337
    title: 'INTEGRATE: Model-based multi-omics data integration to characterize multi-level
      metabolic regulation'
  source_content_hash: sha256:fa46896feb4f9975289f6202b80015e91c1e057742964909d6fc5be76abc8e13
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
  edam_operation: http://edamontology.org/operation_0242
  edam_topics:
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5263157894736842
n_claims: 19
---

# Constraint-based flux sampling of cell-relative models

## Summary

Samples the feasible flux region of each cell-relative constraint-based metabolic model with randomSampling, then classifies every reaction's flux direction/difference between cell-line pairs via Mann-Whitney U tests to derive FFD (Flux Fold Differences) categories for INTEGRATE's concordance analysis.

## When to use

Use when you have cell-relative metabolic models (built from a generic model plus RAS and extracellular-flux constraints) and need per-reaction, pairwise flux-difference classifications to feed FFD-vs-RPS concordance scoring.

## When NOT to use

- When the input is a single generic model without cell-specific RAS/extracellular constraints — the sampled flux region would not reflect cell-relative phenotypes.
- When the goal is differential expression itself rather than flux differences; use getRASscore/rasTtest instead.
- When metabolomics-only classification is needed (RPS) — use createMetabolicDataset/concordanceAnalysis without flux sampling.

## Inputs

- Cell-relative SBML metabolic models (e.g. ENGRO2_<cellLine>.xml from rasIntegration, one per cell line such as MCF102A, MDAMB231, SKBR3, MCF7, MDAMB361)
- ysi_ratio.csv extracellular flux ratios
- medium.csv nutrient availability bounds
- Biomass reaction name and replica suffix list
- nSamples and nBatches counts

## Outputs

- Per-cell-line sampled flux distributions: randomSampling_<modelId>_nSol_<nSamples>_<cellLine>_<timeStamp>.csv
- Per-pair Mann-Whitney U test result files classifying flux differences (FFD categories) for each cell-line pair

## How to apply

First convert each cell-relative SBML model (e.g. ENGRO2_MCF102A.xml from rasIntegration) to a mat file and split it into irreversible form using the MATLAB function, since sampling operates on the irreversible model. Then run randomSampling with nSamples and nBatches arguments, imposing a lower bound epsilon (default 1e-4) on the biomass reaction to keep solutions near steady growth; outputs are per-cell-line CSVs named randomSampling_<modelId>_nSol_<nSamples>_<cellLine>_<timeStamp>.csv. Optionally impose extracellular flux-ratio and nutrient-availability constraints (ysi_ratio.csv, medium.csv) and the fixed exchange defaults for lactate, glucose, glutamine, and glutamate demand with replicas 'A'/'B'. Next run mannWhitneyUTest.py <timeStampInput> <nSamples> on each pair of cell-line models (e.g. MCF102A vs SKBR3) to classify reactions by flux difference significance, yielding FFD classes consumed by concordanceAnalysis against RAS and RPS.

## Related tools

- **randomSampling** (Samples the feasible flux region of each cell-relative model, in nSamples × nBatches, with an epsilon lower bound on biomass) — https://github.com/qLSLab/integrate
- **mannWhitneyUTest** (Compares sampled flux distributions between cell-line pairs to produce FFD classifications) — https://github.com/qLSLab/integrate
- **MATLAB** (Used to convert each cell-relative model to a mat file and split reversible reactions into irreversible format before sampling)
- **rasIntegration** (Upstream step producing the cell-relative SBML models that are the sampling input) — https://github.com/qLSLab/integrate

## Examples

```
python pipeline/randomSampling.py 100 5; python pipeline/mannWhitneyUTest.py <timeStampInput> 100
```

## Evaluation signals

- Every output file name must match the pattern randomSampling_<modelId>_nSol_<nSamples>_<cellLine>_<timeStamp>.csv
- Sampling must impose a biomass lower bound epsilon (default 1e-4) — verify via the run configuration
- Each cell-line pair must yield a Mann-Whitney U test output file covering all reactions in the irreversible model
- FFD classifications should be consumable by concordanceAnalysis alongside RAS (rasTtest) and RPS (createMetabolicDataset) outputs — cross-check reaction IDs
- Models must be irreversible and in mat format before sampling — confirm conversion succeeded (no reversible reactions remaining)

## Limitations

- Sampling quality depends on the upstream cell-relative model; if rasIntegration constraints (ysi_ratio, medium, RAS) are mis-specified, sampled fluxes will not reflect the cell phenotype
- Sampling is on the irreversible model — reversible reactions are split, so a single original reaction may appear as two sampled directions and downstream concordance must handle this
- The biomass lower bound epsilon (1e-4) forces near-growth solutions; changing it alters the sampled region and hence FFD classes

## Evidence

- [readme] randomSampling samples the feasible flux region of each cell relative model: "* Aim: sample the feasible flux region of each cell relative model"
- [readme] epsilon lower bound on biomass: "    * epsilon: lower bound imposed on the biomass synthesis reaction. Default value: 1e-4."
- [readme] output file naming pattern: "  * File named 'randomSampling_' + modelId + '_nSol_' + str(nSamples) + '_' + cellLine + '_' + timeStamp + '.csv' containing the nSamples sampled solutions from each input sample model"
- [readme] mannWhitneyUTest computes pairwise flux-difference classification: "* Aim: compute the Mann-Whitney U test"
- [readme] irreversible conversion prerequisite: "* Aim: Each model needs to be converted to a mat file in order to exploit the MATLAB function to convert model from the reversible into the irreversible format"
- [readme] input models come from rasIntegration: "  * A SBML model for each input cell line named as modelId + '_' + cellLineName + '.xml'"
- [readme] Mann-Whitney compares per cell-line pair: "  * For each pair of input cell line c_1 and c_2, a file returning the output of Mann-Whitney U test"
- [readme] usage command for sampling: "* Usage: `python pipeline/randomSampling.py nSamples nBatches`"
