---
name: extract-priority-score-calculation
description: Use when prioritizing plant/organism extracts for structural novelty
  after MZmine feature quantification and GNPS FBMN, i.e. when you have a quant table,
  metadata, and optionally ISDB/timaR, SIRIUS/CANOPUS and MEMO inputs.
when_to_use_negative:
- You need spectral annotations themselves rather than extract-level prioritization
  — run ISDB/timaR or SIRIUS/CANOPUS annotation workflows instead.
- Your set contains a single extract — FC specificity and SC outlier detection are
  defined relative to an extract set.
- Only the minimum inputs (quant table + metadata) are available and you expect CC
  or SC in the output — those components require CANOPUS and MEMO files, respectively.
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: Inventa
  role: Pipeline that calculates the FC, LC, CC, SC components and the final Priority
    rank from the input tables
  repo: https://github.com/luigiquiros/inventa.git
- name: MZmine3
  role: Produces the quantitative feature table consumed by quant_table() for the
    FC computation
- name: GNPS
  role: FBMN job whose standard-format metadata table and annotation outputs feed
    the FC/LC calculation
- name: MEMO
  role: MS2 sample vectorization used by the Similarity Component to flag spectrally
    dissimilar extracts (outliers)
  repo: https://github.com/mandelbrot-project/memo
- name: SIRIUS/CANOPUS
  role: Chemical taxonomy predictions (npc_summary.csv via canopus_treemap) driving
    the Class Component
  repo: https://github.com/kaibioinfo/canopus_treemap
- name: timaR
  role: Taxonomically informed weighted in-silico annotations used to flag annotated
    (non-counted) features in FC
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:89fb33ad90db7880d1432c0674d98e07f3a2ef473530da7be8136363a76b7717
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luigiquiros/inventa.git
  tools:
  - GNPS
  - Inventa
  - MEMO
  - MZmine2
  - MZmine3
  - SIRIUS/CANOPUS
  - timaR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# extract-priority-score-calculation

## Summary

Computes Inventa's Priority rank for natural extracts by adding four components — Feature Component (FC), Literature Component (LC), Class Component (CC) and Similarity Component (SC) — estimated from untargeted MS data, spectral annotations, CANOPUS taxonomy and literature reports.

## When to use

Use when prioritizing plant/organism extracts for structural novelty after MZmine feature quantification and GNPS FBMN, i.e. when you have a quant table, metadata, and optionally ISDB/timaR, SIRIUS/CANOPUS and MEMO inputs.

## When NOT to use

- You need spectral annotations themselves rather than extract-level prioritization — run ISDB/timaR or SIRIUS/CANOPUS annotation workflows instead.
- Your set contains a single extract — FC specificity and SC outlier detection are defined relative to an extract set.
- Only the minimum inputs (quant table + metadata) are available and you expect CC or SC in the output — those components require CANOPUS and MEMO files, respectively.

## Inputs

- MZmine 2 or MZmine 3 quantitative feature table (.csv, 'Peak area' or 'Peak height' columns; Ion Identity usage must be specified)
- GNPS-format metadata table (.tsv) with ATTRIBUTE_Species and ATTRIBUTE_Organe headers, same as used for FBMN
- Optional: timaR weighted in-silico annotation results (ISDB) file
- Optional: SIRIUS/CANOPUS npc_summary.csv recomputed via canopus_treemap
- Optional: MEMO vectorized dissimilarity matrix of MS2 peaks and neutral losses
- Optional: literature report counts for the taxon of each extract

## Outputs

- TSV results file with FC, LC, CC, SC, combined Priority rank and associated metrics for every extract
- Per-extract Feature Component (FC) and feature specificity (FS) values

## How to apply

Load the MZmine 2/3 quantitative feature table (using only 'Peak area', 'row m/z' and 'row retention time' columns; only 'Peak height' OR 'Peak area' is considered at a time) together with a GNPS-format metadata table with mandatory ATTRIBUTE_Species (cleaned via Open Tree of Life) and ATTRIBUTE_Organe headers. Row-normalize intensities per extract and compute feature specificity: with min_specificity = 90 a feature is specific when at least 90% (threshold 0.90) of its normalized peak area lies in one extract; with annotation_preference = 0 only features without structural annotation (ISDB/timaR and SIRIUS/CANOPUS hits) count. FC = (number of specific non-annotated features) / (total features of each extract). Optionally compute LC (1 minus reported-compound fractions at species/genus/family levels), CC (1 if the CANOPUS class is new to both species and genus, 0.5 each), and SC (1 if the extract is an outlier via MEMO-based MS2 outlier detection). Sum the four components with a user-defined modulating factor wn per component and write all scores plus the final rank to the output .tsv in the results folder.

## Related tools

- **Inventa** (Pipeline that calculates the FC, LC, CC, SC components and the final Priority rank from the input tables) — https://github.com/luigiquiros/inventa.git
- **MZmine3** (Produces the quantitative feature table consumed by quant_table() for the FC computation)
- **GNPS** (FBMN job whose standard-format metadata table and annotation outputs feed the FC/LC calculation)
- **MEMO** (MS2 sample vectorization used by the Similarity Component to flag spectrally dissimilar extracts (outliers)) — https://github.com/mandelbrot-project/memo
- **SIRIUS/CANOPUS** (Chemical taxonomy predictions (npc_summary.csv via canopus_treemap) driving the Class Component) — https://github.com/kaibioinfo/canopus_treemap
- **timaR** (Taxonomically informed weighted in-silico annotations used to flag annotated (non-counted) features in FC)

## Examples

```
git clone https://github.com/luigiquiros/inventa.git && conda env create -n inventa -f inventa/environment.yml && conda activate inventa
```

## Evaluation signals

- FC and FS values for the Pristimera indica roots extract reproduce the reported 0.36 and 0.37.
- FC lies in [0,1]; e.g. an FC of 0.6 means 60% of the extract's total features are specific within the set and lack structural annotations.
- An LC of 1 indicates no reported compounds for the considered taxon; an SC of 1 means the extract is classified as an outlier in the extract set.
- The output TSV contains a final rank for every sample consistent with the wn-weighted sum of the four components.
- Feature specificity of 1.0 holds exactly when ≥90% of a feature's normalized peak area is detected in the target extract (min_specificity = 90).

## Limitations

- Binder instances shut down after 15 min of inactivity — parameters and results must be saved locally.
- Only one of 'Peak height' or 'Peak area' is considered at a time; other exported columns must be dropped manually or inside quant_table().
- With only the minimum inputs (GNPS job, metadata, quant table) at least the FC is computed; LC, CC and SC require literature, CANOPUS and MEMO files respectively.
- Species names must be cleaned to currently recognized names (e.g. via Open Tree of Life) before running, otherwise the LC taxon lookup is wrong.

## Evidence

- [readme] The **Feature Component (FC)** is a ratio of the number of specific non-annotated features over the total number of features of each extract.: "The **Feature Component (FC)** is a ratio of the number of specific non-annotated features over the total number of features of each extract."
- [readme] The combined score (addition of the four components) can be modulated according to the user preference.: [evidence span withheld — anchor status: external_doc]
- [methods] A modulating factor (wn) gives each component a relative weight according to the user's preferences.: "A modulating factor (wn) gives each component a relative weight according to the user's preferences."
- [readme] An SC value of '1' implies the extract is classified as an outlier within the extract set studied.: "An SC value of ‘1’ implies the extract is classified as an outlier within the extract set studied."
- [methods] The rank comes from the addition of four individual components: "The rank comes from the addition of four individual components"
