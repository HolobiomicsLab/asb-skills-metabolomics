---
name: svd-pathway-activity-decomposition
description: Use when peak-intensity matrices with compound annotations (KEGG/ChEBI
  IDs) must be converted to pathway-level activity scores, especially when noise and
  missing peaks make ORA/GSEA unreliable.
when_to_use_negative:
- If most peak features lack compound annotations — unannotated peaks are not used
  for pathway analysis, so coverage will be too low to rank pathways meaningfully
- When the goal is to analyse individual metabolites rather than groups of metabolites
  as sets (pathways, GNPS Molecular Families, MS2LDA Mass2Motifs)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: PALS
  role: Python library / command-line tool that performs the database queries, SVD-based
    activity-level decomposition (PLAGE/mPLAGE), GEV-calibrated p-values, and pathway
    ranking
  repo: github.com/glasgowcompbio/PALS
- name: pandas
  role: Holds the intensity (int_df) and annotation (annotation_df) data frames passed
    programmatically to PALS DataSource
- name: SciPy
  role: preprocessing module used to standardise the log2-transformed intensity matrix
    to zero mean and unit variance
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:f250b2582c2c66b4fdf019e495ec9336df4da3f398e00c6996c0a19a643b807c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PALS
  - Python
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# svd-pathway-activity-decomposition

## Summary

Decomposes per-metabolite-set intensity matrices via singular value decomposition (the PLAGE method), taking the first metacompound (leading singular vector) as the set's activity levels, then ranks metabolite sets by their activity-level change using mPLAGE p-values.

## When to use

Use when peak-intensity matrices with compound annotations (KEGG/ChEBI IDs) must be converted to pathway-level activity scores, especially when noise and missing peaks make ORA/GSEA unreliable.

## When NOT to use

- If most peak features lack compound annotations — unannotated peaks are not used for pathway analysis, so coverage will be too low to rank pathways meaningfully
- When the goal is to analyse individual metabolites rather than groups of metabolites as sets (pathways, GNPS Molecular Families, MS2LDA Mass2Motifs)

## Inputs

- Peak-intensity CSV matrix (rows = peak features with peak ID in column one, columns = samples, second line indicating sample group membership)
- Annotation CSV matrix with two columns: peak ID and assigned metabolite annotation (KEGG or ChEBI database IDs)
- Pathway database choice (PiMP_KEGG, COMPOUND, ChEBI for metabolomics; UniProt, ENSEMBL for proteomics/transcriptomics)
- List of case/control comparisons (e.g. Stage_1/Control Stage_2/Control)
- min_replace threshold for imputation (default 5000)

## Outputs

- Pathway ranking table (test_output.csv) with calibrated p-value columns sortable per comparison, plus unq_pw_F, tot_ds_F, and F_coverage columns
- Per-pathway activity levels (first metacompound / leading singular vector of each set's intensity matrix)

## How to apply

Load the peak-intensity CSV (rows = peak features, columns = samples, second line holding group labels) and the two-column annotation matrix mapping peak IDs to KEGG or ChEBI IDs; peaks without annotations are excluded. Impute zeros on load: all-zero samples in an experimental factor are replaced by the minimum intensity value (--min_replace, default 5000) and partially-zero samples by the mean of the non-zero values in that factor, then log2-transform and standardise with SciPy's preprocessing module to zero mean and unit variance across samples. For each metabolite set in the chosen pathway database (e.g. PiMP_KEGG, COMPOUND, ChEBI), extract its annotated standardised intensity matrix, compute its SVD, and take the first metacompound as the set's activity levels. Compute two-sample t-statistics on the metacompound for each case/control comparison (e.g. Stage_1/Control), build a permutation null by shuffling sample labels, and fit a generalised extreme value (GEV) distribution by maximum likelihood to the extreme permuted t-statistics to calibrate per-pathway p-values. Rank pathways by the calibrated p-value columns, retaining the unq_pw_F (unique formulae per pathway), tot_ds_F (formula hits in the dataset) and F_coverage (tot_ds_F/unq_pw_F) columns.

## Related tools

- **PALS** (Python library / command-line tool that performs the database queries, SVD-based activity-level decomposition (PLAGE/mPLAGE), GEV-calibrated p-values, and pathway ranking) — github.com/glasgowcompbio/PALS
- **pandas** (Holds the intensity (int_df) and annotation (annotation_df) data frames passed programmatically to PALS DataSource)
- **SciPy** (preprocessing module used to standardise the log2-transformed intensity matrix to zero mean and unit variance)

## Examples

```
python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control Stage_2/Control --min_replace 5000
```

## Evaluation signals

- Output CSV must contain p-value, unq_pw_F, tot_ds_F, and F_coverage columns
- F_coverage must equal tot_ds_F / unq_pw_F for each pathway row
- For the HAT case study, reproduced changing-pathway counts should match the paper's reported 52/74 (Plasma S1:C, S2:C) and 42/128, 42/143 (CSF) counts
- CSF S2-vs-control ranking should show all detected T. brucei auxotrophic amino-acid pathway activities decreasing (glycine excluded as undetected), matching Supplementary Table S6
- Standardised intensity matrix should have zero mean and unit variance across samples after log2 transformation

## Limitations

- mPLAGE is more robust to missing features and noisy data than ORA and GSEA, but peaks without annotations are silently dropped from pathway analysis
- Normalisation choices can have a significant impact on pathway analysis results; PALS offers a framework to investigate this rather than removing the sensitivity
- Offline Reactome mode provides only metabolic pathways unless --use_all_reactome_pathways is used with an online Neo4j server connection

## Evidence

- [intro] decomposes activity levels in pathways via the PLAGE method: "Here we introduce **PALS (Pathway Activity Level Scoring)**, a complete tool that performs database queries of pathways, decomposes activity levels in pathways via [the PLAGE method]"
- [readme] The results are found to be more robust to noise and missing peaks compared to the alternatives (ORA, GSEA).: "The results are found to be more robust to noise and missing peaks compared to the alternatives (ORA, GSEA). This is particularly important for metabolomics peak data, where noise and missing peaks"
- [methods] Imputation and standardisation of the intensity matrix on load: "if all of the samples in a single experimental factor have intensities of zero these are replaced by the minimum intensity value"
- [readme] Output columns unq_pw_F, tot_ds_F, F_coverage: "The column `unq_pw_F` lists the unique 
formulae found in that pathway, `tot_ds_F` lists the formula hits found in the dataset, and `F_coverage` is the proportion
of `tot_ds_F` to `unq_pw_F`."
- [readme] Peaks without annotations are excluded from pathway analysis: "users also provide a list of compound annotations assigned to peak features (peaks that do not have 
annotations will not be used for pathway analysis)"
