---
name: clustering-accuracy-evaluation-against-cell-phenotypes
description: Use when you have a pretreated SCMS metabolomics feature matrix for cells
  with known phenotype labels (e.g., primary/metastatic pairs, untreated vs drug-treated)
  and need to quantify how well unsupervised clustering alone discriminates the phenotypes
  before building a supervised classifier.
when_to_use_negative:
- When labeled training data are available and the goal is high-accuracy phenotype
  prediction — supervised/meta-learning (MetaPhenotype) reaches >90% accuracy whereas
  unsupervised clustering may be as low as 7-14%
- When the input is raw single-probe SCMS spectra — data must first undergo the established
  pretreatment protocol (noise/background subtraction, TIC normalization, Geena2 peak
  alignment, missing-value removal)
- When cells have highly similar metabolomic profiles (e.g., IGR39/IGR37) and reliable
  phenotype discrimination is required — PCA demonstrably overlaps such pairs
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaboAnalyst 6.0
  role: Web platform used to run both hierarchical clustering analysis (HCA) and principal
    component analysis (PCA) on the pretreated SCMS feature matrices and produce score
    plots
- name: Geena2
  role: Online bioinformatics tool for peak alignment of SCMS peaks with a 10 ppm
    mass tolerance upstream of clustering
- name: MetaPhenotype
  role: Supervised meta-learning model whose >90% classification accuracy is the comparison
    target demonstrating the limits of the unsupervised baseline
  repo: github.com/songyuan93/MetaPhenotype
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:b1e49006404c73fda2138856553f4812031873339816169fb356e63b19036b10
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Geena2
  - MetaPhenotype
  - MetaboAnalyst 6.0
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Clustering Accuracy Evaluation Against Cell Phenotypes

## Summary

A benchmarking skill that applies unsupervised methods (HCA and PCA via MetaboAnalyst 6.0) to pretreated single-cell mass spectrometry (SCMS) metabolomics feature matrices and scores the resulting cluster assignments against known cell phenotypes (primary vs metastatic), establishing the baseline that motivates supervised/meta-learning models like MetaPhenotype.

## When to use

Use when you have a pretreated SCMS metabolomics feature matrix for cells with known phenotype labels (e.g., primary/metastatic pairs, untreated vs drug-treated) and need to quantify how well unsupervised clustering alone discriminates the phenotypes before building a supervised classifier.

## When NOT to use

- When labeled training data are available and the goal is high-accuracy phenotype prediction — supervised/meta-learning (MetaPhenotype) reaches >90% accuracy whereas unsupervised clustering may be as low as 7-14%
- When the input is raw single-probe SCMS spectra — data must first undergo the established pretreatment protocol (noise/background subtraction, TIC normalization, Geena2 peak alignment, missing-value removal)
- When cells have highly similar metabolomic profiles (e.g., IGR39/IGR37) and reliable phenotype discrimination is required — PCA demonstrably overlaps such pairs

## Inputs

- Pretreated SCMS metabolomics feature matrix (cells x metabolic features) with known primary/metastatic phenotype labels, e.g., dataset.csv from the MetaPhenotype repository
- Cell-pair and treatment-state metadata (untreated and vemurafenib-treated IGR39/IGR37 and WM793/WM1205Lu pairs)

## Outputs

- Per-cell HCA cluster assignments and phenotype classification accuracy (%) per cell pair/treatment state
- PCA score plots per cell pair showing primary vs metastatic separation or overlap
- A baseline discrimination report identifying which cell pairs are separable by unsupervised methods

## How to apply

Load the pretreated single-cell feature matrix (peaks exported, noise/background subtracted, TIC-normalized, peak-aligned at 10 ppm mass tolerance via Geena2, missing values removed) into MetaboAnalyst 6.0. Perform hierarchical clustering analysis (HCA), assigning each cell to a primary or metastatic cluster per cell pair, then compute classification accuracy of the cluster assignments against the known phenotype labels (the article reports 72% for drug-treated IGR39/IGR37, but only 7% untreated and 14% treated for WM115/WM266-4-type pairs). In parallel run PCA on the same data and inspect score plots for phenotype separation: WM pairs show visual separation while the IGR39/IGR37 pair shows heavily overlapping metabolomic profiles. The rationale is to expose the ceiling of unsupervised analysis — PCA cannot effectively discriminate phenotypically similar cells like IGR39/IGR37 — which justifies supervised approaches (MetaPhenotype, >90% accuracy with few samples).

## Related tools

- **MetaboAnalyst 6.0** (Web platform used to run both hierarchical clustering analysis (HCA) and principal component analysis (PCA) on the pretreated SCMS feature matrices and produce score plots)
- **Geena2** (Online bioinformatics tool for peak alignment of SCMS peaks with a 10 ppm mass tolerance upstream of clustering)
- **MetaPhenotype** (Supervised meta-learning model whose >90% classification accuracy is the comparison target demonstrating the limits of the unsupervised baseline) — github.com/songyuan93/MetaPhenotype

## Examples

```
python base.py  # MetaPhenotype repo: dataloade.py reads Input data/dataset.csv; run HCA/PCA on the same matrix via MetaboAnalyst 6.0 and score clusters against phenotype labels
```

## Evaluation signals

- HCA-derived cluster/phenotype agreement accuracy should reproduce reported values within reporting tolerance: 72% for drug-treated IGR39/IGR37 cells, ~7% untreated and ~14% treated for the WM cell pairs
- PCA score plots for WM cell pairs should show visual separation of primary and metastatic populations, while IGR39/IGR37 score plots should show heavily overlapping metabolomic profiles
- Every cell in the feature matrix receives exactly one cluster/phenotype assignment per pair, enabling direct accuracy computation against ground-truth labels
- Clustering must be run on the pretreated matrix (post-TIC normalization, post-10 ppm Geena2 alignment); rerunning on raw peaks should be flagged as invalid

## Limitations

- HCA and PCA do not reliably discriminate phenotypes across all groups — accuracy as low as 7% (untreated WM pair) and overlapping IGR39/IGR37 PCA profiles mean unsupervised results cannot be trusted as final phenotype calls
- PCA cannot effectively discriminate phenotypically similar cells (e.g., IGR39/IGR37) with high similarity in metabolomic profiles
- Ambient SCMS methods yield relatively small numbers of cells, so cluster assignments are sensitive to limited sample size

## Evidence

- [results] HCA accuracy on drug-treated vs untreated pairs: [evidence span withheld — anchor status: not_found]
- [abstract] Motivation for supervised alternative: [evidence span withheld — anchor status: external_doc]
- [readme] README input data: [evidence span withheld — anchor status: external_doc]
