---
name: log-transformation-with-column-scaling
description: Use when a raw GC/MS intensity matrix (e.g., metabolites x samples) shows
  skewed feature abundance distributions and must be preprocessed for metabolite identification
  or biomarker discovery.
when_to_use_negative:
- Input is already a normalized or preprocessed feature table produced by the MetPC
  pipeline's preprocessing steps
- Data are not metabolomics intensities from an instrument such as GC/MS and would
  be distorted by log transformation (e.g., count data with zeros requiring different
  handling)
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: pandas
  role: Loads and manipulates the raw GC/MS intensity matrix as a DataFrame during
    preprocessing
- name: NumPy
  role: Performs column-mean centering and log transformation of feature intensities
- name: scikit-learn
  role: Scales columns so features are on a comparable scale per the pipeline's preprocessing
    steps
- name: MetPC
  role: Pipeline whose preprocessing steps this transformation implements prior to
    metabolite identification and biomarker discovery
  repo: jjs3098/CNU-Bioinformatics-Lab
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo9050103
    title: 'MetPC: Metabolite Pipeline Consisting of Metabolite Identification and
      Biomarker Discovery Under the Control of Two-Dimensional FDR'
  source_content_hash: sha256:bf5dac1a6b6994385da7926eb5a0cede969ed45e5afd33ee322652b4ad28a516
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jjs3098/CNU-Bioinformatics-Lab
  tools:
  - MetPC
  - NumPy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Log-transformation with column scaling

## Summary

Preprocessing routine for GC/MS metabolomics intensity matrices that centers each metabolite feature, applies a log transformation to stabilize variance and reduce skewness, and scales columns so features are on a comparable scale before downstream biomarker discovery, as used in the MetPC pipeline.

## When to use

Use when a raw GC/MS intensity matrix (e.g., metabolites x samples) shows skewed feature abundance distributions and must be preprocessed for metabolite identification or biomarker discovery.

## When NOT to use

- Input is already a normalized or preprocessed feature table produced by the MetPC pipeline's preprocessing steps
- Data are not metabolomics intensities from an instrument such as GC/MS and would be distorted by log transformation (e.g., count data with zeros requiring different handling)

## Inputs

- Raw GC/MS metabolomics intensity matrix (features x samples), e.g., a 3226 x 57 Schisandra chinensis matrix
- Sample group labels (e.g., 27 China vs. 30 Korea) preserved through preprocessing

## Outputs

- Processed intensity matrix of identical dimensions (3226 rows x 57 columns), e.g., written to 'preprocessed_schisandra_matrix.csv'
- Centered, log-transformed, and column-scaled feature values ready for biomarker discovery

## How to apply

Load the raw GC/MS intensity matrix (e.g., 3226 metabolite features x 57 samples) into a pandas DataFrame. Center each feature by subtracting its column mean using NumPy, then apply a log transformation (log2 or log10) to the centered intensities to stabilize variance and reduce skewness in the metabolite abundance distribution. Scale each column so features are on a comparable scale, as prescribed by the preprocessing steps of the MetPC pipeline. Finally, verify the processed matrix retains its original dimensions (3226 x 57) and that sample group labels (e.g., 27 China and 30 Korea Schisandra samples) are preserved for downstream fdr2d-controlled biomarker testing.

## Related tools

- **pandas** (Loads and manipulates the raw GC/MS intensity matrix as a DataFrame during preprocessing)
- **NumPy** (Performs column-mean centering and log transformation of feature intensities)
- **scikit-learn** (Scales columns so features are on a comparable scale per the pipeline's preprocessing steps)
- **MetPC** (Pipeline whose preprocessing steps this transformation implements prior to metabolite identification and biomarker discovery) — jjs3098/CNU-Bioinformatics-Lab

## Examples

```
df = pd.read_csv('raw_schisandra_matrix.csv'); centered = df - df.mean(axis=0); log_centered = np.log2(centered - centered.min(axis=0) + 1); scaled = sklearn.preprocessing.scale(log_centered, axis=0)
```

## Evaluation signals

- Processed matrix must have exactly the same dimensions as the input (e.g., 3226 rows x 57 columns)
- All processed values must be finite and non-missing after log transformation of centered intensities
- Distributional check: feature abundance skewness is reduced relative to the raw matrix
- Sample group composition is preserved (27 China and 30 Korea samples remain intact and correctly labeled)
- Output file 'preprocessed_schisandra_matrix.csv' is written and loadable with the expected schema

## Limitations

- Log transformation of centered intensities can fail on non-positive values (zeros or negatives introduced by centering), so transformation base and ordering must match the pipeline specification
- The article's discussion section was not extractable, so scaling specifics beyond the stated preprocessing steps could not be verified

## Evidence

- [abstract] abstract: "we developed a new bioinformatics tool that includes a few of preprocessing steps and biomarker discovery procedure."
- [other] task_004 workflow: [evidence span withheld — anchor status: not_found]
- [other] task_004 finding: [evidence span withheld — anchor status: not_found]
