---
name: gcms-metabolomics-matrix-preprocessing
description: Use when starting from a raw GC/MS metabolomics intensity matrix (e.g.,
  3226 features x 57 samples) whose columns are sample groups, and a variance-stabilized,
  centered feature table is needed before metabolite identification or biomarker discovery.
when_to_use_negative:
- Input is already a preprocessed (centered and log-transformed) feature table — reapplying
  centering/log would distort intensities.
- Data are not GC/MS metabolomics intensities (e.g., raw instrument files such as
  mzML) — this skill expects a feature intensity matrix, not spectral acquisition
  data.
- The matrix dimensions or sample grouping differ from the pipeline's expectation
  and no adjustment is planned — the validation step (3226 x 57, 27+30 groups) would
  fail.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3825
tools:
- name: pandas
  role: Load and hold the raw GC/MS intensity matrix as a DataFrame and write the
    processed matrix to CSV.
- name: NumPy
  role: Column-mean centering and log transformation of feature intensities.
- name: scikit-learn
  role: Column-wise scaling so metabolite features are on a comparable scale.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo9050103
    title: 'MetPC: Metabolite Pipeline Consisting of Metabolite Identification and
      Biomarker Discovery Under the Control of Two-Dimensional FDR'
  source_content_hash: sha256:2d6840316d49ff044c9da48b2822d6b19fd600ae42008de1f1829d54eb68ac27
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NumPy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3825
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# gcms-metabolomics-matrix-preprocessing

## Summary

Applies the preprocessing stage of the MetPC metabolomics pipeline (DOI 10.3390/metabo9050103) to a raw GC/MS intensity matrix, centering each feature and log-transforming intensities to stabilize variance before downstream metabolite identification and fdr2d-controlled biomarker discovery.

## When to use

Use when starting from a raw GC/MS metabolomics intensity matrix (e.g., 3226 features x 57 samples) whose columns are sample groups, and a variance-stabilized, centered feature table is needed before metabolite identification or biomarker discovery.

## When NOT to use

- Input is already a preprocessed (centered and log-transformed) feature table — reapplying centering/log would distort intensities.
- Data are not GC/MS metabolomics intensities (e.g., raw instrument files such as mzML) — this skill expects a feature intensity matrix, not spectral acquisition data.
- The matrix dimensions or sample grouping differ from the pipeline's expectation and no adjustment is planned — the validation step (3226 x 57, 27+30 groups) would fail.

## Inputs

- Raw GC/MS intensity matrix as a pandas DataFrame (3226 metabolite features x 57 samples)
- Sample group labels distinguishing 27 China and 30 Korea Schisandra chinensis samples

## Outputs

- Preprocessed centered and log-transformed 3226 x 57 data matrix (e.g., 'preprocessed_schisandra_matrix.csv')
- Validation report confirming row/column dimensions and preservation of sample group membership

## How to apply

Load the raw Schisandra chinensis GC/MS intensity matrix (3226 metabolite features x 57 samples, 27 China and 30 Korea) into a pandas DataFrame. Center each feature by subtracting its column mean using NumPy, then apply a log transformation (log2 or log10) to the centered intensities to reduce skewness in the metabolite abundance distribution. Scale each column so features are on a comparable scale, as prescribed by the pipeline's preprocessing steps. Write the result to a CSV (e.g., 'preprocessed_schisandra_matrix.csv') and verify it is exactly 3226 x 57 with both sample groups preserved.

## Related tools

- **pandas** (Load and hold the raw GC/MS intensity matrix as a DataFrame and write the processed matrix to CSV.)
- **NumPy** (Column-mean centering and log transformation of feature intensities.)
- **scikit-learn** (Column-wise scaling so metabolite features are on a comparable scale.)

## Examples

```
import pandas as pd, numpy as np; df = pd.read_csv('raw_schisandra_matrix.csv', index_col=0); proc = np.log2(df - df.mean()); proc.to_csv('preprocessed_schisandra_matrix.csv')
```

## Evaluation signals

- Processed matrix has exactly 3226 rows (metabolites) and 57 columns (samples).
- The 27 China and 30 Korea sample groups are preserved after preprocessing.
- Each feature column has (near-)zero mean after the centering step.
- Log-transformed intensities show reduced skewness relative to the raw centered intensities.
- Output CSV ('preprocessed_schisandra_matrix.csv') round-trips to the same 3226 x 57 shape on reload.

## Limitations

- The article documents preprocessing only as a stage of MetPC feeding metabolite identification (hierarchical model with EM algorithm) and fdr2d biomarker discovery; it does not prescribe a specific log base (log2 vs log10), which is a decision left to the analyst.
- No discussion-section text was available in the extractable document, so failure modes and edge cases for the preprocessing step are not elaborated.

## Evidence

- [abstract] A new bioinformatics tool was developed that includes preprocessing steps and a biomarker discovery procedure for metabolomics data.: "we developed a new bioinformatics tool that includes a few of preprocessing steps and biomarker discovery procedure."
- [abstract] MetPC pipeline stages: "For metabolite identification, we considered a hierarchical statistical model coupled with an Expectation-Maximization (EM) algorithm"
- [other] Processed matrix dimensions and preprocessing: [evidence span withheld — anchor status: not_found]
- [other] Workflow centering step: [evidence span withheld — anchor status: not_found]
