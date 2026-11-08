---
name: spearman-rank-correlation-analysis
description: Use when a per-cell matrix (e.g. MORPHnMOL.csv from a SpaceM scAnalysis
  run) contains both normalized lipid intensities and microscopy-derived fluorescence
  such as LD540, and the goal is to find lipids positively or negatively correlated
  with that readout across single cells.
when_to_use_negative:
- Input is already aggregate/bulk lipid data rather than a single-cell spatio-molecular
  matrix — correlations over per-cell distributions are the point of this skill
- The intensity distributions are clearly normal and a linear (Pearson) correlation
  is sufficient — Pearson would be an alternative here
- The matrix has not yet been produced by the SpaceM pipeline (no MORPHnMOL.csv in
  the scAnalysis sub-folder); run SpaceM first
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SpaceM
  role: Produces the per-cell spatio-molecular matrix (MORPHnMOL.csv) that combines
    normalized lipid intensities with microscopy-derived phenotypic properties such
    as LD540 fluorescence, which is the input for the Spearman correlations
  repo: https://github.com/alexandrovteam/SpaceM
- name: Python
  role: Language in which the spatio-molecular matrix is loaded (pandas) and Spearman
    correlations are computed (e.g. scipy.stats.spearmanr) on lipid-vs-LD540 column
    pairs
- name: SCANPY
  role: Single-cell analysis ecosystem used for downstream processing of the spatio-molecular
    matrices in which the per-cell correlations are computed
  repo: https://github.com/alexandrovteam/SpaceM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:38b364c66ba6ab4e8566f9c9fc182823e662a883eba697746105f8a5ecc195c4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/alexandrovteam/SpaceM
  tools:
  - Python
  - SCANPY
  - SpaceM
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# spearman-rank-correlation-analysis

## Summary

Computes non-parametric Spearman rank correlations between per-cell lipid species intensities and microscopy-derived phenotypic readouts (e.g. LD540 fluorescence) from a SpaceM spatio-molecular matrix, to identify lipids that mark a steatotic subpopulation of dHepaRG hepatocytes.

## When to use

Use when a per-cell matrix (e.g. MORPHnMOL.csv from a SpaceM scAnalysis run) contains both normalized lipid intensities and microscopy-derived fluorescence such as LD540, and the goal is to find lipids positively or negatively correlated with that readout across single cells.

## When NOT to use

- Input is already aggregate/bulk lipid data rather than a single-cell spatio-molecular matrix — correlations over per-cell distributions are the point of this skill
- The intensity distributions are clearly normal and a linear (Pearson) correlation is sufficient — Pearson would be an alternative here
- The matrix has not yet been produced by the SpaceM pipeline (no MORPHnMOL.csv in the scAnalysis sub-folder); run SpaceM first

## Inputs

- Final spatio-molecular matrix MORPHnMOL.csv (from the scAnalysis sub-folder of the SpaceM Analysis folder), with one row per cell, normalized lipid intensities, and microscopy-derived per-cell fluorescence (e.g. LD540)
- List of lipid species columns to correlate (e.g. TG(48:0), LPE(21:1), LPC(18:1)) and the name of the fluorescence column to correlate against

## Outputs

- lipid_ld540_correlations.csv with Spearman rho and p-value for each lipid–fluorescence pair
- Interpretation of lipids as positive or negative markers of the steatotic (LD-high) hepatocyte subpopulation

## How to apply

Load the final spatio-molecular matrix MORPHnMOL.csv from the scAnalysis sub-folder of the Analysis folder using Python/pandas; each row is a single cell with normalized lipid intensities and microscopy-derived phenotypic properties such as fluorescence intensities. Extract the lipid intensity columns of interest (e.g. the triglyceride TG(48:0) and the isomeric lysophospholipids LPE(21:1)/LPC(18:1)) together with the per-cell LD540 fluorescence intensity column. Because single-cell metabolomics intensity distributions are non-normal, compute Spearman rank correlations (e.g. scipy.stats.spearmanr) rather than Pearson. Record the Spearman rho and p-value for each lipid–fluorescence pair and write the results to lipid_ld540_correlations.csv; validation is that TG(48:0) recovers a positive correlation (Spearman r = 0.38) and LPE(21:1)/LPC(18:1) recovers an anti-correlation (Spearman r = -0.23) against LD540 in the fatty-acid-stimulated dHepaRG NAFLD model.

## Related tools

- **SpaceM** (Produces the per-cell spatio-molecular matrix (MORPHnMOL.csv) that combines normalized lipid intensities with microscopy-derived phenotypic properties such as LD540 fluorescence, which is the input for the Spearman correlations) — https://github.com/alexandrovteam/SpaceM
- **Python** (Language in which the spatio-molecular matrix is loaded (pandas) and Spearman correlations are computed (e.g. scipy.stats.spearmanr) on lipid-vs-LD540 column pairs)
- **SCANPY** (Single-cell analysis ecosystem used for downstream processing of the spatio-molecular matrices in which the per-cell correlations are computed) — https://github.com/alexandrovteam/SpaceM

## Examples

```
import pandas as pd; from scipy.stats import spearmanr; df = pd.read_csv('Analysis/scAnalysis/MORPHnMOL.csv'); rho, p = spearmanr(df['TG(48:0)'], df['LD540'])
```

## Evaluation signals

- Reproduction check: TG(48:0) must come out positively correlated with LD540 with Spearman r = 0.38
- Reproduction check: LPE(21:1)/LPC(18:1) must come out anti-correlated with LD540 with Spearman r = -0.23
- Output schema check: lipid_ld540_correlations.csv contains a rho and p-value for each lipid–fluorescence pair
- Input check: MORPHnMOL.csv exists in the scAnalysis sub-folder of the Analysis folder, confirming the SpaceM spatio-molecular matrix (not intermediate tile data) was used
- Method check: correlations were computed with a rank-based (Spearman) statistic, appropriate for the non-normal single-cell intensity distributions, rather than Pearson

## Limitations

- Correlation against LD540 identifies lipids associated with the steatotic subpopulation but does not itself establish causality; the same study notes that increased sphingolipids upon NASH lack a mechanistic explanation
- Bimodality of intermediate cells could either reflect a clustering artifact or biological heterogeneity of LD compositions, so state-level interpretation of correlated lipids should be made cautiously
- Co-sampling of neighboring cells by the same ablation mark only minimally affects cell type prediction, but excluding co-sampling ablation marks reduced statistical significance of metabolic state markers, likely due to lower numbers of cells

## Evidence

- [results] lipids that mark the steatotic subpopulation: [evidence span withheld — anchor status: not_found]
