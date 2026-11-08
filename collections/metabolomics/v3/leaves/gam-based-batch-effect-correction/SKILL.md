---
name: gam-based-batch-effect-correction
description: Use when an untargeted metabolomics feature table (raw intensities, samples
  × features, with QC runs and batch labels) shows drift or batch effects and outlier-containing
  features require robust normalization before differential abundance testing.
when_to_use_negative:
- The dataset exceeds the capacity of the multivariate SERFF Shiny web service — SERFF
  failed to complete on the 618-run × 8913-feature FAME dataset, so use Metanorm's
  feature-level methods instead
- Multiple testing correction is required for the final inference — the article explicitly
  leaves multiple testing correction out of scope
- Automated outlier detection/removal prior to normalization is expected — the article
  notes this is nontrivial and not provided
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: Metanorm R package
  role: Implements tGAM, rGAM, and rLOESS normalization (plus QC-RLSC/QC-RSC comparators)
    with batchwise processing, QCcheck, PCA plots, and parallelized high-throughput
    execution
  repo: https://github.com/UGent-LIMET/Metanorm
- name: R
  role: Runtime environment for the Metanorm package (R >= 4.4.0 required) in which
    normalization and diagnostic plotting are executed
  repo: https://github.com/UGent-LIMET/Metanorm
- name: SERFF
  role: Multivariate normalization comparator; attempted via its Shiny web interface
    (https://slfan.shinyapps.io/ShinySERRF/) but failed on the FAME dataset due to
    capacity limits
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:cdfcbc8229186f64c70b8f7f89ee01dfffb00f94b388a6c92f71164a8602f4d7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/UGent-LIMET/Metanorm
  tools:
  - Metanorm
  - Metanorm R package
  - R
  - SERFF
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# gam-based-batch-effect-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Batch-effect/drift correction for metabolomics feature tables using generalized additive model-based normalization (tGAM/rGAM) and robust LOESS (rLOESS), implemented in the Metanorm R package, which models signal drift across sample runs and tolerates outlier metabolite intensities that degrade QC-sample-based methods.

## When to use

Use when an untargeted metabolomics feature table (raw intensities, samples × features, with QC runs and batch labels) shows drift or batch effects and outlier-containing features require robust normalization before differential abundance testing.

## When NOT to use

- The dataset exceeds the capacity of the multivariate SERFF Shiny web service — SERFF failed to complete on the 618-run × 8913-feature FAME dataset, so use Metanorm's feature-level methods instead
- Multiple testing correction is required for the final inference — the article explicitly leaves multiple testing correction out of scope
- Automated outlier detection/removal prior to normalization is expected — the article notes this is nontrivial and not provided

## Inputs

- Unnormalized numerical intensity matrix (rows = compounds, columns = sample runs), natural-log transformed
- Vector of sample-run types ('QC' vs other) for each column
- Vector of batch labels for each sample run (for batchwise normalization)
- Group/phenotype labels (e.g. healthy-weight n=337 vs overweight/obesity n=99) for downstream differential abundance testing

## Outputs

- Normalized intensity matrix (same dimensions as input)
- Pre- vs post-normalization intensity-vs-order plots per compound (written to plotdir)
- PCA score plots before and after normalization, labeled by batch
- Downstream DAM lists (e.g. two-sided Wilcoxon rank sum tests, p < 0.05) and pairwise DAM overlap percentages across methods

## How to apply

Natural-log transform all intensities, then call the Metanorm package's metanorm() function on the numerical matrix (rows = compounds, columns = samples) with model='tGAM' (recommended default; rGAM and rLOESS are faster alternatives), supplying the sample-type vector ('QC' vs other) and the batch vector to normalize batchwise; set QCcheck=TRUE so Metanorm checks for discrepancies between QC and biological samples, and optionally supply plotdir for diagnostic output. The rationale is that occasional disproportionately high or low metabolite intensities (outliers) deteriorate existing QC-sample-based methods (QC-RLSC, QC-RSC), whereas the (G)AM approaches use flexible nonlinear modeling, hypothesis testing, and observation weighting to jointly or selectively use QC and biological samples. Because the problem is pleasingly parallel, Metanorm exploits parallelization for efficient high-throughput normalization of large matrices (e.g. 618 runs × 8913 features). Optionally run QC-RLSC (with QConly=TRUE) as a comparator and benchmark downstream DAM overlap between methods.

## Related tools

- **Metanorm R package** (Implements tGAM, rGAM, and rLOESS normalization (plus QC-RLSC/QC-RSC comparators) with batchwise processing, QCcheck, PCA plots, and parallelized high-throughput execution) — https://github.com/UGent-LIMET/Metanorm
- **R** (Runtime environment for the Metanorm package (R >= 4.4.0 required) in which normalization and diagnostic plotting are executed) — https://github.com/UGent-LIMET/Metanorm
- **SERFF** (Multivariate normalization comparator; attempted via its Shiny web interface (https://slfan.shinyapps.io/ShinySERRF/) but failed on the FAME dataset due to capacity limits)

## Examples

```
normdat <- metanorm(rawdata, model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/metanormOut/")
```

## Evaluation signals

- Pre- vs post-normalization PCA score plots labeled by batch show diminished batch effects
- Pairwise DAM overlap between robust methods (rLOESS, rGAM, tGAM) falls in the reported 79–87% range, while QC-RLSC shows weaker 55–59% agreement
- Individual compound pre/post-normalization intensity-vs-order plots (from plotdir) show corrected signal drift in both QC and biological samples
- Replicate concordance improves and drift/batch effects are reduced relative to QC-RLSC/QC-RSC, as demonstrated in the article's simulation and experimental datasets
- QCcheck output confirms no unexpected discrepancies between QC and biological samples

## Limitations

- Outlier treatment is feature-level only; multivariate outlier detection is not implemented
- Multiple testing correction is not applied and is left as essential downstream work
- SERFF and TIGER multivariate methods are not benchmarked within Metanorm, and SERFF failed on the 618 × 8913 FAME dataset (server capacity exceeded)
- Unexpected differences between QC samples and biological samples require due analyst scrutiny rather than automated handling
- Using reduced QC-sample RSD or increased DAM counts as success proxies is a known benchmarking pitfall

## Evidence

- [readme] README example usage: [evidence span withheld — anchor status: external_doc]
- [readme] README QC vs biological samples: [evidence span withheld — anchor status: external_doc]
