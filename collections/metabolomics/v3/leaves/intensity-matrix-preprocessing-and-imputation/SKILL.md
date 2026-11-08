---
name: intensity-matrix-preprocessing-and-imputation
description: Use when a peak-intensity matrix (rows = peak features, columns = samples,
  with a group line or experimental_design dict) contains zero intensities or unnormalised
  scale, and must be prepared for PALS pathway activity scoring.
when_to_use_negative:
- Input is already a normalised, imputed intensity matrix with no zero values and
  the downstream method expects raw data as-is
- Data are proteomics or transcriptomics features mapped via UniProt/ENSEMBL IDs where
  the compound-imputation defaults and KEGG/ChEBI annotation assumptions do not apply
  without adjustment
- Peak features carry no compound annotations at all — unannotated peaks are simply
  excluded from pathway analysis, so preprocessing alone will not make them usable
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PALS
  role: Hosts the imputation, log-2 transformation and standardisation logic applied
    when the intensity matrix is loaded (run_pals.py / DataSource)
  repo: github.com/glasgowcompbio/PALS
- name: SciPy
  role: preprocessing module used to standardise the log-2 transformed intensity matrix
    to zero mean and unit variance
- name: pandas
  role: int_df and annotation_df DataFrames are passed directly to DataSource when
    PALS is used programmatically
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:ecbcf2559f7ca358dadaad1712825ee9bb61c0dc65fa00719cbe0220fbd5f3f5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PALS
  - Python
  - SciPy
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# intensity-matrix-preprocessing-and-imputation

## Summary

Preprocessing pipeline in PALS that imputes zero intensities in a peak-intensity matrix, log-2 transforms it, and standardises it (zero mean, unit variance) with SciPy so it is ready for PLAGE/ORA/GSEA pathway ranking.

## When to use

Use when a peak-intensity matrix (rows = peak features, columns = samples, with a group line or experimental_design dict) contains zero intensities or unnormalised scale, and must be prepared for PALS pathway activity scoring.

## When NOT to use

- Input is already a normalised, imputed intensity matrix with no zero values and the downstream method expects raw data as-is
- Data are proteomics or transcriptomics features mapped via UniProt/ENSEMBL IDs where the compound-imputation defaults and KEGG/ChEBI annotation assumptions do not apply without adjustment
- Peak features carry no compound annotations at all — unannotated peaks are simply excluded from pathway analysis, so preprocessing alone will not make them usable

## Inputs

- Peak-intensity CSV matrix (row_id column plus per-sample columns; second line maps samples to groups, e.g. beer1, beer2)
- Annotation CSV matrix mapping peak IDs to KEGG or ChEBI compound IDs (columns row_id, entity_id)
- experimental_design dictionary specifying groups and case/control comparisons (programmatic use)
- --min_replace imputation value (default 5000)

## Outputs

- Imputed, log-2 transformed and standardised intensity matrix (zero mean, unit variance across samples) held in a pandas DataFrame
- DataSource object ready to be passed to PLAGE, ORA, or GSEA for pathway ranking

## How to apply

Load the intensity CSV (row 1 = peak/sample IDs, row 2 = group labels) into a pandas DataFrame; when imputation is triggered, if all samples in a single experimental factor have zero intensities they are replaced by the minimum intensity value, controllable via --min_replace (default 5000), while partially-zero factors get the mean of non-zero samples in that factor. The matrix is then log-2 transformed and standardised with the SciPy preprocessing module so intensities have zero mean and unit variance across samples. Peaks lacking compound annotations (KEGG or ChEBI IDs in the annotation CSV) are excluded downstream, so prepare a two-column annotation matrix (peak ID -> entity_id) alongside the intensity matrix. The imputed/transformed matrix feeds directly into DataSource for PLAGE, ORA, or GSEA ranking. Correctness is judged by checking no zero intensities remain within factors after imputation and that the standardised matrix has zero mean and unit variance per feature.

## Related tools

- **PALS** (Hosts the imputation, log-2 transformation and standardisation logic applied when the intensity matrix is loaded (run_pals.py / DataSource)) — github.com/glasgowcompbio/PALS
- **SciPy** (preprocessing module used to standardise the log-2 transformed intensity matrix to zero mean and unit variance)
- **pandas** (int_df and annotation_df DataFrames are passed directly to DataSource when PALS is used programmatically)

## Examples

```
python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control Stage_2/Control --min_replace 5000
```

## Evaluation signals

- No zero intensities remain within any experimental factor after imputation: fully-zero factors replaced by min_replace (default 5000), partially-zero factors replaced by the factor's non-zero mean
- Standardised matrix has zero mean and unit variance across samples (verifiable with SciPy preprocessing output)
- Matrix passes a two-group-column CSV schema check: row_id first column and a 'group' second line mapping samples to factors
- Annotation matrix maps every retained peak to at least one KEGG or ChEBI entity_id; peaks without annotations are excluded from downstream pathway analysis
- CLI run accepts --min_replace 5000 (default) and produces the pathway ranking output with unq_pw_F, tot_ds_F, and F_coverage columns, confirming preprocessing completed

## Limitations

- Imputation is rule-based (min-replace / mean-replace of zeros); no alternative imputation strategies are documented in the README
- Only zero-valued intensities are imputed — missing peaks represented as NaN or absent rows are not handled by this step
- Peaks without compound annotations are silently excluded from pathway analysis, so the effective feature set after preprocessing may be much smaller than the input matrix
- No software version numbers or changelog are stated for PALS, so preprocessing behaviour may drift across repository versions

## Evidence

- [readme] Data imputation is performed to the intensity matrix when it is loaded: if all of the samples in a single experimental factor have intensities of zero these are replaced by the minimum intensity value: "Data imputation is performed to the intensity matrix when it is loaded: if all of the samples in a single experimental factor have intensities of zero these are replaced by the minimum intensity value"
- [readme] The data is subsequently transformed to log-2 base and standardised using the preprocessing module in Scipy such that the intensity matrix has a zero mean and unit variance across the samples.: "The data is subsequently transformed to log-2 base and standardised using the preprocessing module in Scipy such that the intensity matrix has a zero mean and unit variance across the samples."
- [readme] min_replace: The minimum intensity value for data imputation, e.g. --min_replace 5000. Defaults to 5000.: "min_replace: The minimum intensity value for data imputation, e.g. `--min_replace 5000`. Defaults to 5000."
- [methods] peaks that do not have annotations will not be used for pathway analysis: "users also provide a list of compound annotations assigned to peak features (peaks that do not have annotations will not be used for pathway analysis)"
