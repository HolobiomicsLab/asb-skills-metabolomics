---
name: cross-validation-span-selection-comparison
description: Use when normalizing LC-MS metabolomics run-order data with Metanorm
  and you must decide between GCV and LOOCV smoother selection, or need to demonstrate
  that downstream differential abundance results are insensitive to that choice.
when_to_use_negative:
- When the goal is to compare different normalization methods against each other (e.g.
  robust methods vs QC-RLSC DAM overlap) rather than to compare the two span-selection
  schemes within one method
- When the input data are already normalized or when no QC samples exist to anchor
  QC-based drift fitting, since metanorm() requires an unnormalized matrix and representative
  QC runs for QCcheck = TRUE
- When a single normalization run suffices and no sensitivity assessment of span/smoother
  selection is needed — running both cv schemes doubles computation, which is a real
  cost for the slower tGAM model
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: metanorm
  role: R function performing QC-based drift normalization under either GCV or LOOCV
    span/smoother selection via the cv argument; the same call is repeated with the
    other cv value to build the scheme comparison
  repo: github.com/UGent-LIMET/Metanorm
- name: Metanorm R package
  role: Package providing metanorm(), plotPCA(), and per-compound pre- vs post-normalization
    diagnostics used to visualize both schemes' behavior
  repo: github.com/UGent-LIMET/Metanorm
- name: R
  role: Runtime environment; the package requires R version >= 4.4.0
- name: ggplot2
  role: Rendering PCA score plots and scheme-comparison figures saved alongside the
    plotdir diagnostics
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:572f5836829e8887d6fcbb013fab047c4f964c94b7a8b44c2142a05f9c23f7bb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Metanorm R package
  - R
  - ggplot2
  - metanorm
  - rGAM
  - rLOESS
  - tGAM
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# cross-validation-span-selection-comparison

## Summary

Compares the two span/smoother selection schemes supported by the Metanorm metanorm() function — cross-validated generalized cross-validation (GCV) and leave-one-out cross-validation (LOOCV), selected via the cv argument — by running each normalization method under both schemes and verifying that differential abundance testing outcomes and p-value orderings remain comparable.

## When to use

Use when normalizing LC-MS metabolomics run-order data with Metanorm and you must decide between GCV and LOOCV smoother selection, or need to demonstrate that downstream differential abundance results are insensitive to that choice.

## When NOT to use

- When the goal is to compare different normalization methods against each other (e.g. robust methods vs QC-RLSC DAM overlap) rather than to compare the two span-selection schemes within one method
- When the input data are already normalized or when no QC samples exist to anchor QC-based drift fitting, since metanorm() requires an unnormalized matrix and representative QC runs for QCcheck = TRUE
- When a single normalization run suffices and no sensitivity assessment of span/smoother selection is needed — running both cv schemes doubles computation, which is a real cost for the slower tGAM model

## Inputs

- Raw (unnormalized) LC-MS metabolomics intensity matrix in Metanorm's example.RData structure: numerical matrix with rows = compounds and columns = sample runs (e.g. 618 runs x 8913 features for the FAME saliva cohort)
- Sample-type vector (metanorm.qc) marking each run as 'QC' or a non-QC biological sample
- Batch vector assigning each sample run to its acquisition batch
- Candidate normalization model choice (tGAM, rGAM, rLOESS, or QC-RLSC)

## Outputs

- Two normalized data matrices per method — one under GCV span/smoother selection and one under LOOCV — covering the full feature set
- Per-method comparison of DAM counts and percentage DAM overlap between the two cross-validation schemes
- Per-method comparison of p-value orderings between GCV and LOOCV
- Diagnostic pre- vs post-normalization intensity-vs-order plots and PCA score plots (plotdir + plotPCA) for both schemes, in the style of Supplementary Fig. S10–S11

## How to apply

