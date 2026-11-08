---
name: differential-abundance-overlap-analysis
description: Use when comparing two or more normalization strategies for metabolomics
  LC-MS data and you need a downstream functional measure of agreement — overlap of
  differentially abundant metabolites — rather than QC-RSD or replicate-MAD metrics
  alone.
when_to_use_negative:
- When you only need QC-based variance metrics — the authors warn against benchmarking
  normalization by DAM counts or QC relative standard deviation (RSD) alone, so use
  this skill only as a comparative agreement measure, not as a standalone quality
  verdict
- When normalization has already been applied with a single method and there is no
  alternative normalized matrix to compare against (overlap requires at least two
  DAM sets)
- When the goal is differential analysis of one condition pair without cross-method
  benchmarking
edam_topics:
- http://edamontology.org/topic_0153
tools:
- name: Metanorm R package
  role: Provides metanorm() to produce the per-method normalized matrices (tGAM/rGAM/rLOESS,
    QC-RLSC with QConly) on which differential abundance testing and DAM overlap computation
    are performed
  repo: github.com/UGent-LIMET/Metanorm
- name: metanorm
  role: Normalization function called per method; tGAM is the recommended default,
    QC-RLSC requires QConly = TRUE
  repo: github.com/UGent-LIMET/Metanorm
- name: R
  role: Runtime environment (version >= 4.4.0) hosting the Metanorm package, differential
    abundance testing, and overlap computation
- name: ggplot2
  role: Used to render comparison figures (PC score plots, supplementary-figure equivalents
    for GCV vs LOOCV DAM comparisons)
- name: pak
  role: Installs the Metanorm package from GitHub (pak::pak("UGent-LIMET/Metanorm"))
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:3cf1429bdcd41c15eb7cc63bf46bea9dd0a22e5729d0a422501d91d5ca2b3785
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Metanorm R package
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# differential-abundance-overlap-analysis

## Summary

Quantifies agreement between normalization methods by running differential abundance (DAM) testing on each normalized matrix and computing pairwise percentage overlaps of DAMs. In this study it showed robust methods (rLOESS, rGAM, tGAM) agree far more with each other (79-87% on FAME, 92% on BioHEART) than QC-RLSC agrees with them (55-59%).

## When to use

Use when comparing two or more normalization strategies for metabolomics LC-MS data and you need a downstream functional measure of agreement — overlap of differentially abundant metabolites — rather than QC-RSD or replicate-MAD metrics alone.

## When NOT to use

- When you only need QC-based variance metrics — the authors warn against benchmarking normalization by DAM counts or QC relative standard deviation (RSD) alone, so use this skill only as a comparative agreement measure, not as a standalone quality verdict
- When normalization has already been applied with a single method and there is no alternative normalized matrix to compare against (overlap requires at least two DAM sets)
- When the goal is differential analysis of one condition pair without cross-method benchmarking

## Inputs

- Unnormalized LC-MS metabolomics intensity matrix (e.g., BioHEART plasma: 1359 runs x 53 metabolites; FAME saliva: 618 runs x 8913 features), rows = compounds, columns = sample runs
- Sample-type vector distinguishing QC vs non-QC runs (metanorm.qc / type)
- Batch annotation vector per run
- Normalization method specifications (tGAM, rGAM, rLOESS, QC-RLSC with QConly=TRUE, SERFF, TIGER) and cv choice (GCV vs LOOCV)

## Outputs

- Per-method normalized data matrices
- Per-method DAM lists (differentially abundant metabolites with test statistics)
- Pairwise DAM overlap percentages across normalization methods (e.g., 79-87% among robust methods, 55-59% with QC-RLSC on FAME; 92% among robust, 29-41% for SERFF on BioHEART)
- Per-method DAM counts and p-value orderings compared across GCV vs LOOCV

## How to apply

