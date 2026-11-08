---
name: between-replicate-dispersion-quantification
description: Use when an LC-MS metabolomics experiment includes replicate runs (or
  pooled QC replicates) and you need to score how well a normalization method (tGAM,
  rGAM, rLOESS, QC-RLSC, QC-RSC, SERFF, TIGER) reduced between-replicate technical
  dispersion.
when_to_use_negative:
- The dataset contains no replicate runs (the paper recommends including replicate
  samples in the design precisely to support this evaluation)
- The data have already been normalized — compute MAD reduction against the unnormalized
  baseline, not a previously normalized matrix
- You intend to benchmark using QC-sample RSD reduction or counts of differentially
  abundant metabolites as proxies — the authors identify both as common benchmarking
  pitfalls
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: metanorm
  role: R package providing metanorm() to generate per-method normalized matrices
    (model = "tGAM", "QC-RLSC", etc.) whose between-replicate MAD reductions are then
    compared
  repo: https://github.com/UGent-LIMET/Metanorm
- name: R
  role: Runtime environment (version >= 4.4.0) in which metanorm() is called and the
    MAD statistics are computed
  repo: https://cloud.r-project.org/index.html
- name: pak
  role: 'Used to install the Metanorm package from GitHub: pak::pak("UGent-LIMET/Metanorm")'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:0ff24837e261d8d82c190152be91bdff999eae79d08c76c629d1338aeebcdd76
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UGent-LIMET/Metanorm
  tools:
  - R
  - fANCOVA
  - ggplot2
  - metanorm
  - mgcv
  - pak
  - pbapply
  - rGAM
  - rLOESS
  - tGAM
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# between-replicate-dispersion-quantification

## Summary

Quantifies normalization performance by computing the MAD of pairwise normalized differences between replicate LC-MS runs and expressing the reduction relative to unnormalized data. This is the paper's preferred benchmarking metric, avoiding pitfalls such as RSD reduction of QC samples or inflated differential-abundance counts.

## When to use

Use when an LC-MS metabolomics experiment includes replicate runs (or pooled QC replicates) and you need to score how well a normalization method (tGAM, rGAM, rLOESS, QC-RLSC, QC-RSC, SERFF, TIGER) reduced between-replicate technical dispersion.

## When NOT to use

- The dataset contains no replicate runs (the paper recommends including replicate samples in the design precisely to support this evaluation)
- The data have already been normalized — compute MAD reduction against the unnormalized baseline, not a previously normalized matrix
- You intend to benchmark using QC-sample RSD reduction or counts of differentially abundant metabolites as proxies — the authors identify both as common benchmarking pitfalls

## Inputs

- Unnormalized LC-MS intensity matrix (rows = compounds, columns = sample runs; e.g., BioHEART plasma: 1359 runs x 53 metabolites)
- Vector annotating each run as QC or non-QC sample type (e.g., metanorm.qc)
- Batch annotation vector for each sample run (e.g., batch)
- Index/vector identifying which runs are replicates
- Normalized versions of the intensity matrix, one per normalization method

## Outputs

- Per-method between-replicate MAD reduction percentage vs. unnormalized data (e.g., 49-52% for robust methods, 41% SERFF, 40% TIGER, 18-22% QC-RL(R)SC)
- Ranked comparison of normalization methods by replicate concordance improvement

## How to apply

First normalize the raw intensity matrix (rows = compounds, columns = sample runs) with each candidate method via the metanorm() function, using the QC/sample-type vector, batch vector, and QCcheck = TRUE. For each normalized dataset, restrict to replicate runs, compute the MAD of pairwise between-replicate normalized differences, and express the reduction as a percentage relative to the same statistic on the unnormalized matrix. On the BioHEART plasma dataset (1359 runs x 53 metabolites) the reference targets are 49-52% reduction for the robust methods (rLOESS, rGAM, tGAM), 41% for SERFF, 40% for TIGER, and 18-22% for QC-RLSC and QC-RSC. Rationale: replicate concordance directly measures reduction of technical variance without relying on QC RSD reduction, a benchmarking pitfall the authors explicitly warn against. Replicate samples should be deliberately included in the experimental design to make this evaluation possible.

## Related tools

- **metanorm** (R package providing metanorm() to generate per-method normalized matrices (model = "tGAM", "QC-RLSC", etc.) whose between-replicate MAD reductions are then compared) — https://github.com/UGent-LIMET/Metanorm
- **R** (Runtime environment (version >= 4.4.0) in which metanorm() is called and the MAD statistics are computed) — https://cloud.r-project.org/index.html
- **pak** (Used to install the Metanorm package from GitHub: pak::pak("UGent-LIMET/Metanorm"))

## Examples

```
normdat <- metanorm(rawdata[1:5,], model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/Documents/metanormExample/")
```

## Evaluation signals

- Between-replicate MAD reductions fall in the reported ranges: 49-52% for rLOESS/rGAM/tGAM, 41% for SERFF, 40% for TIGER, 18-22% for QC-RLSC and QC-RSC (vs. unnormalized, BioHEART dataset)
- MAD is computed only over replicate run pairs, with the identical compound set retained between raw and normalized matrices
- Each normalized input is generated with the method's prescribed settings (e.g., QC-RLSC with QConly = TRUE; metanorm called with QCcheck = TRUE and batch vector)
- Computed reductions and accompanying PCA score plots match the values/figures reported in Figs. 3-4 of the paper
- QCcheck = TRUE confirms QCs are representative of biological samples before trusting the dispersion comparison

## Limitations

- Outliers can deteriorate the performance of existing normalization methods, so methods lacking outlier robustness (e.g., QC-RLSC/QC-RSC) may show reduced replicate-concordance gains
- tGAM has a higher computational cost than rGAM and rLOESS, although normalization remains feasible within minutes for thousands of metabolites and hundreds of samples
- Discrepancies between QC and biological samples can invalidate QC-based filtering/normalization; the package's QCcheck and inspection of per-compound pre-/post-normalization plots are required to catch subpar normalization

## Evidence

- [abstract] the robust methods, relative to several popular existing strategies, improved replicate concordance, and reduced drift and batch effects: "the robust methods, relative to several popular existing strategies, improved replicate concordance, and reduced drift and batch effects"
- [discussion] We recommend inclusion of both QC and replicate samples to support data quality and normalization performance investigation: "We recommend inclusion of both QC and replicate samples to support data quality and normalization performance investigation"
- [intro] Given common pitfalls in benchmarking normalization strategies, such as using a reduction of the relative standard deviation of QC samples: "Given common pitfalls in benchmarking normalization strategies, such as using a reduction of the relative standard deviation of QC samples or an increased number of differentially abundant"
- [readme] We further recommend using both QC as well as biological samples for normalization, and to have metanorm check for discrepancies: "We further recommend using both QC as well as biological samples for normalization, and to have metanorm check for discrepancies between QC and biological samples."
