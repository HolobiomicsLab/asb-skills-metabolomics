---
name: ms2-spectral-vectorization-memo
description: Use when you need to compare metabolomics samples by their overall MS2
  fragmentation content rather than aligned features — e.g. chemodiverse samples with
  poor feature overlap, strong RT shifts, or different LC/MS methods — or to flag
  spectrally dissimilar extracts for novelty prioritization.
when_to_use_negative:
- Your samples have rich feature overlap and standard retention-time alignment suffices
  — MEMO suits chemodiverse samples with poor feature overlap or strong RT shifts,
  so applying it there adds complexity without benefit.
- You need per-feature or annotated-structure comparison (e.g. FC based on non-annotated
  features) rather than a whole-sample MS2-spectrum-level comparison; the SC operates
  on unannotated peaks/neutral losses and does not use the feature alignment table.
- Your data are MS1-only with no MS2 fragmentation spectra — MEMO requires fragment
  spectra (MS2 peaks and precursor neutral losses) to build fingerprints.
edam_operation: http://edamontology.org/operation_0394
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MEMO
  role: Computes Ms2 basEd saMple vectOrization fingerprints and the RT-agnostic sample
    dissimilarity matrix from MS2 spectra.
  repo: https://github.com/mandelbrot-project/memo
- name: Inventa
  role: Consumes the MEMO vectorized matrix as the vectorized_data_filename input
    and mines it with outlier detection to compute the Similarity Component (SC).
  repo: https://github.com/luigiquiros/inventa
- name: GNPS
  role: Source of the FBMN job whose MS2 spectra and standard metadata format feed
    the MEMO vectorization.
- name: scikit-bio
  role: Dependency in the Inventa conda environment supporting the vectorized-matrix
    computation pipeline.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:7ed46b6e8795bcf56cca76d27c65138ceabdcee69862325b9c2f82f79b36f5ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mandelbrot-project/memo
  tools:
  - GNPS
  - Inventa
  - MEMO
  - scikit-bio
  edam_operation: http://edamontology.org/operation_0394
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# ms2-spectral-vectorization-memo

## Summary

Vectorizes metabolomics samples into MS2-based fingerprints using the MEMO metric — counting MS2 peaks and neutral losses per sample — then compares samples via an RT-agnostic dissimilarity matrix to detect spectrally dissimilar extracts (outliers) feeding the Inventa Similarity Component (SC).

## When to use

Use when you need to compare metabolomics samples by their overall MS2 fragmentation content rather than aligned features — e.g. chemodiverse samples with poor feature overlap, strong RT shifts, or different LC/MS methods — or to flag spectrally dissimilar extracts for novelty prioritization.

## When NOT to use

- Your samples have rich feature overlap and standard retention-time alignment suffices — MEMO suits chemodiverse samples with poor feature overlap or strong RT shifts, so applying it there adds complexity without benefit.
- You need per-feature or annotated-structure comparison (e.g. FC based on non-annotated features) rather than a whole-sample MS2-spectrum-level comparison; the SC operates on unannotated peaks/neutral losses and does not use the feature alignment table.
- Your data are MS1-only with no MS2 fragmentation spectra — MEMO requires fragment spectra (MS2 peaks and precursor neutral losses) to build fingerprints.

## Inputs

- MS2 fragmentation spectra from a GNPS FBMN job (feature-based molecular networking)
- MEMO vectorized dissimilarity matrix (vectorized_data_filename, e.g. Celastraceae_memomatrix.csv) or MEMO package fingerprints
- Metadata table (GNPS format with ATTRIBUTE_Species and ATTRIBUTE_Organe headers) linking samples to organisms

## Outputs

- MS2 fingerprint vectors per sample (peak/neutral-loss occurrence counts)
- MEMO dissimilarity matrix aligning all samples
- SC (Similarity Component) score per extract (1 = outlier, 0 = inlier) written to a .tsv results file

## How to apply

Load each sample's GNPS feature-based molecular networking (FBMN) MS2 spectra and count the occurrences of MS2 peaks and neutral losses (to the precursor) per sample, producing an MS2 fingerprint vector per sample. Align these fingerprints across the sample set to build a MEMO dissimilarity matrix (in Inventa, the vectorized_data_filename, e.g. Celastraceae_memomatrix.csv); the metric is retention-time agnostic, so no feature alignment across samples is needed. In the Inventa SC workflow, feed the vectorized matrix into multiple unsupervised outlier detection algorithms — LOF, One-Class SVM, and Isolation Forest from scikit-learn — and assign SC = 1 to extracts classified as outliers by any algorithm, SC = 0 to inliers. The SC is then combined with the FC, LC, and CC components (each weighted by a user-defined modulating factor wn) into the final priority rank written as a .tsv file.

## Related tools

- **MEMO** (Computes Ms2 basEd saMple vectOrization fingerprints and the RT-agnostic sample dissimilarity matrix from MS2 spectra.) — https://github.com/mandelbrot-project/memo
- **Inventa** (Consumes the MEMO vectorized matrix as the vectorized_data_filename input and mines it with outlier detection to compute the Similarity Component (SC).) — https://github.com/luigiquiros/inventa
- **GNPS** (Source of the FBMN job whose MS2 spectra and standard metadata format feed the MEMO vectorization.)
- **scikit-bio** (Dependency in the Inventa conda environment supporting the vectorized-matrix computation pipeline.)

## Examples

```
conda install -c conda-forge scikit-bio && python -c "from memo import memo_matrix; M = memo_matrix(gnps_job, metadata)"
```

## Evaluation signals

- Check the MEMO output is a sample-vs-sample dissimilarity matrix whose row/column count equals the number of samples in the metadata table.
- Confirm each fingerprint vector encodes counts of MS2 peaks and neutral losses (to the precursor), not annotated compound features.
- In the Inventa SC output, verify SC = 1 only for extracts flagged as outliers by at least one of LOF, One-Class SVM, or Isolation Forest, and SC = 0 for inliers.
- Replication check: in the Celastraceae proof of concept, Pristimera indica roots extract must receive SC = 1 consistent with its top priority rank.
- Verify the final results are written as a tab-separated .tsv file in the results folder containing SC values alongside sample identifiers.

## Limitations

- MEMO is retention-time agnostic — it deliberately discards RT information, so it cannot be used where chromatographic retention behavior is the discriminating signal.
- The SC score is binary (0/1) and depends on the choice and behavior of unsupervised outlier detection algorithms; borderline extracts may be inconsistently flagged.
- MEMO suits chemodiverse samples — with poor feature overlap or strong RT shifts — but the README notes filtering steps (e.g. removing peaks/losses from blanks) must be applied to fingerprints before visualization/comparison.

## Evidence

- [readme] SC outlier detection on MEMO matrix: "The matrix is mined through multiple outlier detection machine learning algorithms to highlight spectrally dissimilar extracts (outliers). An SC value of ‘1’ implies the extract is classified as an"
- [readme] MEMO fingerprint definition: [evidence span withheld — anchor status: external_doc]
- [readme] MEMO applicability scope: [evidence span withheld — anchor status: external_doc]
- [methods] Inventa SC input specification: "`vectorized_data_filename`: [MEMO](https://github.com/mandelbrot-project/memo) package output format."
- [methods] SC via unsupervised outlier algorithms: [evidence span withheld — anchor status: not_found]
- [results] Pristimera indica SC=1 validation: [evidence span withheld — anchor status: not_found]
