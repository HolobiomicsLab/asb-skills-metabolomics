---
name: untargeted-metabolomics-feature-scoring
description: Use when a set of untargeted metabolomics extracts (GNPS FBMN job + MZmine
  2/3 quant table) must be prioritized for structural novelty, with per-extract FC,
  FS, LC, CC, and SC scores and a final rank.
when_to_use_negative:
- Input is only a spectral library search result or single-sample spectrum without
  an extract set — FC specificity and MEMO-based SC require a multi-extract set to
  compare against
- No GNPS job or MZmine quantification table exists; the Feature Component cannot
  be computed and only optional components (LC, CC, SC) would run
- The goal is de novo structure elucidation of a single annotated compound rather
  than extract-level prioritization for novelty
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: Inventa
  role: computes the FC, LC, CC, SC components and combines them into the Priority
    Score rank
  repo: https://github.com/luigiquiros/inventa
- name: MZmine3
  role: produces the feature quantitative table consumed as data_process_origin='MZMine3'
- name: GNPS
  role: provides the FBMN job and metadata format; its annotations are cleaned with
    ppm_error=5, shared_peaks=10, cosine=0.7
- name: timaR
  role: supplies taxonomically weighted in silico annotation results (tima_results_filename)
  repo: https://taxonomicallyinformedannotation.github.io/tima-r/index.html
- name: SIRIUS/CANOPUS
  role: chemical class predictions used for the Class Component via canopus_npc_summary
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: MEMO
  role: MS2-based sample vectorization generating the dissimilarity matrix mined by
    outlier detection for SC
  repo: https://github.com/mandelbrot-project/memo
- name: canopus_treemap
  role: recomputes the Sirius project space to produce the NPC class summary (C.npcSummary().to_csv)
  repo: https://github.com/kaibioinfo/canopus_treemap
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:300f40074267da17c581c3eea2a8a8884f95a6c5cfa2e2e75edabdb9094d2b5d
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
  - canopus_treemap
  - scikit-bio
  - timaR
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7368421052631579
n_claims: 19
---

# untargeted-metabolomics-feature-scoring

## Summary

Rank natural extracts for chemical novelty by combining four Inventa components — Feature Component (FC), Literature Component (LC), Class Component (CC), and Similarity Component (SC) — into a weighted Priority Score over untargeted MS data (MZmine quant table, GNPS annotations, SIRIUS/CANOPUS, timaR, MEMO).

## When to use

Use when a set of untargeted metabolomics extracts (GNPS FBMN job + MZmine 2/3 quant table) must be prioritized for structural novelty, with per-extract FC, FS, LC, CC, and SC scores and a final rank.

## When NOT to use

- Input is only a spectral library search result or single-sample spectrum without an extract set — FC specificity and MEMO-based SC require a multi-extract set to compare against
- No GNPS job or MZmine quantification table exists; the Feature Component cannot be computed and only optional components (LC, CC, SC) would run
- The goal is de novo structure elucidation of a single annotated compound rather than extract-level prioritization for novelty

## Inputs

- GNPS-format metadata table (mandatory headers ATTRIBUTE_Species, ATTRIBUTE_Organe)
- MZmine 2/3 feature quantitative table (Peak area, row m/z, row retention time columns)
- timaR weighted in silico annotation results file
- SIRIUS/CANOPUS canopus_npc_summary.tsv (recomputed project space)
- MEMO vectorized dissimilarity matrix (Celastraceae_memomatrix.csv)

## Outputs

- Tab-separated results file in the results folder with per-extract FC, FS, LC, CC, SC values and final Priority Score rank
- Ranked extract list (e.g., top extract Pristimera indica roots, PS 3.23)

## How to apply

