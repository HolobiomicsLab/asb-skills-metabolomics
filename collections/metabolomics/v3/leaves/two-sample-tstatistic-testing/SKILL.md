---
name: two-sample-tstatistic-testing
description: Use when ranked metabolite sets (e.g. KEGG pathways from PiMP_KEGG) need
  p-values based on activity-level changes between case and control groups, from a
  log2-transformed, standardised metabolomics intensity matrix with known experimental
  groups.
when_to_use_negative:
- Peaks lack compound annotations (KEGG/ChEBI IDs) — unannotated peaks are excluded
  from pathway analysis, so coverage will be too sparse for meaningful activity testing
- The goal is overrepresentation or rank-enrichment testing on membership lists only
  (use ORA or GSEA in PALS instead of activity-level decomposition)
- There is no case/control contrast defined — t-statistics and the permutation null
  require two-sample comparisons
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: PALS
  role: 'Implements the mPLAGE decomposition: SVD-derived activity levels, t-statistics
    per comparison, permutation null, and GEV-calibrated p-values'
  repo: https://github.com/glasgowcompbio/PALS
- name: SciPy
  role: Standardises the intensity matrix to zero mean and unit variance via its preprocessing
    module before testing
- name: pandas
  role: Holds int_df and annotation_df dataframes passed to DataSource when PALS is
    used programmatically
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:e522ec16b686eb7897d851a9e5aa2801953c6fb8e9907decff2b9b5938094cc8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/PALS
  tools:
  - PALS
  - Python
  - SciPy
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# two-sample-tstatistic-testing

## Summary

Computes two-sample t-statistics on the leading singular vector (first metacompound) of each metabolite set's standardised intensity matrix, builds a permutation null, and calibrates set-level p-values with a GEV distribution fitted by maximum likelihood. This is the statistical core of the mPLAGE method in PALS.

## When to use

Use when ranked metabolite sets (e.g. KEGG pathways from PiMP_KEGG) need p-values based on activity-level changes between case and control groups, from a log2-transformed, standardised metabolomics intensity matrix with known experimental groups.

## When NOT to use

- Peaks lack compound annotations (KEGG/ChEBI IDs) — unannotated peaks are excluded from pathway analysis, so coverage will be too sparse for meaningful activity testing
- The goal is overrepresentation or rank-enrichment testing on membership lists only (use ORA or GSEA in PALS instead of activity-level decomposition)
- There is no case/control contrast defined — t-statistics and the permutation null require two-sample comparisons

## Inputs

- Feature intensity CSV (rows = peak features, columns = samples with a group line, e.g. notebooks/test_data/HAT/int_df.csv)
- Annotation CSV mapping peak IDs to KEGG or ChEBI compound IDs (e.g. notebooks/test_data/HAT/annotation_df.csv)
- Experimental design with groups and case/control comparisons (e.g. Stage_1/Control, Stage_2/Control)
- Pathway database (--db PiMP_KEGG, COMPOUND, ChEBI, UniProt, or ENSEMBL)
- min_replace threshold for imputation (default 5000)

## Outputs

- Per-pathway GEV-calibrated p-values for each defined comparison
- Pathway ranking table sortable by p-value columns, including coverage columns unq_pw_F, tot_ds_F, and F_coverage (e.g. test_output.csv)

## How to apply

First prepare the data: impute zero intensities per experimental factor (all-zero samples get min_replace=5000, partially-zero samples get the factor's non-zero mean), log2-transform, and standardise with SciPy's preprocessing module to zero mean and unit variance. For each pathway, extract the standardised intensity matrix of its annotated features, run an SVD, and take the first metacompound (leading singular vector) as the set's activity levels. Compute two-sample t-statistics on the first metacompound for each defined comparison (e.g. Stage_1/Control and Stage_2/Control). Build a permutation null by shuffling sample group labels and recomputing the t-statistics per comparison, then fit a generalised extreme value (GEV) distribution to the extreme permuted t-statistics by maximum-likelihood estimation and use it to calibrate each pathway's p-value. Rationale: testing the leading activity vector is more robust to missing features and noise than ORA or GSEA on metabolomics peak data.

## Related tools

- **PALS** (Implements the mPLAGE decomposition: SVD-derived activity levels, t-statistics per comparison, permutation null, and GEV-calibrated p-values) — https://github.com/glasgowcompbio/PALS
- **SciPy** (Standardises the intensity matrix to zero mean and unit variance via its preprocessing module before testing)
- **pandas** (Holds int_df and annotation_df dataframes passed to DataSource when PALS is used programmatically)

## Examples

```
python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control Stage_2/Control
```

## Evaluation signals

- t-statistics are computed on the first metacompound (leading singular vector) per pathway, not on individual features
- GEV parameters are fitted to the extreme (minimum/maximum) permuted t-statistics by maximum-likelihood estimation
- Output table contains sortable p-value columns per comparison plus coverage columns unq_pw_F, tot_ds_F, F_coverage with F_coverage = tot_ds_F / unq_pw_F
- Standardised intensity matrix has zero mean and unit variance across samples (verifiable with SciPy preprocessing)
- Rankings replicate the reference run on HAT data (Stage_1/Control, Stage_2/Control) with the same min_replace=5000 imputation

## Limitations

- Peaks without annotations are silently excluded from pathway analysis, so p-values depend on annotation coverage
- Permutation-based GEV calibration is stochastic; results depend on the number of label shuffles
- Normalisation choices can significantly impact pathway analysis results; PALS offers a framework to investigate this rather than guaranteeing invariance

## Evidence

- [intro] SVD activity decomposition precedes testing: "a complete tool that performs database queries of pathways, decomposes activity levels in pathways via [the PLAGE method]"
- [other] GEV calibration of p-values: [evidence span withheld — anchor status: not_found]
- [methods] Standardisation requirement: "standardised using the preprocessing module in Scipy such that the intensity matrix has a zero mean and unit variance"
- [readme] Imputation rule before testing: "if only some of the sample values in a factor are zero then these are replaced by the mean value of the non-zero samples in that factor"
- [readme] Concrete invocation on HAT data: "python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control Stage_2/Control"
