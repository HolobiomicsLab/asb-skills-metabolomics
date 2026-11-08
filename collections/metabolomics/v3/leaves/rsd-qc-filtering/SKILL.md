---
name: rsd-qc-filtering
description: Use when you have an XCMS peak table plus pooled QC injections and want
  a threshold-based reproducibility filter (RSD < 30%) on features before statistical
  analysis, or as a baseline to compare against classifier-driven Pass/Fail flagging.
when_to_use_negative:
- When peaks are already labeled Pass/Fail by a trained peak-quality classifier such
  as MetaClean — RSD filtering cannot assess integration quality, only reproducibility.
- When the dataset contains no pooled QC injections (RSD must be computed on a repeatedly
  injected QC sample).
- When vendor-processed outputs are the only available input — the MetaClean paper
  notes this scope limitation, so convert to an XCMS object or feature table first.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: XCMS
  role: Produces the feature table (peak-picking, retention-time correction, grouping)
    that the RSD filter is applied to
  repo: https://CRAN.R-project.org/package=MetaClean
- name: MetaClean
  role: Benchmark comparator — the paper compared RSD < 30% filtering against the
    MetaClean global classifier as an alternative automated filtering layer
  repo: https://CRAN.R-project.org/package=MetaClean
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:456ecb7dec2f4e38e6aea4d6125ecdfdec290f02bdd488dbf552e6bbf94315b5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IPO
  - MetaClean
  - XCMS
  - scmamp
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# rsd-qc-filtering

## Summary

Filter untargeted LC–MS metabolomics features by relative standard deviation (RSD) computed on peaks in a routinely injected pooled QC sample, retaining peaks with RSD < 30% as a lightweight, label-free alternative to classifier-based peak quality flagging.

## When to use

Use when you have an XCMS peak table plus pooled QC injections and want a threshold-based reproducibility filter (RSD < 30%) on features before statistical analysis, or as a baseline to compare against classifier-driven Pass/Fail flagging.

## When NOT to use

- When peaks are already labeled Pass/Fail by a trained peak-quality classifier such as MetaClean — RSD filtering cannot assess integration quality, only reproducibility.
- When the dataset contains no pooled QC injections (RSD must be computed on a repeatedly injected QC sample).
- When vendor-processed outputs are the only available input — the MetaClean paper notes this scope limitation, so convert to an XCMS object or feature table first.

## Inputs

- XCMS feature table (peak intensities per sample) including pooled QC sample injections
- Sample-class labels distinguishing pooled QC from biological/QA samples

## Outputs

- Subset feature table containing only peaks with RSD < 30% in pooled QCs
- Per-feature RSD values for the pooled QC group (optional, useful for downstream auditing)

## How to apply

Compute RSD for each feature's peak intensities across pooled QC injections using an RSD < 30% threshold as the cutoff for reproducible peaks, following the common metabolomics practice referenced by Dunn et al. 2011 and Want et al. 2010. Apply the cutoff to create a revised feature table containing only peaks that pass the QC-reproducibility bar. In the MetaClean benchmark this filter was used both to build the labeled development/test datasets and as the alternative approach compared head-to-head against the MetaClean global classifier, so it can serve either as a standalone filter or as a baseline evaluation condition. The filter does not assess peak integration quality directly — it only removes features whose pooled-QC variance is too high — so peaks that pass may still be low-quality integrations that a trained classifier (e.g. AdaBoost on the M11 metric set) would flag.

## Related tools

- **XCMS** (Produces the feature table (peak-picking, retention-time correction, grouping) that the RSD filter is applied to) — https://CRAN.R-project.org/package=MetaClean
- **MetaClean** (Benchmark comparator — the paper compared RSD < 30% filtering against the MetaClean global classifier as an alternative automated filtering layer) — https://CRAN.R-project.org/package=MetaClean

## Examples

```
R> retained <- peaks[rowSds(peaks[, qc_cols]) / rowMeans(peaks[, qc_cols]) < 0.30, ]
```

## Evaluation signals

- Recompute RSD on the retained feature set: every surviving peak should have RSD < 30% in pooled QCs.
- Compare retained peak counts before and after filtering; a large unexplained increase (e.g. peaks surviving with RSD ≥ 30%) indicates a mis-specified QC label or threshold.
- Cross-check against the article's condition: the filter is intended to remove noisy, poorly reproducible peaks — surviving features should show tighter QC-group intensity variance than removed ones.
- If used as a baseline against MetaClean, verify that classifier-flagged Fail peaks include peaks that pass the RSD filter (the paper notes MetaClean caught poor integrations RSD missed).

## Limitations

- RSD filtering is threshold-based and does not capture peak integration quality; the paper shows MetaClean's classifier performed better at distinguishing high-quality from low-quality peaks than RSD < 30% alone.
- The threshold of RSD < 30% is a commonly cited but not universally optimal cutoff; it may need adjustment for different platforms, ionization modes, or biological matrices.
- Requires pooled QC injections — cannot be applied when only biological samples are available.

## Evidence
