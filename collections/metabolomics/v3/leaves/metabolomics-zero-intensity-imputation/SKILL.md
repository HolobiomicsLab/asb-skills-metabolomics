---
name: metabolomics-zero-intensity-imputation
description: Use when a peak intensity matrix (rows = peak features, columns = samples
  with a group line) contains zero values that must be imputed before log-2 transformation
  and pathway-activity scoring, and imputation must respect experimental factors.
when_to_use_negative:
- The intensity matrix contains no zero values within any experimental factor (no
  imputation is needed).
- Only some samples in a factor are zero but you require a global (whole-matrix) imputation
  strategy instead of the factor-wise non-zero mean PALS applies.
- The data are already log-2 transformed and standardised — imputation in PALS is
  performed at load time, before those preprocessing steps.
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0219
tools:
- name: PALS
  role: Performs the factor-aware zero-intensity imputation automatically when the
    intensity matrix is loaded (via DataSource), governed by the --min_replace parameter
    (default 5000).
  repo: https://github.com/glasgowcompbio/PALS
- name: pandas
  role: Loads the peak-intensity matrix and annotation matrices as DataFrames that
    are passed directly to PALS for imputation and downstream PLAGE analysis.
- name: SciPy
  role: Standardises the imputed matrix (zero mean, unit variance) via its preprocessing
    module after the imputation and log-2 transformation steps.
provenance:
  source_task_ids:
  - task_008
  - task_006
  - task_005
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:6faf05bc8ab70e5297e1ee29d108a2cbd8cc2f6fb8481cf6db15cdcf3f203555
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/PALS
  tools:
  - PALS
  - Python
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# metabolomics-zero-intensity-imputation

## Summary

Factor-aware imputation of zero intensities in metabolomics peak-intensity matrices, as implemented when PALS loads data: all-zero factors are replaced by a minimum intensity value (default 5000) and partially-zero factors by the factor's non-zero mean, prior to log-2 transformation and standardisation.

## When to use

Use when a peak intensity matrix (rows = peak features, columns = samples with a group line) contains zero values that must be imputed before log-2 transformation and pathway-activity scoring, and imputation must respect experimental factors.

## When NOT to use

- The intensity matrix contains no zero values within any experimental factor (no imputation is needed).
- Only some samples in a factor are zero but you require a global (whole-matrix) imputation strategy instead of the factor-wise non-zero mean PALS applies.
- The data are already log-2 transformed and standardised — imputation in PALS is performed at load time, before those preprocessing steps.

## Inputs

- Peak intensity CSV matrix (row_id + sample columns, optional second line with group/factor labels, e.g. notebooks/test_data/HAT/int_df.csv)
- Experimental factor/group mapping (group line in CSV or experimental_design dict for programmatic use)
- min_replace threshold (minimum intensity value for imputation, default 5000)

## Outputs

- Imputed intensity matrix (all-zero factor values replaced by min_replace; partial zeros replaced by factor non-zero mean), ready for log-2 transformation and SciPy standardisation
- Downstream pathway ranking CSV (e.g. test_output.csv, rhamnaceae_motif_ranking.csv) whose statistics depend on the imputed values

## How to apply

Load the intensity matrix with a group line mapping each sample column to its experimental factor. When PALS loads the matrix, apply two rules: (1) if ALL samples in a single experimental factor have zero intensity, replace them with the minimum intensity value, controlled by the --min_replace parameter (default 5000, e.g. `--min_replace 5000`); (2) if only SOME samples in a factor are zero, replace them with the mean value of the non-zero samples in that factor. When constructing a DataSource programmatically, pass `min_replace=5000` (or a user-chosen value) to the constructor. The imputed matrix is subsequently log-2 transformed and standardised (zero mean, unit variance) with the SciPy preprocessing module, so imputation must occur before these steps. In the Rhamnaceae GNPS/MS2LDA workflow and the HAT workflows, the default minimum replacement value of 5000 was used for imputing zero intensities within an experimental factor.

## Related tools

- **PALS** (Performs the factor-aware zero-intensity imputation automatically when the intensity matrix is loaded (via DataSource), governed by the --min_replace parameter (default 5000).) — https://github.com/glasgowcompbio/PALS
- **pandas** (Loads the peak-intensity matrix and annotation matrices as DataFrames that are passed directly to PALS for imputation and downstream PLAGE analysis.)
- **SciPy** (Standardises the imputed matrix (zero mean, unit variance) via its preprocessing module after the imputation and log-2 transformation steps.)

## Examples

```
python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control Stage_2/Control --min_replace 5000
```

## Evaluation signals

- No zero intensity values remain within any experimental factor after loading (all-zero factors filled with min_replace=5000; partial zeros filled with the factor's non-zero mean).
- Imputation precedes log-2 transformation and SciPy zero-mean/unit-variance standardisation in the pipeline order.
- Downstream output CSV (e.g. test_output.csv) contains p-value, unq_pw_F, tot_ds_F and F_coverage columns and reproduces reported results when the default min_replace 5000 is used.
- Re-running with a different --min_replace value changes imputed intensities but the imputation rule (all-zero vs partial-zero) remains factor-wise, not global.

## Limitations

- Imputation is factor-wise: it requires correct group labels (group line in the CSV or experimental_design dict); mislabelled factors will fill zeros with statistics from the wrong sample set.
- The minimum replacement value (default 5000) is a user-set parameter; a poorly chosen value can distort pathway activity rankings, and normalisation choices are noted in the paper as having a significant impact on pathway analysis results.
- PALS performs imputation only at load time; there is no exposed option to skip imputation or apply alternative imputation strategies.

## Evidence

- [readme] Data imputation is performed to the intensity matrix when it is loaded: "Data imputation is performed to the intensity matrix when it is loaded: if all of the samples in a single experimental factor have intensities of zero these are replaced by the minimum intensity value"
- [readme] Partial-zero rule: "and if only some of the sample values in a factor are zero then these are replaced by the mean value of the non-zero samples in that factor."
- [readme] Post-imputation preprocessing order: "The data is subsequently transformed to log-2 base and standardised using the preprocessing module in Scipy such that the intensity matrix has a zero mean and unit variance across the samples."
- [readme] min_replace CLI parameter: "--min_replace: The minimum intensity value for data imputation, e.g. `--min_replace 5000`. Defaults to 5000."
- [methods] Programmatic min_replace parameter: "reactome_query=reactome_query, min_replace=min_replace)"
- [abstract] Normalisation impact finding: "normalisation can have a significant impact on pathway analysis results, and PALS offers a framework to further investigate this."
