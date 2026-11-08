---
name: permutation-null-sampling
description: Use when pathway activity levels (e.g. the first metacompound/SVD singular
  vector per metabolite set) have been computed and calibrated p-values are needed
  that account for noise and missing peaks in metabolomics intensity data, for comparisons
  like Stage_1/Control.
when_to_use_negative:
- Peaks lack compound annotations (KEGG or ChEBI IDs) — unannotated features are not
  used for pathway analysis, so the permutation null would be built on an empty or
  tiny feature set
- No case/control comparison is defined — the permutation null is built by shuffling
  group labels and recomputing two-sample t-statistics per comparison, which requires
  explicit group structure
- The goal is annotation-enrichment ranking rather than activity-level decomposition
  — ORA or GSEA (both included in PALS) may be more appropriate comparators
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: PALS
  role: 'Implements mPLAGE: SVD-derived activity levels, label-shuffling permutation
    tests, and GEV-calibrated p-values; run via pals/run_pals.py with method=PLAGE'
  repo: https://github.com/glasgowcompbio/PALS
- name: SciPy
  role: Preprocessing module standardises the imputed, log2-transformed intensity
    matrix to zero mean and unit variance before t-statistics are computed
- name: pandas
  role: Provides int_df / annotation_df DataFrames passed directly to DataSource when
    PALS is used programmatically
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:7d0be3801bcdf7c9de1789c0d6e81805923af1ea13adf100685427628aadd19d
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
yes_rate: 0.75
n_claims: 16
---

# permutation-null-sampling

## Summary

Builds an empirical null distribution for pathway activity t-statistics by repeatedly shuffling sample group labels, then calibrates pathway p-values by fitting a generalised extreme value (GEV) distribution (maximum-likelihood) to the extreme permuted statistics, as implemented in PALS' mPLAGE method.

## When to use

Use when pathway activity levels (e.g. the first metacompound/SVD singular vector per metabolite set) have been computed and calibrated p-values are needed that account for noise and missing peaks in metabolomics intensity data, for comparisons like Stage_1/Control.

## When NOT to use

- Peaks lack compound annotations (KEGG or ChEBI IDs) — unannotated features are not used for pathway analysis, so the permutation null would be built on an empty or tiny feature set
- No case/control comparison is defined — the permutation null is built by shuffling group labels and recomputing two-sample t-statistics per comparison, which requires explicit group structure
- The goal is annotation-enrichment ranking rather than activity-level decomposition — ORA or GSEA (both included in PALS) may be more appropriate comparators

## Inputs

- Intensity CSV of peak features by samples (rows = peaks, columns = samples; second line optionally encodes sample group membership, e.g. notebooks/test_data/HAT/int_df.csv)
- Annotation CSV mapping peak IDs to compound IDs (KEGG or ChEBI), e.g. notebooks/test_data/HAT/annotation_df.csv
- Experimental design with case/control comparisons (e.g. Stage_1/Control, Stage_2/Control)
- Pathway/metabolite-set database (--db PiMP_KEGG, COMPOUND, ChEBI, UniProt, ENSEMBL)
- min_replace imputation threshold (default 5000)

## Outputs

- Pathway ranking table (e.g. test_output.csv) with calibrated GEV-based p-value columns per comparison
- Coverage columns: unq_pw_F (unique formulae per pathway), tot_ds_F (formula hits in dataset), F_coverage (proportion)

## How to apply

First prepare the intensity matrix: impute all-zero samples in an experimental factor with min_replace (default 5000) and partially-zero samples with the factor's non-zero mean, log2-transform, then standardise with SciPy's preprocessing module to zero mean and unit variance across samples. For each metabolite set (e.g. PiMP_KEGG pathways), extract the standardised intensity matrix of its annotated features, compute the SVD, and take the first metacompound as the set's activity levels. Compute two-sample t-statistics on that metacompound for each defined case/control comparison (e.g. Stage_1/Control and Stage_2/Control). Build the permutation null by repeatedly shuffling sample group labels and recomputing the t-statistics per comparison; fit a GEV distribution by maximum likelihood to the extreme permuted t-statistics and use it to calibrate each pathway's final p-value. Rank pathways by the calibrated p-value columns and report coverage columns (unq_pw_F, tot_ds_F, F_coverage).

## Related tools

- **PALS** (Implements mPLAGE: SVD-derived activity levels, label-shuffling permutation tests, and GEV-calibrated p-values; run via pals/run_pals.py with method=PLAGE) — https://github.com/glasgowcompbio/PALS
- **SciPy** (Preprocessing module standardises the imputed, log2-transformed intensity matrix to zero mean and unit variance before t-statistics are computed)
- **pandas** (Provides int_df / annotation_df DataFrames passed directly to DataSource when PALS is used programmatically)

## Examples

```
python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control Stage_2/Control --min_replace 5000
```

## Evaluation signals

- Output CSV contains sortable p-value columns plus unq_pw_F, tot_ds_F, and F_coverage, with F_coverage equal to tot_ds_F / unq_pw_F
- Calibrated p-values lie in [0, 1] and derive from the GEV fit to the permuted extreme t-statistics rather than a parametric t-distribution
- Re-running with the same intensity CSV, annotation CSV, --db PiMP_KEGG and --comparisons Stage_1/Control Stage_2/Control reproduces the same pathway ordering (up to permutation-test stochasticity)
- Only annotated peaks contribute: pathways whose features are all unannotated should yield no ranking entry
- Preprocessing invariant: after imputation (min_replace=5000) and log2 transformation, the standardised intensity matrix has zero mean and unit variance across samples

## Limitations

- P-value calibration depends on the number of permutations; too few shuffles give an unstable GEV maximum-likelihood fit to the extreme statistics
- Normalisation choices (imputation value, log2, standardisation) can have a significant impact on pathway analysis results, and must be fixed explicitly
- Only metabolic pathways are available in offline Reactome mode, and the permutation test requires defined two-group comparisons

## Evidence

- [other] GEV null calibration: [evidence span withheld — anchor status: not_found]
- [readme] Standardisation before testing: "The data is subsequently transformed to log-2 base and standardised using the preprocessing module in Scipy such that the intensity matrix has a zero mean and unit variance across the samples."
- [methods] Imputation rules feeding the null: "if all of the samples in a single experimental factor have intensities of zero these are replaced by the minimum intensity value"
- [readme] Ranking output structure: "Pathways are identified by their id and can be sorted by the `p-value` columns. The column `unq_pw_F` lists the unique formulae found in that pathway"
