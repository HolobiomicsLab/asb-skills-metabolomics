---
name: tic-normalization-single-cell
description: Use when you have raw per-cell peak lists (m/z, retention time, intensity)
  from single-probe SCMS of single cells and need to remove injection-to-injection
  intensity variation before peak alignment, feature filtering, or ML classification.
when_to_use_negative:
- Input is already a pretreated, TIC-normalized feature matrix (e.g., a Geena2-aligned
  table) — re-normalization would double-correct intensities.
- Data are bulk metabolomics rather than single-probe SCMS single-cell peak lists
  — the per-cell TIC normalization assumes each row is one cell.
- Peaks fall outside the 150–1500 m/z range or were not exported from Xcalibur Qual
  Browser 3.0 — the alignment step in Geena2 expects those bounds.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Thermo Xcalibur Qual Browser 3.0
  role: Exports the raw per-cell peak lists (m/z, retention time, intensity) that
    serve as input for TIC normalization.
- name: Geena2
  role: Aligns the TIC-normalized peaks across cells (10 ppm tolerance, 150–1500 m/z)
    into a consistent feature matrix.
- name: MetaPhenotype
  role: Consumes the TIC-normalized, aligned peak-intensity matrix as training/evaluation
    data for 1D-CNN meta-learning classification.
  repo: github.com/songyuan93/MetaPhenotype
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:d297d17721da5ff070aca7eb25e628134cfaa6d6fb57bb8d22be9c2b04361fee
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Geena2
  - MetaPhenotype
  - R
  - Thermo Xcalibur Qual Browser 3.0
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# tic-normalization-single-cell

## Summary

Normalizes Single-probe SCMS peak-intensity tables by total ion current (TIC) so that individual melanoma cells analyzed across separate injections become quantitatively comparable before machine learning.

## When to use

Use when you have raw per-cell peak lists (m/z, retention time, intensity) from single-probe SCMS of single cells and need to remove injection-to-injection intensity variation before peak alignment, feature filtering, or ML classification.

## When NOT to use

- Input is already a pretreated, TIC-normalized feature matrix (e.g., a Geena2-aligned table) — re-normalization would double-correct intensities.
- Data are bulk metabolomics rather than single-probe SCMS single-cell peak lists — the per-cell TIC normalization assumes each row is one cell.
- Peaks fall outside the 150–1500 m/z range or were not exported from Xcalibur Qual Browser 3.0 — the alignment step in Geena2 expects those bounds.

## Inputs

- Raw per-cell peak lists (m/z, retention time, intensity) exported from Thermo Xcalibur Qual Browser 3.0
- Single-probe SCMS data from primary/metastatic melanoma cell lines (e.g., IGR39/IGR37 pairs, with/without vemurafenib)
- In-house R background-subtraction script

## Outputs

- TIC-normalized per-cell peak-intensity table
- Geena2-aligned peak matrix (10 ppm tolerance, 150–1500 m/z)
- Pretreated matrix with species missing >50% of values removed, ready for PCA and MetaPhenotype training

## How to apply

Export each analyzed cell's peak list (m/z, RT, intensity) from Thermo Xcalibur Qual Browser 3.0. Apply noise subtraction by removing species with relative intensity below 3×10^3, then subtract background species using an in-house R script. Normalize each cell's peak intensities by its total ion current so cells are comparable across injections. Align the TIC-normalized peaks across cells with Geena2 using a 10 ppm mass tolerance over the 150–1500 m/z range, and finally drop metabolic species missing values in more than 50% of cells to produce the pretreated peak-intensity matrix fed to PCA and MetaPhenotype.

## Related tools

- **Thermo Xcalibur Qual Browser 3.0** (Exports the raw per-cell peak lists (m/z, retention time, intensity) that serve as input for TIC normalization.)
- **Geena2** (Aligns the TIC-normalized peaks across cells (10 ppm tolerance, 150–1500 m/z) into a consistent feature matrix.)
- **MetaPhenotype** (Consumes the TIC-normalized, aligned peak-intensity matrix as training/evaluation data for 1D-CNN meta-learning classification.) — github.com/songyuan93/MetaPhenotype

## Examples

```
Rscript background_subtraction.R --input dataset.csv && python base.py
```

## Evaluation signals

- Each cell's peak intensities should sum to the same total (TIC) after normalization — verify by re-summing the normalized matrix.
- No species with relative intensity below 3×10^3 should remain in the normalized table.
- The Geena2 alignment output must contain only peaks in the 150–1500 m/z range matched at 10 ppm tolerance.
- The final matrix should contain no metabolic species with values missing in more than 50% of the cells.
- Running PCA on the pretreated matrix should reproduce the article's reported discrimination of primary vs metastatic phenotypes (e.g., separation seen for A375 pair).

## Limitations

- PCA on TIC-normalized data cannot effectively discriminate phenotypes with high metabolomic similarity such as IGR39/IGR37 cells; downstream ML (MetaPhenotype) is needed in that case.
- Ambient SCMS methods yield relatively small numbers of cells, so TIC normalization operates on limited sample sizes.
- Species are only tentatively labeled using METLIN and HMDB, so the normalized matrix carries putative assignments only.

## Evidence

- [readme] Readme confirms input data are the SCMS dataset.csv and MetaPhenotype model code: [evidence span withheld — anchor status: external_doc]
