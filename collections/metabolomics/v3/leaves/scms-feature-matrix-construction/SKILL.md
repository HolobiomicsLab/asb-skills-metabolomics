---
name: scms-feature-matrix-construction
description: Use when raw single-probe SCMS spectra from cell experiments must be
  converted into a cleaned, normalized, aligned feature table (e.g., dataset.csv with
  cells × metabolite species) before machine learning classification or meta-learning.
when_to_use_negative:
- Input is already a pretreated, peak-aligned feature table (e.g., an existing dataset.csv)
  — rerunning pretreatment would double-normalize or drop valid species.
- The goal is bulk metabolomics feature construction; conventional bulk analysis cannot
  evaluate cell heterogeneity, so single-cell-specific pretreatment is the wrong frame.
- Samples come from an ambient SCMS protocol with different acquisition parameters;
  the 10 ppm Geena2 tolerance and established protocol were validated for this Single-probe
  setup.
edam_operation: http://edamontology.org/operation_2941
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: Thermo Xcalibur Qual Browser 3.0
  role: Export peaks from raw LTQ Orbitrap XL spectra for pretreatment
- name: Geena2
  role: Online bioinformatics tool for peak alignment with a mass tolerance of 10
    ppm
- name: MetaboAnalyst 6.0
  role: Principal component analysis of the aligned matrix for exploratory phenotype
    discrimination
- name: MetaPhenotype
  role: Downstream meta-learning classifier consuming the dataset.csv feature matrix
    via dataloade.py
  repo: github.com/songyuan93/MetaPhenotype
- name: Single-probe SCMS
  role: Upstream acquisition technique producing the single-cell metabolomic profiles
    that feed this skill
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:2d033fd8e168a3283c278e8931095bc924ed1114cda30db5377937ddb31d810b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Geena2
  - MetaPhenotype
  - MetaboAnalyst 6.0
  - Single-probe SCMS
  - Thermo Xcalibur Qual Browser 3.0
  edam_operation: http://edamontology.org/operation_2941
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# scms-feature-matrix-construction

## Summary

Builds the pretreated, peak-aligned single-cell mass spectrometry feature matrix (dataset.csv) that serves as the training input for MetaPhenotype and baseline classifiers, using the established SCMS pretreatment protocol on Single-probe-derived spectra.

## When to use

Use when raw single-probe SCMS spectra from cell experiments must be converted into a cleaned, normalized, aligned feature table (e.g., dataset.csv with cells × metabolite species) before machine learning classification or meta-learning.

## When NOT to use

- Input is already a pretreated, peak-aligned feature table (e.g., an existing dataset.csv) — rerunning pretreatment would double-normalize or drop valid species.
- The goal is bulk metabolomics feature construction; conventional bulk analysis cannot evaluate cell heterogeneity, so single-cell-specific pretreatment is the wrong frame.
- Samples come from an ambient SCMS protocol with different acquisition parameters; the 10 ppm Geena2 tolerance and established protocol were validated for this Single-probe setup.

## Inputs

- Raw SCMS spectra acquired on a Thermo LTQ Orbitrap XL mass spectrometer with the Single-probe
- Peak lists exported from Thermo Xcalibur Qual Browser 3.0
- Mass tolerance parameter (10 ppm) for Geena2 peak alignment
- METLIN and HMDB database references for species labeling

## Outputs

- Pretreated, TIC-normalized, peak-aligned cell × species intensity matrix saved as dataset.csv
- Tentatively annotated metabolite species labels (METLIN/HMDB)
- Optional PCA plot for exploratory discrimination check

## How to apply

Export peaks from Thermo LTQ Orbitrap XL raw files via Thermo Xcalibur Qual Browser 3.0, then apply the established SCMS pretreatment protocol: noise subtraction, background subtraction, and total ion count (TIC) normalization on the per-cell spectra. Peak-align the resulting peak lists across cells with Geena2 using a mass tolerance of 10 ppm, and remove species with missing values to produce a cell × species intensity matrix. Tentatively label detected species against METLIN and HMDB databases and optionally inspect global structure with PCA in MetaboAnalyst 6.0. Save the pretreated, aligned matrix as dataset.csv, which the MetaPhenotype repo's dataloade.py reads as input for base.py training runs.

## Related tools

- **Thermo Xcalibur Qual Browser 3.0** (Export peaks from raw LTQ Orbitrap XL spectra for pretreatment)
- **Geena2** (Online bioinformatics tool for peak alignment with a mass tolerance of 10 ppm)
- **MetaboAnalyst 6.0** (Principal component analysis of the aligned matrix for exploratory phenotype discrimination)
- **MetaPhenotype** (Downstream meta-learning classifier consuming the dataset.csv feature matrix via dataloade.py) — github.com/songyuan93/MetaPhenotype
- **Single-probe SCMS** (Upstream acquisition technique producing the single-cell metabolomic profiles that feed this skill)

## Examples

```
python base.py  # reads pretreated, peak-aligned dataset.csv via dataloade.py
```

## Evaluation signals

- After TIC normalization, per-cell total ion counts are equalized across the matrix.
- Peak alignment with Geena2 at 10 ppm tolerance yields a consistent species list across all cells with no duplicated/unmerged features.
- Missing-value removal leaves a matrix that loads as dataset.csv and is parseable by the repo's dataloade.py.
- PCA on the final matrix reproduces the reported exploratory pattern (e.g., IGR39/IGR37 cells not cleanly separable by PCA).
- Downstream MetaPhenotype training on the matrix reaches >90% accuracy on the melanoma cell-pair classification tasks.

## Limitations

- PCA cannot effectively discriminate phenotypically similar cells such as IGR39/IGR37, so PCA should not be used as the final classification step on this matrix.
- Peak labels from METLIN/HMDB are only tentative annotations, not confirmed identities.
- The established pretreatment protocol and 10 ppm tolerance are tied to this Single-probe/LTQ Orbitrap XL workflow; other instruments or acquisition modes may need retuned parameters.
- Ambient SCMS methods yield relatively small numbers of cells, so the resulting matrix may be small and sensitive to overfitting in downstream models.

## Evidence

- [readme] Input file contract: [evidence span withheld — anchor status: external_doc]
