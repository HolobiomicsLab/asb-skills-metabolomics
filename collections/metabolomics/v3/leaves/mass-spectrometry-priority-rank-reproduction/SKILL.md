---
name: mass-spectrometry-priority-rank-reproduction
description: Use when a raw MZmine feature quantification table plus GNPS FBMN metadata
  must be converted into a ranked extract list with FC, FS, LC, CC and SC components
  to prioritize extracts for novel-compound isolation.
when_to_use_negative:
- When no MZmine quantification table and GNPS FBMN metadata exist yet — Inventa cannot
  even compute the Feature Component without them
- When the goal is de novo compound structure elucidation rather than extract-set
  prioritization; Inventa only ranks extracts by novelty potential
- When the extract set lacks taxonomic metadata (ATTRIBUTE_Species) — the Literature
  Component and Open Tree of Life name cleaning depend on it
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3360
tools:
- name: Inventa
  role: Core notebook that computes the FC, LC, CC, SC components and combines them
    into the Priority Score rank
  repo: https://github.com/luigiquiros/inventa
- name: MZmine3
  role: Source of the feature quantification table (data_process_origin='MZMine3',
    ion identity aware)
- name: GNPS
  role: Feature-based molecular networking job and metadata format; annotations cleaned
    with ppm_error=5, shared_peaks=10, cosine=0.7
- name: timaR
  role: Weighted in-silico annotation results feeding the annotation cleanup
- name: SIRIUS/CANOPUS
  role: Chemical class predictions for the Class Component (min_class_confidence=0.8,
    min_recurrence=5)
  repo: https://github.com/kaibioinfo/canopus_treemap
- name: MEMO
  role: MS2 sample vectorization producing the dissimilarity matrix mined by outlier
    detection for the Similarity Component
  repo: https://github.com/mandelbrot-project/memo
- name: scikit-bio
  role: Dependency that may need conda-forge installation before creating the Inventa
    environment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:116b05d3e32d722ee4e1cc06250640f55d9812744b747c698f3b605350d73f04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luigiquiros/inventa
  tools:
  - GNPS
  - Inventa
  - MEMO
  - MZmine3
  - SIRIUS
  - SIRIUS/CANOPUS
  - scikit-bio
  - timaR
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# mass-spectrometry-priority-rank-reproduction

## Summary

Reproduces Inventa's four-component Priority Score ranking of natural extracts from untargeted MS data, benchmarking against the paper's Celastraceae result where Pristimera indica roots extract ranked first with PS 3.23.

## When to use

Use when a raw MZmine feature quantification table plus GNPS FBMN metadata must be converted into a ranked extract list with FC, FS, LC, CC and SC components to prioritize extracts for novel-compound isolation.

## When NOT to use

- When no MZmine quantification table and GNPS FBMN metadata exist yet — Inventa cannot even compute the Feature Component without them
- When the goal is de novo compound structure elucidation rather than extract-set prioritization; Inventa only ranks extracts by novelty potential
- When the extract set lacks taxonomic metadata (ATTRIBUTE_Species) — the Literature Component and Open Tree of Life name cleaning depend on it

## Inputs

- GNPS-format metadata table (e.g. Celastraceae_Set_metadata_pos.tsv) with ATTRIBUTE_Species and ATTRIBUTE_Organe headers
- MZmine3 quantitative table of 'Peak area', 'row m/z', 'row retention time' (Celastraceae_pos_quant.csv)
- timaR weighted spectral-annotation results (tima_results_filename)
- SIRIUS CANOPUS recomputed output (canopus_npc_summary.tsv from canopus_treemap Canopus.npcSummary())
- MEMO vectorized dissimilarity matrix (Celastraceae_memomatrix.csv)

## Outputs

- Tab-separated results file in the results folder ranking all 76 extracts with per-extract FC, FS, LC, CC, SC and final Priority Score
- Top-ranked extract identification (expected: Pristimera indica roots, PS 3.23)

## How to apply

