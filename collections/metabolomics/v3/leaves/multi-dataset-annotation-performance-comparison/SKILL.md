---
name: multi-dataset-annotation-performance-comparison
description: Use when evaluating whether MetaboAnnotatoR's fragment matching outperforms
  tools on diverse LC–MS AIF datasets, or when deciding whether a precision (82–92%)
  and recall (82–85%) at rank 1–5 is acceptable for a given biological matrix.
when_to_use_negative:
- The dataset is a single matrix or already has ground-truth annotations for all features
  — this skill is specifically for comparing across multiple datasets/matrices.
- You need annotations for Orbitrap-acquired or SWATH data — the article explicitly
  states these were not tested and future work is needed.
- Your fragment library is very small for nonlipid metabolites — recall will be misleadingly
  low; use a different evaluation approach.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaboAnnotatoR
  role: Primary annotation tool whose performance is compared against MS-DIAL across
    multiple serum datasets using precision/recall at rank 1–5.
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: MS-DIAL
  role: State-of-the-art AIF annotation baseline used as the reference for performance
    comparison.
- name: R
  role: Implementation language for MetaboAnnotatoR; used to run comparisons and compute
    precision/recall metrics.
  repo: https://github.com/gggraca/MetaboAnnotatoR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:c247dcaa84969fd7647e5cb37318d21a3ebc18c147f5be7219c8bee9bcff464b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gggraca/MetaboAnnotatoR
  tools:
  - MS-DIAL
  - MetaboAnnotatoR
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# multi-dataset-annotation-performance-comparison

## Summary

This skill is used to compare annotation performance between MetaboAnnotatoR and MS-DIAL across multiple datasets and matrices, using precision and recall as metrics. It helps understand when high-rank scores (1–5) are reliable and when they fail for datasets dominated by nonlipid metabolites.

## When to use

Use when evaluating whether MetaboAnnotatoR's fragment matching outperforms state-of-the-art tools on diverse LC–MS AIF datasets, or when deciding whether a precision (82–92%) and recall (82–85%) at rank 1–5 is acceptable for a given biological matrix.

## When NOT to use

- The dataset is a single matrix or already has ground-truth annotations for all features — this skill is specifically for comparing across multiple datasets/matrices.
- You need annotations for Orbitrap-acquired or SWATH data — the article explicitly states these were not tested and future work is needed.
- Your fragment library is very small for nonlipid metabolites — recall will be misleadingly low; use a different evaluation approach.

## Inputs

- Multiple LC–MS AIF datasets (centroid mode chromatograms), each with a feature table and expert manual annotations (e.g., 599 manually annotated features across three human serum datasets)
- Fragment libraries from MetaboAnnotatoR (GitHub repo) plus optional custom .txt / .msp libraries
- MS-DIAL annotations on the same datasets for comparison

## Outputs

- Per-dataset precision and recall values at rank 1–5 for MetaboAnnotatoR and MS-DIAL
- Cross-matrix comparison table showing where performance degrades (e.g., 60–89% precision, 10–88% recall for nonlipid metabolites)
- Identified scope limits (Orbitrap mass accuracy, SWATH DIA schemes, small nonlipid fragment libraries)

## How to apply

First, reproduce the annotation pipeline on multiple serum datasets (Lipid+, Lipid−, HILIC+) processed with MetaboAnnotatoR, using the same manually annotated ground truth (599 features). Second, compute precision and recall at rank 1–5 for each dataset, comparing MetaboAnnotatoR results against MS-DIAL outputs. Third, stratify performance by biological matrix (human serum vs. other matrices) to see that precision stays good (82–92%) but recall drops for nonlipid metabolites due to small fragment libraries. Fourth, test on higher mass accuracy instruments (e.g., Orbitraps) and other DIA schemes (e.g., SWATH) to identify scope limits — the article notes these are not yet validated. Rationale: the article explicitly compares against MS-DIAL as state-of-the-art, and shows that performance varies by matrix and library coverage.

## Related tools

- **MetaboAnnotatoR** (Primary annotation tool whose performance is compared against MS-DIAL across multiple serum datasets using precision/recall at rank 1–5.) — https://github.com/gggraca/MetaboAnnotatoR
- **MS-DIAL** (State-of-the-art AIF annotation baseline used as the reference for performance comparison.)
- **R** (Implementation language for MetaboAnnotatoR; used to run comparisons and compute precision/recall metrics.) — https://github.com/gggraca/MetaboAnnotatoR

