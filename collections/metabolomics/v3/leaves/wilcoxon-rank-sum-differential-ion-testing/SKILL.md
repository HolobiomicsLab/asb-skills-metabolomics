---
name: wilcoxon-rank-sum-differential-ion-testing
description: Use when a peak table matrix of spatial metabolomics pixels (X/Y coordinates
  + m/z intensity columns) needs ion-level tissue-vs-background classification, e.g.
  to quantify the proportion of tissue-enriched ions.
when_to_use_negative:
- Data is raw vendor imaging files rather than an SMQVP-format peak table matrix with
  X/Y coordinate columns plus m/z intensity columns
- The goal is pixel-level background noise removal or noise ion filtering (QVP filter
  steps), not ion-level differential tissue-vs-background expression testing
- Sample groups are paired/dependent measurements, where a rank-sum test on independent
  samples is not the correct statistic
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SMQVP
  role: Web application whose QVP3 / Ion Expression Proportion module performs the
    tissue-vs-background ion proportion analysis
  repo: github.com/mzlab-research/SMQVP
- name: R
  role: Implementation environment for the SMQVP Shiny application that delivers the
    analysis
  repo: github.com/mzlab-research/SMQVP
- name: scipy.stats.ranksums
  role: Per-ion Wilcoxon rank-sum test between tissue and background pixel intensity
    vectors
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:a55ba08a90821c6e11cda2b8996b80187aef768c288482e4c623bbcb959a1407
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SMQVP
  - scipy.stats.ranksums
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# wilcoxon-rank-sum-differential-ion-testing

## Summary

Non-parametric differential testing (Wilcoxon rank-sum) comparing per-ion pixel intensities between user-delineated tissue and background regions of a spatial metabolomics peak table, to compute the tissue-enriched ion expression proportion (QVP3 module of SMQVP).

## When to use

Use when a peak table matrix of spatial metabolomics pixels (X/Y coordinates + m/z intensity columns) needs ion-level tissue-vs-background classification, e.g. to quantify the proportion of tissue-enriched ions.

## When NOT to use

- Data is raw vendor imaging files rather than an SMQVP-format peak table matrix with X/Y coordinate columns plus m/z intensity columns
- The goal is pixel-level background noise removal or noise ion filtering (QVP filter steps), not ion-level differential tissue-vs-background expression testing
- Sample groups are paired/dependent measurements, where a rank-sum test on independent samples is not the correct statistic

## Inputs

- Peak table matrix CSV/tabular file: first two columns = X and Y pixel coordinates, subsequent columns = m/z ion intensities per pixel
- AFADESI-based mouse brain dataset (53,812 pixels, 2,654 ion peaks) from https://ngdc.cncb.ac.cn/omix/release/OMIX009541
- User-defined tissue vs background region assignment per pixel (lasso-delineated on the median pixel intensity image)

## Outputs

- Per-ion table (e.g. tissue_enriched_ions.csv) with fold change, Wilcoxon p-value, and tissue-enriched flag for each of the 2,654 ions
- Overall tissue-enriched ion proportion (expected ~0.37 for the AFADESI mouse brain reference dataset)

## How to apply

Load the peak table matrix (e.g. 53,812 pixels x 2,654 ion peaks from the AFADESI mouse brain dataset, OMIX009541) with the first two columns holding X/Y pixel coordinates. Partition pixels into tissue and background groups per the lasso-delineated regions selected on the median pixel intensity image (five tissue and five background regions in the SMQVP demonstration). For each ion column, run a Wilcoxon rank-sum test between tissue and background pixel intensities (scipy.stats.ranksums, which returns the z-score and two-sided p-value) and compute the fold change of the median tissue intensity over the median background intensity. Classify an ion as tissue-enriched when fold change > 1 AND p < 0.05, then report the fraction of the 2,654 detected ions meeting both criteria. In the SMQVP paper's AFADESI mouse brain dataset this Ion Expression Proportion (QVP3) analysis classified 37% of detected ions as tissue-enriched, which serves as the reproduction target.

## Related tools

- **SMQVP** (Web application whose QVP3 / Ion Expression Proportion module performs the tissue-vs-background ion proportion analysis) — github.com/mzlab-research/SMQVP
- **R** (Implementation environment for the SMQVP Shiny application that delivers the analysis) — github.com/mzlab-research/SMQVP
- **scipy.stats.ranksums** (Per-ion Wilcoxon rank-sum test between tissue and background pixel intensity vectors)

## Examples

```
from scipy.stats import ranksums; z, p = ranksums(tissue_intensities, background_intensities); enriched = (fc > 1) & (p < 0.05)
```

## Evaluation signals

- Reproduced tissue-enriched ion proportion falls in [0.35, 0.39], matching the paper's reported 37% of detected ions
- Every ion row in the output carries fold change, p-value, and tissue-enriched classification fields
- Classification invariant: an ion is flagged only when fold change > 1 AND p < 0.05
- Output ion count equals the input peak count (2,654 ions for the reference AFADESI dataset)
- Per-ion p-values are two-sided Wilcoxon rank-sum p-values computed over the full 53,812-pixel intensity vector split by the tissue/background assignment

## Limitations

- Region delineation (tissue vs background grouping of the 53,812 pixels) is user-defined; different lasso selections change the computed proportion
- No software version specified for SMQVP and no changelog available for the repository, complicating exact reproduction
- Fold change defined on median intensities with an absolute threshold (fold change > 1) does not account for magnitude of effect beyond p < 0.05

## Evidence

- [other] Workflow step: per-ion Wilcoxon test and threshold: [evidence span withheld — anchor status: not_found]
- [other] QVP3 target value: [evidence span withheld — anchor status: not_found]
- [intro] Region selection via lasso tool: "Use the lasso tool to delineate background and tissue regions on the interaction image"
- [readme] Input data format definition: "The input for SMQVP is a peak table matrix with the following structure: - First two columns: X and Y coordinates of the pixels"
- [readme] Reference dataset dimensions: "The data includes 53,812 pixels and 2,654 ion peaks."