Load Celastraceae_Set_metadata_pos.tsv (GNPS-format metadata with ATTRIBUTE_Species and ATTRIBUTE_Organe), Celastraceae_pos_quant.csv (MZmine3 'Peak area' table), timaR spectral match results, SIRIUS annotations, canopus_npc_summary.tsv, and Celastraceae_memomatrix.csv into the Inventa notebook with data_process_origin='MZMine3', use_ion_dentity=True, ionisation_mode='pos'. Clean annotations with ppm_error=5, shared_peaks=10, cosine=0.7 and cut-offs min_score_final=0.0, min_ZodiacScore=0.9, min_ConfidenceScore=0.0. Compute FC with min_specificity=90 and only_feature_specificity=False; compute LC with max_comp_reported_sp=20, max_comp_reported_g=50, max_comp_reported_f=500 and taxonomic weights ws=wg=wf=1; compute SC via machine-learning outlier detection on the MEMO dissimilarity matrix; compute CC with min_class_confidence=0.8 and min_recurrence=5. Combine components with equal weights w1=w2=w3=w4=1 and rank; export the full results as a tab-separated file in the results folder.

## Related tools

- **Inventa** (Core notebook that computes the FC, LC, CC, SC components and combines them into the Priority Score rank) — https://github.com/luigiquiros/inventa
- **MZmine3** (Source of the feature quantification table (data_process_origin='MZMine3', ion identity aware))
- **GNPS** (Feature-based molecular networking job and metadata format; annotations cleaned with ppm_error=5, shared_peaks=10, cosine=0.7)
- **timaR** (Weighted in-silico annotation results feeding the annotation cleanup)
- **SIRIUS/CANOPUS** (Chemical class predictions for the Class Component (min_class_confidence=0.8, min_recurrence=5)) — https://github.com/kaibioinfo/canopus_treemap
- **MEMO** (MS2 sample vectorization producing the dissimilarity matrix mined by outlier detection for the Similarity Component) — https://github.com/mandelbrot-project/memo
- **scikit-bio** (Dependency that may need conda-forge installation before creating the Inventa environment)

## Examples

```
git clone https://github.com/luigiquiros/inventa.git && conda env create -n inventa -f inventa/environment.yml && conda activate inventa && jupyter notebook notebook/inventa.ipynb
```

## Evaluation signals

- Top-ranked extract must be Pristimera indica roots extract with PS 3.23, FC 0.36, FS 0.37, LC 0.87, CC 1, SC 1, matching Table 1
- FS of 0.37 must be interpretable as 37% of the extract's features being specific within the 76-extract set
- Output must be a TAB-separated file in the results folder containing all component values per extract
- CC maxima must be decomposable as CCs + CCg (a CC of 1 means the class is new to both species and genus, 0.5 each)
- Run with equal modulating weights w1=w2=w3=w4=1 must reproduce the rank order without manual rescaling

## Limitations

- Binder instances shut down after 15 min of inactivity, so parameters and results must be saved locally
- The quantitative table must contain only 'Peak area' or 'Peak height' (one at a time); other columns must be dropped manually or inside quant_table()
- Columns headed 'Unknown: number' in the quantitative table are handled by the script; extra exported columns like ion identities require manual df.drop() edits in quand_table()
- CANOPUS NPC summary requires recomputing the SIRIUS project space with canopus_treemap's Canopus class (from canopus import Canopus)

## Evidence

- [methods] default parameters Celastraceae top rank: [evidence span withheld — anchor status: not_found]
- [methods] annotation cleanup thresholds: [evidence span withheld — anchor status: not_found]
- [readme] FC definition: "The **Feature Component (FC)** is a ratio of the number of specific non-annotated features over the total number of features of each extract."
- [readme] LC definition: "The **Literature Component (LC)** is a score based on the number of compounds reported in the literature for the taxon of a given extract."
- [readme] SC definition and MEMO: "The matrix is mined through multiple outlier detection machine learning algorithms to highlight spectrally dissimilar extracts (outliers)."
- [readme] CC definition: "A CC value of 1 implies that the chemical class is new to both the species (CCs 0.5) and the genus (CCg 0.5)."
- [readme] required minimum inputs: [evidence span withheld — anchor status: not_found]
- [readme] output format: [evidence span withheld — anchor status: external_doc]
- [readme] conda environment install: "conda env create -n inventa -f inventa/environment.yml"
