---
name: figure-panel-reproduction-from-deposited-data
description: Use when a published figure must be reproduced from deposited deconvoluted
  GC-MS feature tables that carry sample metadata (e.g., body-location coordinates),
  and the panel depicts per-feature intensity distributions across spatial locations.
when_to_use_negative:
- The GC-MS data have not yet been auto-deconvoluted — run MSHub/GNPS deconvolution
  first, since this skill starts from deconvoluted feature tables.
- The deposited feature tables lack per-sample spatial metadata (e.g., body-location
  coordinates), so no spatial map can be reconstructed.
- The goal is novel annotation or molecular networking rather than reproduction of
  a specific published panel with its original thresholds.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: GNPS
  role: Source of the auto-deconvoluted GC-MS feature tables and the balance-score/cosine
    annotation metrics that are filtered and visualized
  repo: github.com/bittremieux/GNPS_GC
- name: '''ili'
  role: 3D human body model used to render each compound's per-body-location intensity
    distribution and rebuild the Figure 2f–i panels
- name: pandas
  role: Loading, merging, and threshold-filtering the headspace and liquid-injection
    feature tables
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-020-0700-3
    title: Auto-deconvolution and molecular networking of gas chromatography–mass
      spectrometry data
  source_content_hash: sha256:2c2e760aa51b68c3235425cb831c9e6d95d17488dea87b04f7521080ce93d809
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - '''ili'
  - GNPS
  - pandas
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# figure-panel-reproduction-from-deposited-data

## Summary

Rebuild published figure panels—in this case the molecular spatial maps of the human volatilome (Figure 2f–i)—from the deposited GNPS_GC deconvoluted feature tables by re-applying the authors' quality filters (balance ≥50%, cosine >0.9) and rendering per-body-location intensities in 'ili.

## When to use

Use when a published figure must be reproduced from deposited deconvoluted GC-MS feature tables that carry sample metadata (e.g., body-location coordinates), and the panel depicts per-feature intensity distributions across spatial locations.

## When NOT to use

- The GC-MS data have not yet been auto-deconvoluted — run MSHub/GNPS deconvolution first, since this skill starts from deconvoluted feature tables.
- The deposited feature tables lack per-sample spatial metadata (e.g., body-location coordinates), so no spatial map can be reconstructed.
- The goal is novel annotation or molecular networking rather than reproduction of a specific published panel with its original thresholds.

## Inputs

- Headspace GNPS deconvolution feature table with body-location coordinates (from github.com/bittremieux/GNPS_GC)
- Liquid-injection GNPS deconvolution feature table with body-location coordinates
- Balance score and spectral cosine similarity columns per deconvoluted feature
- 3D human body model loaded in 'ili

## Outputs

- Filtered feature table retaining balance ≥50% and cosine >0.9 features (filtered_feature_table.csv)
- Per-body-location intensity profiles for target compounds (squalene, hexanoic acid, globulol, phenylene dibenzoate)
- Reproduced spatial map panels equivalent to Figure 2f–i, exported as body_volatilome_maps.png

## How to apply

Load the headspace and liquid-injection GNPS deconvolution feature tables from the GNPS_GC release (github.com/bittremieux/GNPS_GC) into pandas DataFrames and merge them on deconvoluted feature identity. Reproduce the authors' annotation-quality thresholds by keeping only features with a balance score of at least 50%—the metric the paper introduces to quantify reproducibility of fragmentation patterns across all samples—and spectral cosine similarity above 0.9. Extract the per-body-location intensity profiles for the target compounds (e.g., squalene, hexanoic acid, globulol, phenylene dibenzoate). Render each compound's intensity distribution on the 3D human body model in 'ili to rebuild the Figure 2f–i panels, then export the images and the filtered feature table to verify exact reproduction.

## Related tools

- **GNPS** (Source of the auto-deconvoluted GC-MS feature tables and the balance-score/cosine annotation metrics that are filtered and visualized) — github.com/bittremieux/GNPS_GC
- **'ili** (3D human body model used to render each compound's per-body-location intensity distribution and rebuild the Figure 2f–i panels)
- **pandas** (Loading, merging, and threshold-filtering the headspace and liquid-injection feature tables)

## Examples

```
import pandas as pd; ft = pd.concat([pd.read_csv(f) for f in ['headspace_features.csv','li_features.csv']]); filt = ft[(ft.balance>=50)&(ft.cosine>0.9)]; filt.to_csv('filtered_feature_table.csv', index=False)
```

## Evaluation signals

- The filtered feature table contains only rows with balance score ≥50% and cosine similarity >0.9.
- Exported body maps show squalene present at all body locations but reduced on the feet, matching the reported finding.
- The merged table row count matches the union of headspace and liquid-injection features keyed on deconvoluted feature identity.
- The rendered panels visually correspond to Figure 2f–i of the original manuscript (doi:10.1038/s41587-020-0700-3).
- filtered_feature_table.csv and body_volatilome_maps.png are produced as final artifacts.

## Limitations

- Reproduction depends entirely on the deposited GNPS_GC release feature tables; the companion README provides no changelog or per-file schema documentation (the article's discussion notes 'No changelog found').
- Filter thresholds (balance 50%, cosine 0.9) are those chosen by the authors for this panel; other reproduction targets may use different cutoffs.
- The balance score only quantifies reproducibility of fragmentation patterns across samples in the deposited dataset; features absent from the release cannot be recovered.

## Evidence