Run metanorm() from the Metanorm R package (requires R >= 4.4.0, installed via pak::pak("UGent-LIMET/Metanorm")) on the same raw intensity matrix under both values of the cv argument (GCV and LOOCV), for each candidate model — tGAM (default), rGAM, rLOESS, and QC-RLSC with QConly = TRUE and batch-wise normalization, with QCcheck = TRUE and diagnostic plots written to a plotdir. On the untargeted FAME saliva cohort (618 runs, 8913 metabolic features) this means eight normalized matrices: four methods x two cross-validation schemes. Run differential abundance testing on each normalized matrix and compute the percentage of overlapping differentially abundant metabolites (DAMs) between schemes within each method, and compare DAM counts and p-value orderings between GCV and LOOCV. The published benchmark found robust-method DAM overlaps of 79–87% among the three robust methods (rLOESS, rGAM, tGAM) and 55–59% overlap between QC-RLSC and the robust methods, with GCV vs LOOCV yielding highly comparable DAM counts and p-value orderings across all methods; render the scheme comparison as PCA score-plot and intensity-vs-order figure equivalents (Supplementary Fig. S10–S11 style) using plotPCA/ggplot2 saved alongside the plotdir diagnostics. If DAM counts or p-value orderings diverge materially between the two schemes within a method, investigate span estimation instability before trusting either normalized matrix.

## Related tools

- **metanorm** (R function performing QC-based drift normalization under either GCV or LOOCV span/smoother selection via the cv argument; the same call is repeated with the other cv value to build the scheme comparison) — github.com/UGent-LIMET/Metanorm
- **Metanorm R package** (Package providing metanorm(), plotPCA(), and per-compound pre- vs post-normalization diagnostics used to visualize both schemes' behavior) — github.com/UGent-LIMET/Metanorm
- **R** (Runtime environment; the package requires R version >= 4.4.0)
- **ggplot2** (Rendering PCA score plots and scheme-comparison figures saved alongside the plotdir diagnostics)

## Examples

```
normdat_gcv <- metanorm(rawdata, model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, cv = "GCV", plotdir = "~/plots/gcv/"); normdat_loocv <- metanorm(rawdata, model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, cv = "LOOCV", plotdir = "~/plots/loocv/")
```

## Evaluation signals

- Running metanorm() with each cv value (GCV and LOOCV) on the identical raw matrix and model yields two normalized matrices of identical dimensions (features x runs) to the input
- Within each method, the percentage of overlapping DAMs between GCV- and LOOCV-normalized data falls in a high-agreement band (robust methods: 79–87% among themselves; QC-RLSC: 55–59% with robust methods), and GCV vs LOOCV within a method shows highly comparable DAM counts and p-value orderings
- QCcheck = TRUE passes without flagging discrepancies between QC and biological samples before comparing schemes
- PCA score plots of GCV- vs LOOCV-normalized data show equivalent batch-effect reduction relative to the unnormalized input
- Diagnostic pre- vs post-normalization intensity-vs-order plots for both schemes show consistent drift removal across biological (non-QC) samples, not just QC samples

## Limitations

- tGAM has a higher computational cost than rLOESS and rGAM, so running both cross-validation schemes doubles an already longer run — normalization still completes within minutes for thousands of metabolites and hundreds of samples, but practitioners should budget accordingly
- Agreement between schemes (or between methods) is quantified by DAM overlap percentages, which the authors warn are common pitfalls in benchmarking normalization strategies if used in isolation from visual diagnostics
- The 79–87% and 55–59% overlap figures are specific to the FAME saliva cohort (618 runs, 8913 features); other datasets may show different scheme sensitivity
- Metanorm's GAM-based procedures can adaptively revert to sample-based normalization, which may mask drift-fitting differences between GCV and LOOCV on some features

## Evidence

- [methods] Cross-validation scheme comparison via cv argument and DAM overlap benchmark: [evidence span withheld — anchor status: not_found]
- [results] Robust vs QC-RLSC DAM overlap percentages on FAME cohort: [evidence span withheld — anchor status: not_found]
- [results] GCV vs LOOCV produces comparable downstream results: [evidence span withheld — anchor status: not_found]
- [readme] Package installation requirement grounding the runtime: "Make sure `R` (**version >= 4.4.0**) is installed on your computer"
- [readme] Recommended QC check before comparing schemes: "We further recommend using both QC as well as biological samples for normalization, and to have metanorm check for discrepancies between QC and biological samples"
- [intro] DAM overlap alone is an insufficient benchmark signal: "common pitfalls in benchmarking normalization strategies, such as using a reduction of the relative standard deviation of QC samples or an increased number of differentially abundant"