First normalize the full intensity matrix under each candidate method via metanorm() from the Metanorm R package (model = 'tGAM' default, 'rLOESS', 'rGAM' for robust methods; model = 'QC-RLSC' with QConly = TRUE for QC-driven drift fitting; batch-wise with QCcheck = TRUE), running each method under both GCV and LOOCV span selection via the cv argument where applicable. Then run differential abundance testing on each of the normalized matrices (with multiple testing correction, which the authors note is essential for downstream inference), compute pairwise percentages of overlapping DAMs, and additionally compare DAM counts and p-value orderings across cross-validation settings. Interpret large overlap (>~79-87%) among robust methods as evidence of stable biological signal, and substantially lower overlap (29-41% for SERFF vs robust methods, 55-59% for QC-RLSC vs robust methods on FAME) as evidence of method-dependent DAM sets. Avoid using an increased DAM count alone as a proxy for normalization quality — the authors explicitly flag this as a benchmarking pitfall.

## Related tools

- **Metanorm R package** (Provides metanorm() to produce the per-method normalized matrices (tGAM/rGAM/rLOESS, QC-RLSC with QConly) on which differential abundance testing and DAM overlap computation are performed) — github.com/UGent-LIMET/Metanorm
- **metanorm** (Normalization function called per method; tGAM is the recommended default, QC-RLSC requires QConly = TRUE) — github.com/UGent-LIMET/Metanorm
- **R** (Runtime environment (version >= 4.4.0) hosting the Metanorm package, differential abundance testing, and overlap computation)
- **ggplot2** (Used to render comparison figures (PC score plots, supplementary-figure equivalents for GCV vs LOOCV DAM comparisons))
- **pak** (Installs the Metanorm package from GitHub (pak::pak("UGent-LIMET/Metanorm")))

## Examples

```
load(system.file("extdata","example.RData",package="metanorm")); normdat <- metanorm(rawdata[1:5,], model="tGAM", type=metanorm.qc, QCcheck=TRUE, batch=batch, plotdir="~/metanormExample/")
```

## Evaluation signals

- On the FAME saliva cohort, robust-method (rLOESS/rGAM/tGAM) pairwise DAM overlaps must fall in the reported 79-87% band, and QC-RLSC overlaps with robust methods in the 55-59% band
- On the BioHEART dataset, joint-DAM overlap among robust methods should reach ~92%, with SERFF overlaps at 29-41%
- GCV vs LOOCV within each method must yield highly comparable DAM counts and p-value orderings
- Downstream differential testing must include multiple testing correction before defining DAM sets (essential for valid inference)
- DAM sets should not be judged by count alone: an increased number of DAMs is explicitly called out as a benchmarking pitfall, so overlap percentages are the target metric

## Limitations

- Multiple testing correction is essential for downstream statistical inference but was not treated in depth in this work
- Overlapping DAMs depend on upstream normalization choices; QC-RLSC yields markedly weaker agreement (55-59%) because it fits drift on QC samples only (QConly = TRUE), unlike the robust methods that use both QC and biological samples
- Cross-metabolite information is not leveraged in feature-level normalization or testing, which could otherwise further enhance downstream findings
- QC-based filtering (e.g., on QC-sample RSD) may be invalid when QC and biological sample distributions differ, potentially biasing the DAM sets that feed the overlap computation

## Evidence

- [intro] common pitfalls in benchmarking normalization strategies: "Given common pitfalls in benchmarking normalization strategies, such as using a reduction of the relative standard deviation of QC samples or an increased number of differentially abundant"
- [discussion] multiple testing correction essential: "While not in the scope of this work, multiple testing correction is essential for downstream statistical inference."
- [abstract] robust methods improved replicate concordance: "the robust methods, relative to several popular existing strategies, improved replicate concordance, and reduced drift and batch effects"
- [readme] QC and biological samples recommendation: "We further recommend using both QC as well as biological samples for normalization, and to have metanorm check for discrepancies between QC and biological samples."
- [methods] QCcheck argument: "QCcheck = TRUE,      # check whether QCs are representative"
- [discussion] RSD filtering caveat: "differences between QC and biological samples may invalidate filtering on e.g., relative standard deviation (RSD) observed in QC samples"