Load the GNPS-format metadata table, the MZmine quantitative table ('Peak area', 'row m/z', 'row retention time' columns), timaR annotation results, SIRIUS/CANOPUS npc_summary, and the MEMO vectorized dissimilarity matrix into the Inventa notebook with data_process_origin='MZMine3', use_ion_dentity=True, ionisation_mode='pos'. Clean GNPS annotations with ppm_error=5, shared_peaks=10, cosine=0.7 and annotation cut-offs min_score_final=0.0, min_ZodiacScore=0.9, min_ConfidenceScore=0.0. Compute FC with min_specificity=90 (only_feature_specificity=False), LC with max_comp_reported_sp=20, max_comp_reported_g=50, max_comp_reported_f=500 and weights ws=wg=wf=1, SC via machine-learning outlier detection on the MEMO matrix, and CC with min_class_confidence=0.8 and min_recurrence=5. Sum the four components with modulating weights w1=w2=w3=w4=1 to obtain the Priority Score and export the ranked results as a tab-separated file in the results folder.

## Related tools

- **Inventa** (computes the FC, LC, CC, SC components and combines them into the Priority Score rank) — https://github.com/luigiquiros/inventa
- **MZmine3** (produces the feature quantitative table consumed as data_process_origin='MZMine3')
- **GNPS** (provides the FBMN job and metadata format; its annotations are cleaned with ppm_error=5, shared_peaks=10, cosine=0.7)
- **timaR** (supplies taxonomically weighted in silico annotation results (tima_results_filename)) — https://taxonomicallyinformedannotation.github.io/tima-r/index.html
- **SIRIUS/CANOPUS** (chemical class predictions used for the Class Component via canopus_npc_summary) — https://bio.informatik.uni-jena.de/software/sirius/
- **MEMO** (MS2-based sample vectorization generating the dissimilarity matrix mined by outlier detection for SC) — https://github.com/mandelbrot-project/memo
- **canopus_treemap** (recomputes the Sirius project space to produce the NPC class summary (C.npcSummary().to_csv)) — https://github.com/kaibioinfo/canopus_treemap

## Examples

```
conda env create -n inventa -f inventa/environment.yml && conda activate inventa && jupyter lab notebook/inventa.ipynb  # set data_process_origin='MZMine3', min_specificity=90, min_ZodiacScore=0.9, w1=w2=w3=w4=1
```

## Evaluation signals

- Top-ranked extract matches Table 1: Pristimera indica roots with PS 3.23, FC 0.36, FS 0.37, LC 0.87, CC 1, SC 1
- FC/FS values lie in [0,1] and FS=0.37 means 37% of the extract's features are specific within the 76-extract set
- Output is a valid .tsv in the results folder containing all four component columns plus the final rank
- Annotation filtering retains only GNPS matches with ppm error ≤5, ≥10 shared peaks, and cosine ≥0.7, and ZodiacScore ≥0.9
- CC values respect the 0.5/0.5 decomposition: CC=1 implies the class is new to both species (CCs 0.5) and genus (CCg 0.5)

## Limitations

- Binder instances shut down after 15 min of inactivity, so parameters and results must be saved locally
- The quantitative table must contain only 'Peak area' or 'Peak height' (one at a time); extra columns must be dropped manually in quand_table()
- CANOPUS NPC support requires recomputing the Sirius project space with the canopus_treemap package before the npc summary can be generated
- If environment creation fails, scikit-bio must be installed from conda-forge before creating the environment

## Evidence

- [methods] component weights: "The rank comes from the addition of four individual components"
- [readme] FC definition: "The **Feature Component (FC)** is a ratio of the number of specific non-annotated features over the total number of features of each extract."
- [readme] SC definition: "The matrix is mined through multiple outlier detection machine learning algorithms to highlight spectrally dissimilar extracts (outliers)."
- [methods] annotation thresholds: "ppm_error = 5                     # min error in ppm to consider an annotation valable"
- [abstract] key finding: "The Pristimera indica (Willd.) A.C.Sm roots extract was highlighted as a promising source of potentially novel compounds"
