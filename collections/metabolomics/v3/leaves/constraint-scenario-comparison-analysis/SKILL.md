---
name: constraint-scenario-comparison-analysis
description: Use when randomSampling CSVs exist for multiple cell-relative models
  under several constraint configurations and the goal is to test whether combined
  constraints (medium + YSI + RAS) yield better inter-model separation of sampled
  feasible flux distributions than single-constraint scenarios.
when_to_use_negative:
- The sampled flux distribution CSVs from randomSampling do not yet exist for the
  cell-relative models (run Step 4 rasIntegration and Step 6 randomSampling first).
- The goal is to classify which fluxes are regulated at the metabolic vs gene expression
  level (use the intersection of transcriptomics- and metabolomics-based outputs,
  i.e. concordanceAnalysis, not embedding comparison).
- Only two cell lines need pairwise flux-distribution comparison; the mannWhitneyUTest
  pipeline step already returns per-pair test outputs without embedding.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: INTEGRATE
  role: Pipeline whose randomSampling step produces the sampled flux distribution
    CSVs per cell-relative model that this skill compares across constraint scenarios
  repo: https://github.com/qLSLab/integrate
- name: Python
  role: Runtime used to run t-SNE dimensionality reduction (scikit-learn) on the sampled
    flux vectors and to write the embedding CSV
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1009337
    title: 'INTEGRATE: Model-based multi-omics data integration to characterize multi-level
      metabolic regulation'
  source_content_hash: sha256:207187a998166e76d2916688f396248120996252cec3c4dce48f476e67f8f401
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qLSLab/integrate
  tools:
  - INTEGRATE
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# constraint-scenario-comparison-analysis

## Summary

Compares t-SNE 2D embeddings of flux distributions sampled from five cell-relative ENGRO2 models (MCF102A, SKBR3, MCF7, MDAMB231, MDAMB361) across constraint scenarios (nutrient availability alone, extracellular flux ratios alone, transcriptomics-derived constraints alone, all combined) to quantify which scenario best separates the models.

## When to use

Use when randomSampling CSVs exist for multiple cell-relative models under several constraint configurations and the goal is to test whether combined constraints (medium + YSI + RAS) yield better inter-model separation of sampled feasible flux distributions than single-constraint scenarios.

## When NOT to use

- The sampled flux distribution CSVs from randomSampling do not yet exist for the cell-relative models (run Step 4 rasIntegration and Step 6 randomSampling first).
- The goal is to classify which fluxes are regulated at the metabolic vs gene expression level (use the intersection of transcriptomics- and metabolomics-based outputs, i.e. concordanceAnalysis, not embedding comparison).
- Only two cell lines need pairwise flux-distribution comparison; the mannWhitneyUTest pipeline step already returns per-pair test outputs without embedding.

## Inputs

- randomSampling_' + modelId + '_nSol_' + str(nSamples) + '_' + cellLine + '_' + timeStamp + '.csv' files of sampled flux solutions for each of MCF102A, SKBR3, MCF7, MDAMB231, MDAMB361, generated per constraint scenario
- Constraint scenario labels: type 1 (nutrients availability / medium.csv) alone, type 2 (extracellular flux ratios / ysi_ratio.csv) alone, type 3 (transcriptomics-derived RAS constraints) alone, and all three combined

## Outputs

- fig3_tsne_embedding.csv containing 2D t-SNE coordinates plus per-sample model/cluster labels
- Cross-scenario comparison of inter-model separation showing combined constraints as the best-separating configuration (Fig 3D)

## How to apply

Load the sampled flux distribution CSVs named 'randomSampling_' + modelId + '_nSol_' + nSamples + '_' + cellLine + '_' + timeStamp + '.csv' for the five cell lines, one set per constraint scenario (type 1 alone, type 2 alone, type 3 alone, all combined). For each scenario, run t-SNE dimensionality reduction in Python (scikit-learn) to embed each model's sampled solutions in 2D, assigning every embedded sample to its source cell-line model. Quantify inter-model separation as the contiguity of each model's points in the 2D embedding, then compare separation across the four scenarios. The expected result is that the all-constraints-combined configuration shows each cell line's sampled flux solutions forming a spatially separated cluster, consistent with Fig 3D of the article. Write the 2D t-SNE coordinates with per-sample model/cluster labels to fig3_tsne_embedding.csv to regenerate the Fig 3 layout.

## Related tools

- **INTEGRATE** (Pipeline whose randomSampling step produces the sampled flux distribution CSVs per cell-relative model that this skill compares across constraint scenarios) — https://github.com/qLSLab/integrate
- **Python** (Runtime used to run t-SNE dimensionality reduction (scikit-learn) on the sampled flux vectors and to write the embedding CSV)

## Examples

```
from sklearn.manifold import TSNE; emb = TSNE(n_components=2).fit_transform(X)
```

## Evaluation signals

- In the combined-constraints embedding, each cell line's sampled flux solutions form a spatially separated cluster from the other four cell lines, matching the reported best inter-model separation (Fig 3D).
- Separation quantified as contiguity of each model's points in 2D is higher for the all-constraints-combined scenario than for any single-constraint scenario.
- fig3_tsne_embedding.csv contains 2D coordinates plus a model/cluster label for every sampled solution across the four scenarios.
- Input files match the pattern 'randomSampling_' + modelId + '_nSol_' + str(nSamples) + '_' + cellLine + '_' + timeStamp + '.csv' and cover all five default cell lines ['MCF102A', 'MDAMB231', 'SKBR3', 'MCF7', 'MDAMB361'].

## Limitations

- Sampling quality depends on randomSampling parameters (nSamples, nBatches, epsilon lower bound of 1e-4 on the biomass synthesis reaction), which control the feasible flux region explored.
- t-SNE embeddings are stochastic, so separation must be judged by cluster contiguity rather than exact coordinate reproduction.
- Results are specific to the five breast cell-line models built on ENGRO2 with the default reaction IDs (Biomass, EX_lac__L_e, EX_glc__D_e, EX_gln__L_e, EX_gluOUT__L_e) and replicas ['_A','_B'].

## Evidence

- [other] Yes: the simultaneous application of the three constraint types (Fig 3D) better separates the flux distributions sampled from each of the five cell-relative models: [evidence span withheld — anchor status: not_found]
- [readme] File named 'randomSampling_' + modelId + '_nSol_' + str(nSamples) + '_' + cellLine + '_' + timeStamp + '.csv' containing the nSamples sampled solutions from each input sample model: "File named 'randomSampling_' + modelId + '_nSol_' + str(nSamples) + '_' + cellLine + '_' + timeStamp + '.csv' containing the nSamples sampled solutions from each input sample model"
- [readme] imposeRasConstraints: 'Y' (yes) or 'N' (no) according to whether transcriptomics derived constraints have to be integrated. Default value: 'Y'.: "imposeRasConstraints: 'Y' (yes) or 'N' (no) according to whether transcriptomics derived constraints have to be integrated. Default value: 'Y'."
- [intro] INTEGRATE sets constraints also on selected extracellular fluxes, according to exo-metabolomics data.: "INTEGRATE sets constraints also on selected extracellular fluxes, according to exo-metabolomics data."
- [readme] Aim: sample the feasible flux region of each cell relative model: "Aim: sample the feasible flux region of each cell relative model"
