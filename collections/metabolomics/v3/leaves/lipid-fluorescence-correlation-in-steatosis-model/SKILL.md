---
name: lipid-fluorescence-correlation-in-steatosis-model
description: Use when a per-cell SpaceM matrix (MORPHnMOL.csv) containing normalized
  lipid intensities plus microscopy-derived fluorescence such as LD540 must be screened
  for lipids that positively or negatively mark a steatotic (lipid-droplet-rich) cell
  subpopulation.
when_to_use_negative:
- Input is already a feature table without per-cell rows (e.g. bulk lipidomics or
  averaged well-level measurements) — single-cell Spearman correlation is not applicable
- The lipid of interest and the fluorescence readout are isomerically ambiguous and
  downstream interpretation requires a definite structural assignment (LPE(21:1) and
  LPC(18:1) are isomeric and cannot be distinguished here)
- Intensity distributions are confirmed normal and a parametric Pearson correlation
  with the fluorescence channel is explicitly desired
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Python
  role: Loads MORPHnMOL.csv with pandas and computes Spearman correlations via scipy.stats.spearmanr
- name: SCANPY
  role: Single-cell analysis ecosystem within which the per-cell correlation analysis
    is run
- name: SpaceM
  role: Upstream pipeline that generates the per-cell spatio-molecular matrix MORPHnMOL.csv
    consumed by this correlation step
  repo: github.com/alexandrovteam/SpaceM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:6df6234a197243ab364dffcfba971ab237d537677bc4f1a827c8b0acf1cf5714
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - SCANPY
  - SpaceM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# lipid-fluorescence-correlation-in-steatosis-model

## Summary

Computes non-parametric single-cell correlations between specific lipid species (e.g. TG(48:0), LPE(21:1)/LPC(18:1)) and microscopy-derived LD540 neutral-lipid fluorescence from a SpaceM spatio-molecular matrix, to identify positive and negative markers of the steatotic subpopulation in fatty-acid-stimulated dHepaRG hepatocytes.

## When to use

Use when a per-cell SpaceM matrix (MORPHnMOL.csv) containing normalized lipid intensities plus microscopy-derived fluorescence such as LD540 must be screened for lipids that positively or negatively mark a steatotic (lipid-droplet-rich) cell subpopulation.

## When NOT to use

- Input is already a feature table without per-cell rows (e.g. bulk lipidomics or averaged well-level measurements) — single-cell Spearman correlation is not applicable
- The lipid of interest and the fluorescence readout are isomerically ambiguous and downstream interpretation requires a definite structural assignment (LPE(21:1) and LPC(18:1) are isomeric and cannot be distinguished here)
- Intensity distributions are confirmed normal and a parametric Pearson correlation with the fluorescence channel is explicitly desired

## Inputs

- Spatio-molecular matrix MORPHnMOL.csv from the SpaceM Analysis/scAnalysis sub-folder (one row per cell; normalized lipid intensities plus microscopy-derived LD540 fluorescence intensities)
- Column of interest: TG(48:0) neutral lipid intensity
- Columns of interest: LPE(21:1) and/or LPC(18:1) lysophospholipid intensities
- Per-cell LD540 fluorescence intensity column

## Outputs

- lipid_ld540_correlations.csv with Spearman rho and p-value for each lipid–LD540 fluorescence pair
- Positive steatotic-subpopulation marker call (expected TG(48:0), Spearman r = 0.38)
- Negative steatotic-subpopulation marker call (expected LPE(21:1)/LPC(18:1), Spearman r = -0.23)

## How to apply

Load the final spatio-molecular matrix MORPHnMOL.csv from the scAnalysis sub-folder of the SpaceM Analysis folder with Python/pandas; each row is a single cell with normalized lipid intensities and per-cell LD540 fluorescence. Extract the columns for the neutral-lipid species TG(48:0) and the lysophospholipids LPE(21:1) and LPC(18:1) together with the LD540 fluorescence column. Because single-cell metabolomics intensity distributions are non-normal, compute Spearman rank correlations (scipy.stats.spearmanr in the Python/Scanpy ecosystem) between each lipid intensity and LD540 across all single cells. Record Spearman rho and p-value per lipid–fluorescence pair and write the results to lipid_ld540_correlations.csv.

## Related tools

- **Python** (Loads MORPHnMOL.csv with pandas and computes Spearman correlations via scipy.stats.spearmanr)
- **SCANPY** (Single-cell analysis ecosystem within which the per-cell correlation analysis is run)
- **SpaceM** (Upstream pipeline that generates the per-cell spatio-molecular matrix MORPHnMOL.csv consumed by this correlation step) — github.com/alexandrovteam/SpaceM

## Examples

```
from scipy.stats import spearmanr; rho, p = spearmanr(df['TG(48:0)'], df['LD540'])  # df = pd.read_csv('MORPHnMOL.csv')
```

## Evaluation signals

- Recovered Spearman rho for TG(48:0) vs LD540 equals ~+0.38 (reported positive marker of the steatotic subpopulation)
- Recovered Spearman rho for LPE(21:1)/LPC(18:1) vs LD540 equals ~-0.23 (reported anti-correlated marker of the bigger subpopulation)
- Output file lipid_ld540_correlations.csv contains both a rho and a p-value for every lipid–fluorescence pair tested
- Correlations are Spearman (non-parametric), not Pearson, consistent with non-normal single-cell intensity distributions
- Row count of the analysis equals the number of cells in MORPHnMOL.csv (correlation over all single cells, not per-well aggregates)

## Limitations

- LPE(21:1) and LPC(18:1) are isomeric lipids that cannot be structurally distinguished, so the negative marker is reported as LPE(21:1) or LPC(18:1)
- MALDI co-sampling of neighboring cells by the same ablation mark can affect per-cell intensity assignments, although excluding co-sampled marks identified the same metabolic states while reducing statistical significance due to fewer cells
- Intermediate-state cells show bimodal lipid-droplet behavior that could reflect a clustering artifact or biological heterogeneity of LD compositions, complicating subpopulation interpretation

## Evidence
