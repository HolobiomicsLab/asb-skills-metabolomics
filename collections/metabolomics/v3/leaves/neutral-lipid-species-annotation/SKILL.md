---
name: neutral-lipid-species-annotation
description: Use when per-cell lipid intensity columns from a MALDI-IMS single-cell
  metabolomics matrix (e.g. SpaceM MORPHnMOL.csv) must be annotated as candidate lipid
  species and scored against a microscopy-derived lipid droplet marker such as LD540.
when_to_use_negative:
- Input is bulk lipidomics or averaged per-well intensities rather than a per-cell
  spatio-molecular matrix — the Spearman correlation is defined over individual cells
- The goal is to resolve isomeric lipids (e.g. separating LPE(21:1) from LPC(18:1));
  single-cell MALDI-IMS annotations group such isobars and the method cannot distinguish
  them
- MALDI data have not been annotated via METASPACE, so m/z features lack lipid identities
  and no neutral-lipid species can be named
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: SpaceM
  role: Generates the per-cell spatio-molecular matrix (MORPHnMOL.csv) whose lipid
    intensity columns and microscopy-derived LD540 values are the substrate for annotation
    and correlation
  repo: github.com/alexandrovteam/SpaceM
- name: METASPACE
  role: Annotates MALDI-imaging m/z features to lipid species (isomeric groups such
    as TG(48:0), LPE(21:1)/LPC(18:1)) prior to per-cell correlation testing
- name: SCANPY
  role: Single-cell analysis framework within which per-cell lipid intensities and
    Spearman correlations against LD540 are computed
- name: Python
  role: Execution environment for loading MORPHnMOL.csv (pandas) and running correlation
    calculations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:9d17feaae2fdc65674a8957624b252156168521cd088c76547863376f8c05851
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - METASPACE
  - Python
  - SCANPY
  - SpaceM
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# neutral-lipid-species-annotation

## Summary

Annotates lipid species from per-cell MALDI imaging mass spectrometry intensities by mapping m/z features to isomeric neutral lipids (triglycerides, lysophospholipids) and testing their rank correlation with lipid-droplet readouts, as demonstrated with TG(48:0) and LPE(21:1)/LPC(18:1) in SpaceM dHepaRG steatosis data.

## When to use

Use when per-cell lipid intensity columns from a MALDI-IMS single-cell metabolomics matrix (e.g. SpaceM MORPHnMOL.csv) must be annotated as candidate lipid species and scored against a microscopy-derived lipid droplet marker such as LD540.

## When NOT to use

- Input is bulk lipidomics or averaged per-well intensities rather than a per-cell spatio-molecular matrix — the Spearman correlation is defined over individual cells
- The goal is to resolve isomeric lipids (e.g. separating LPE(21:1) from LPC(18:1)); single-cell MALDI-IMS annotations group such isobars and the method cannot distinguish them
- MALDI data have not been annotated via METASPACE, so m/z features lack lipid identities and no neutral-lipid species can be named

## Inputs

- Per-cell spatio-molecular matrix (MORPHnMOL.csv) with normalized lipid intensities and LD540 fluorescence per cell
- MALDI-imaging mass spectrometry raw data (.RAW, .UDP, .imzML, .ibd) for metabolite/lipid annotation via METASPACE
- METASPACE analysis results assigning lipid identities to m/z features

## Outputs

- Annotated list of lipid species with per-cell intensity columns (e.g. TG(48:0), LPE(21:1)/LPC(18:1))
- Spearman correlation table of each annotated lipid versus LD540 fluorescence (rho, p-value), e.g. TG(48:0) r=0.38 and LPE(21:1)/LPC(18:1) r=-0.23
- Marker assignment of lipids to LD-rich (steatotic) vs LD-low subpopulations

## How to apply

Load the final spatio-molecular matrix MORPHnMOL.csv from the scAnalysis sub-folder, where each row is a single cell carrying normalized MALDI-derived lipid intensities plus microscopy-derived phenotypic properties such as LD540 fluorescence. Use METASPACE analysis of the .imzML/.ibd MALDI-imaging data to assign lipid identities (e.g. TG(48:0), LPE(21:1), LPC(18:1)) to m/z features before per-cell extraction. Extract the candidate neutral-lipid intensity columns together with the LD540 fluorescence column, then compute non-parametric Spearman rank correlations across all single cells (appropriate for non-normal single-cell intensity distributions) and retain species with distinct positive vs negative coefficients — the reference result being TG(48:0) r=0.38 (steatotic LD-rich subpopulation) versus LPE(21:1)/LPC(18:1) r=-0.23. Treat unresolved isomeric candidates (LPE(21:1) or LPC(18:1)) as a single annotation group, since MALDI-IMS at single-cell resolution cannot separate them.

## Related tools

- **SpaceM** (Generates the per-cell spatio-molecular matrix (MORPHnMOL.csv) whose lipid intensity columns and microscopy-derived LD540 values are the substrate for annotation and correlation) — github.com/alexandrovteam/SpaceM
- **METASPACE** (Annotates MALDI-imaging m/z features to lipid species (isomeric groups such as TG(48:0), LPE(21:1)/LPC(18:1)) prior to per-cell correlation testing)
- **SCANPY** (Single-cell analysis framework within which per-cell lipid intensities and Spearman correlations against LD540 are computed)
- **Python** (Execution environment for loading MORPHnMOL.csv (pandas) and running correlation calculations)

## Examples

```
import pandas as pd; from scipy.stats import spearmanr; df = pd.read_csv('scAnalysis/MORPHnMOL.csv'); rho, p = spearmanr(df['TG(48:0)'], df['LD540'])
```

## Evaluation signals

- Recovered Spearman coefficients reproduce the published marker values: TG(48:0) positively correlated with LD540 at r=0.38 and LPE(21:1)/LPC(18:1) anti-correlated at r=-0.23
- TG(48:0) is identified as a marker of the smaller LD-rich subpopulation and LPE(21:1)/LPC(18:1) of the bigger subpopulation, matching the reported directionality
- The correlation is computed over all single cells from the final matrix (MORPHnMOL.csv in the scAnalysis sub-folder), not over averaged wells
- Isomeric candidates are reported as a combined group (LPE(21:1) or LPC(18:1)) rather than as individually resolved species
- Results are written to a correlation table (e.g. lipid_ld540_correlations.csv) with rho and p-value per lipid–fluorescence pair

## Limitations

- MALDI-IMS at single-cell resolution cannot resolve isomeric lipids, so LPE(21:1) and LPC(18:1) must be treated as one annotation group
- The signal depends on ablation marks being visible and non-overlapping in post-MALDI microscopy; co-sampling of neighboring cells can distort per-cell lipid assignments
- Lipid intensity distributions across single cells are non-normal, which is why rank-based (Spearman) rather than Pearson correlation is used
- Single-cell lipid intensities require the full SpaceM co-registration and normalization pipeline to have been executed correctly before annotation-based correlation is meaningful

## Evidence

- [results] Co-registering single-cell lipid intensities with LD540 identifies TG(48:0) as a positive steatosis marker and LPE(21:1)/LPC(18:1) as an anti-correlated marker of the LD-low subpopulation: [evidence span withheld — anchor status: not_found]
- [abstract] SpaceM couples MALDI-imaging mass spectrometry with light microscopy to produce per-cell metabolic profiles suitable for lipid-state analysis: [evidence span withheld — anchor status: not_found]
