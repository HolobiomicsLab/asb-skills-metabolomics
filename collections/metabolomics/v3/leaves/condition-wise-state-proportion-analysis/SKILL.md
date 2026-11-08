---
name: condition-wise-state-proportion-analysis
description: Use when single cells have already been clustered into metabolic states
  and you need per-condition state fractions to test whether a perturbation (e.g.
  IL-17A inflammation, TPCA-1 treatment) changes the composition of a cell population.
when_to_use_negative:
- Input is raw MALDI-imaging data (.RAW, .UDP, .imzML, .ibd) or pre-/post-MALDI microscopy
  tiles — the SpaceM pipeline (segmentation, co-registration, matrix generation) must
  run first to produce MORPHnMOL.csv
- The biological question concerns cell-type identification from metabolomes (91.3%
  prediction accuracy) rather than within-population state heterogeneity
- You need absolute abundances or marker-metabolite statistics rather than relative
  state composition per condition
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2229
tools:
- name: SpaceM
  role: Produces the per-cell spatio-molecular matrix (MORPHnMOL.csv) that is the
    input for clustering and state-fraction tabulation
  repo: https://github.com/alexandrovteam/SpaceM
- name: SCANPY
  role: Nearest-neighbor graph construction, Leiden clustering of metabolic states,
    and UMAP/PAGA embedding before proportion tabulation
- name: Python
  role: pandas-based grouping and computation of per-condition state fractions, saved
    as metabolic_state_distributions.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:fe548720f5e824a7b4ddd10397f0227af79064eab68fc69d24d8d02cb0c0e2a1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/alexandrovteam/SpaceM
  tools:
  - Python
  - SCANPY
  - SpaceM
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# condition-wise-state-proportion-analysis

## Summary

Quantifies the fraction of single cells assigned to each unsupervised metabolic state (homeostatic, intermediate, steatotic) per culture condition (+FA, +FA+IL17A, +FA+IL17A+TPCA1) in SpaceM spatio-molecular data, revealing how inflammation and NF-κB inhibition shift hepatocyte state balances.

## When to use

Use when single cells have already been clustered into metabolic states and you need per-condition state fractions to test whether a perturbation (e.g. IL-17A inflammation, TPCA-1 treatment) changes the composition of a cell population.

## When NOT to use

- Input is raw MALDI-imaging data (.RAW, .UDP, .imzML, .ibd) or pre-/post-MALDI microscopy tiles — the SpaceM pipeline (segmentation, co-registration, matrix generation) must run first to produce MORPHnMOL.csv
- The biological question concerns cell-type identification from metabolomes (91.3% prediction accuracy) rather than within-population state heterogeneity
- You need absolute abundances or marker-metabolite statistics rather than relative state composition per condition

## Inputs

- MORPHnMOL.csv spatio-molecular matrix from the scAnalysis sub-folder (normalized metabolic profiles plus microscopy-derived phenotypic features per cell, ~29,738 hepatocytes)
- Condition labels for +FA, +FA+IL17A, and +FA+IL17A+TPCA1 culture conditions
- Leiden cluster/state assignments from the Scanpy clustering of metabolic profiles

## Outputs

- metabolic_state_distributions.csv — fraction of cells in each metabolic state per culture condition
- UMAP/PAGA embeddings visualizing homeostatic, intermediate, and steatotic states
- Comparison against reported balances (75.2% / 24.3% / 93% steatotic)

## How to apply

Load the spatio-molecular matrix MORPHnMOL.csv from the scAnalysis sub-folder with Python/pandas, retaining normalized metabolic profiles for the 29,738 hepatocytes across conditions. Preprocess and build a nearest-neighbor graph over the single-cell metabolic profiles with Scanpy (scanpy.pp.neighbors), then cluster with Leiden to resolve the homeostatic, intermediate, and steatotic states. Embed with UMAP and connect states with PAGA (scanpy.tl.umap, scanpy.tl.paga) to confirm the state structure visually. Finally, group cells by condition and cluster label with pandas to compute the fraction of cells in each state, reproducing the reported balances (+FA: 75.2% intermediate / 24.3% steatotic; +FA+IL17A: ~93% steatotic; +FA+IL17A+TPCA1: 75.2% intermediate) and writing metabolic_state_distributions.csv. Rationale: state proportions are the readout that links single-cell clustering to the biological question of whether IL-17A pushes hepatocytes toward the steatotic state and whether NF-κB inhibition reverses this.

## Related tools

- **SpaceM** (Produces the per-cell spatio-molecular matrix (MORPHnMOL.csv) that is the input for clustering and state-fraction tabulation) — https://github.com/alexandrovteam/SpaceM
- **SCANPY** (Nearest-neighbor graph construction, Leiden clustering of metabolic states, and UMAP/PAGA embedding before proportion tabulation)
- **Python** (pandas-based grouping and computation of per-condition state fractions, saved as metabolic_state_distributions.csv)

## Examples

```
import pandas as pd, scanpy as sc; adata = sc.read_csv('scAnalysis/MORPHnMOL.csv'); sc.pp.neighbors(adata); sc.tl.leiden(adata, resolution=0.5); sc.tl.paga(adata); sc.tl.umap(adata); adata.obs.groupby(['condition','leiden']).size().unstack(fill_value=0).div(adata.obs.condition.value_counts(), axis=0).to_csv('metabolic_state_distributions.csv')
```

## Evaluation signals

- +FA condition reproduces the reported 75.2% intermediate / 24.3% steatotic split (fractions should sum to ~100%)
- +FA+IL17A condition reproduces the reported ~93% steatotic fraction
- +FA+IL17A+TPCA1 condition reproduces the reported 75.2% intermediate fraction, showing reversal of the IL-17A effect
- PAGA/UMAP embedding shows three connected states consistent with homeostatic, intermediate, and steatotic labels
- Excluding co-sampling ablation marks should not change the state balance per condition (robustness check from the article)

## Limitations

- Bimodality of intermediate cells could either reflect a clustering artifact or biological heterogeneity of LD compositions, making the intermediate/steatotic boundary potentially ambiguous
- Excluding co-sampling ablation marks reduces the number of cells and statistical significance of metabolic state markers, though the state balance itself is unaffected
- SpaceM's stitching implementation is optimized for the Nikon Ti-E microscope output format, constraining which raw datasets can feed the analysis

## Evidence

- [results] unsupervised clustering resolved an intermediate and a steatotic metabolic state, with 75.2% and 24.3% of cells residing in each state: [evidence span withheld — anchor status: not_found]