## Examples

```
install_github("gggraca/MetaboAnnotatoR", dependencies = TRUE, INSTALL_opts = c('--no-multiarch', '--no-lock'))
```

## Evaluation signals

- Precision at rank 1–5 should fall in the 82–92% range for human serum datasets — a large deviation signals a broken pipeline or wrong library.
- Recall at rank 1–5 should fall in the 82–85% range for human serum; recall below 10% or above 88% indicates a matrix dominated by nonlipid metabolites with insufficient fragment library coverage.
- MetaboAnnotatoR results should equal or outperform MS-DIAL on the same datasets; MS-DIAL outperforming MetaboAnnotatoR signals an installation or parameter mismatch.
- Feature counts must match the expert-annotated ground truth (599 features across the three serum datasets); mismatched counts invalidate the comparison.
- Reproduced Mann–Whitney p-values for intensity-vs-annotation-outcome tests must match the reported pattern (p<0.05 for rank-1 vs non-annotated in Lipid− and HILIC+ but not Lipid+, and for rank-1 vs incorrect only in Lipid−).

## Limitations

- Orbitrap-acquired datasets were not tested, although the authors note that annotation on higher mass accuracy instruments is possible.
- Only AIF is supported; other DIA schemes (e.g., SWATH) are future work, so performance comparisons cannot be generalized to SWATH data.
- Performance varies strongly by biological matrix: precision stays 60–89% but recall drops to 10–88% for nonlipid metabolites due to small fragment libraries, so metrics are not transferable across matrices.
- MetaboAnnotatoR focuses on a subset of features rather than the full dataset, so recall is measured only against those features passing statistical filters, not all detected features.
- Installation may fail with 'mzR has been built against a different Rcpp version'; the README suggests setting `Sys.setenv(R_REMOTES_NO_ERRORS_FROM_WARNINGS="true")` or using `--no-multiarch`.

## Evidence

- [abstract] Precision and recall values of 82–92% and 82–85%, respectively, were obtained for features found in the highest-rank scores (1–5): "Precision and recall values of 82–92% and 82–85%, respectively, were obtained for features found in the highest-rank scores (1–5)"
- [abstract] Further validation for other biological matrices and different instrument types showed variable precision (60–89%) and recall (10–88%) particularly for datasets dominated by nonlipid metabolites: "Further validation for other biological matrices and different instrument types showed variable precision (60–89%) and recall (10–88%) particularly for datasets dominated by nonlipid metabolites"
- [discussion] good precision (82–92%) and recall (82–85%) values were obtained for human serum datasets for high-rank scores (1–5).: "good precision (82–92%) and recall (82–85%) values were obtained for human serum datasets for high-rank scores (1–5)."
- [abstract] These results equal or outperform those obtained using MS-DIAL software, the current state of the art for AIF data annotation: "These results equal or outperform those obtained using MS-DIAL software, the current state of the art for AIF data annotation"
- [results] Rank 1 annotations showed higher median intensities than non-annotated features in all datasets except Lipid+ and higher median intensities than incorrectly annotated features only in the Lipid− dataset (Mann–Whitney p < 0.05).: [evidence span withheld — anchor status: not_found]
- [discussion] The annotation of AIF datasets acquired on higher mass accuracy MS instruments (e.g., Orbitraps), although not tested here, is also possible using MetaboAnnotatoR.: "The annotation of AIF datasets acquired on higher mass accuracy MS instruments (e.g., Orbitraps), although not tested here, is also possible using MetaboAnnotatoR."
- [discussion] Although only AIF is currently supported, future developments might expand MetaboAnnotatoR’s application to other DIA schemes, such as SWATH.: "Although only AIF is currently supported, future developments might expand MetaboAnnotatoR’s application to other DIA schemes, such as SWATH."
- [discussion] These showed modest-to-good (60–88%) precision but low recall due to the small size of the fragment libraries, in particular, the nonlipid library.: "These showed modest-to-good (60–88%) precision but low recall due to the small size of the fragment libraries, in particular, the nonlipid library."
- [abstract] We demonstrate the workflow in three human serum datasets containing 599 features manually annotated by experts: "We demonstrate the workflow in three human serum datasets containing 599 features manually annotated by experts"
- [readme] In some environments, some errors might occur durring installation due to some of the package dependencies, being the most common "mzR has been built against a different Rcpp version".: [evidence span withheld — anchor status: external_doc]
