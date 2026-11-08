---
name: fragmentation-balance-score-filtering
description: Use when MSHub/GNPS deconvolution feature tables (e.g. from GNPS_GC headspace
  or liquid-injection runs) contain noisy or sparsely reproduced fragments and a high-confidence
  subset is needed before mapping intensities or rendering annotations.
when_to_use_negative:
- The input is already a manually validated or annotated feature table for which reproducibility-based
  curation would discard legitimate low-balance features.
- The data are raw GC-MS spectra that have not yet been auto-deconvoluted by MSHub/GNPS
  — the balance score only exists on deconvolution output.
- The analysis goal is LC-MS/MS molecular networking (e.g. pyMolNetEnhancer MS2LDA
  motif mapping), where fragmentation-pattern balance scores from GC-MS deconvolution
  do not apply.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Hosts the MSHub GC-MS auto-deconvolution workflow whose deconvoluted feature
    tables carry the balance scores and cosine similarities being filtered
  repo: http://gnps.ucsd.edu
- name: GNPS_GC release (bittremieux/GNPS_GC)
  role: Provides the headspace and liquid-injection deconvolution feature tables that
    are merged and filtered in pandas
  repo: github.com/bittremieux/GNPS_GC
- name: MSHub
  role: Machine-learning auto-deconvolution engine that generates the fragmentation
    patterns, balance scores, and deconvoluted features the filter operates on
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-020-0700-3
    title: Auto-deconvolution and molecular networking of gas chromatography–mass
      spectrometry data
  source_content_hash: sha256:78ba26f4c6ee3f3135317eea172afeb1997b13dadb1a4a0e527801b9377b295c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - '''ili'
  - GNPS
  - GNPS_GC release (bittremieux/GNPS_GC)
  - MSHub
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# fragmentation-balance-score-filtering

## Summary

Curates deconvoluted GC-MS feature tables produced by MSHub/GNPS auto-deconvolution by retaining only features whose fragmentation patterns are reproducible across samples (balance score ≥ 50%) and whose spectral matches are confident (cosine similarity > 0.9), yielding a high-confidence feature table ready for downstream visualization such as the 3D human volatilome body map.

## When to use

Use when MSHub/GNPS deconvolution feature tables (e.g. from GNPS_GC headspace or liquid-injection runs) contain noisy or sparsely reproduced fragments and a high-confidence subset is needed before mapping intensities or rendering annotations.

## When NOT to use

- The input is already a manually validated or annotated feature table for which reproducibility-based curation would discard legitimate low-balance features.
- The data are raw GC-MS spectra that have not yet been auto-deconvoluted by MSHub/GNPS — the balance score only exists on deconvolution output.
- The analysis goal is LC-MS/MS molecular networking (e.g. pyMolNetEnhancer MS2LDA motif mapping), where fragmentation-pattern balance scores from GC-MS deconvolution do not apply.

## Inputs

- MSHub/GNPS GC-MS deconvolution feature table from headspace analysis (GNPS_GC release, pandas-loadable CSV)
- MSHub/GNPS GC-MS deconvolution feature table from liquid-injection analysis (GNPS_GC release, pandas-loadable CSV)
- Per-feature balance scores and spectral cosine similarity values
- Per-sample / per-body-location intensity profiles keyed to feature identity

## Outputs

- Filtered feature table retaining features with balance score ≥ 50% and cosine similarity > 0.9 (e.g. filtered_feature_table.csv)
- Per-compound intensity profiles (e.g. squalene, hexanoic acid, globulol, phenylene dibenzoate) for downstream spatial visualization

## How to apply

Load the headspace and liquid-injection GNPS_GC deconvolution feature tables into pandas DataFrames and merge them on deconvoluted feature identity. The MSHub/GNPS auto-deconvolution assigns each feature a 'balance score' quantifying the reproducibility of its fragmentation pattern across all samples; retain features with balance score at least 50%. Additionally require spectral cosine similarity above 0.9 so only confidently matched fragment spectra survive. Optionally extract per-body-location (or per-sample) intensity profiles for target compounds such as squalene, hexanoic acid, globulol, and phenylene dibenzoate from the filtered table. Export the filtered table (e.g. filtered_feature_table.csv) and pass intensity profiles downstream to visualization tools such as 'ili.

## Related tools

- **GNPS** (Hosts the MSHub GC-MS auto-deconvolution workflow whose deconvoluted feature tables carry the balance scores and cosine similarities being filtered) — http://gnps.ucsd.edu
- **GNPS_GC release (bittremieux/GNPS_GC)** (Provides the headspace and liquid-injection deconvolution feature tables that are merged and filtered in pandas) — github.com/bittremieux/GNPS_GC
- **MSHub** (Machine-learning auto-deconvolution engine that generates the fragmentation patterns, balance scores, and deconvoluted features the filter operates on)

## Examples

```
import pandas as pd; df = pd.concat([pd.read_csv(t) for t in ['headspace_features.csv','liquid_injection_features.csv']]); df[(df.balance >= 50) & (df.cosine > 0.9)].to_csv('filtered_feature_table.csv', index=False)
```

## Evaluation signals

- Every retained feature row has balance score ≥ 50% and cosine similarity > 0.9; a spot check of the minima of both columns confirms the thresholds were applied jointly.
- The merged table row count equals the union of headspace and liquid-injection feature identities (no lost or duplicated deconvoluted features during the merge).
- Filtered-table intensities for a known abundant skin volatile such as squalene appear across body locations except reduced on the feet, matching Figure 2f-i expectations.
- Output file filtered_feature_table.csv parses cleanly in pandas with intact intensity columns for each body location.
- Downstream 'ili rendering of the filtered profiles reproduces spatial distributions consistent with the published body maps.

## Limitations

- The 50% balance and cosine > 0.9 thresholds were tuned for the human volatilome GC-MS dataset; other datasets may need recalibration.
- Filtering discards features with low fragmentation-pattern reproducibility, which may exclude real but sample-inconsistent compounds.
- The bittremieux/GNPS_GC repository is archived, and its README gives no changelog or usage instructions beyond the manuscript citation.

## Evidence

- [readme] data provenance: [evidence span withheld — anchor status: not_found]
