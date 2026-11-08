---
name: spectral-cosine-similarity-thresholding
description: Use when a deconvoluted GNPS GC-MS feature table must be cleaned before
  interpretation — e.g. to select reliably annotated, reproducible features prior
  to spatial mapping, differential analysis, or figure reproduction.
when_to_use_negative:
- Input is a raw GC-MS chromatogram that has not yet been auto-deconvolved by MSHub/GNPS
  — cosine and balance scores will not exist yet.
- The goal is broad metabolite discovery where low-confidence annotations should be
  kept; the cosine >0.9 cutoff intentionally sacrifices recall for annotation confidence.
- Feature tables originate from LC-MS/MS molecular networking rather than GC-MS deconvolution,
  where the balance-score metric is not defined.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GNPS
  role: Provides the deconvoluted feature tables with balance scores and spectral
    cosine similarity annotations that the threshold filter is applied to
  repo: http://gnps.ucsd.edu
- name: MSHub
  role: Auto-deconvolution engine whose non-negative matrix factorization outputs
    supply the fragmentation patterns scored by balance score and cosine similarity
- name: '''ili'
  role: Downstream consumer of the filtered feature table for rendering compound intensity
    distributions on the 3D human body model
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-020-0700-3
    title: Auto-deconvolution and molecular networking of gas chromatography–mass
      spectrometry data
  source_content_hash: sha256:69efe191e9284bf74a2c4842d9fcd97994c4d2e6741768d03d01fe4bf40a1118
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - '''ili'
  - GNPS
  - MSHub
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# spectral-cosine-similarity-thresholding

## Summary

Filters deconvoluted GC-MS features from GNPS/MSHub outputs by spectral cosine similarity (>0.9) combined with a balance score cutoff (≥50%), retaining only features whose fragmentation patterns are well-annotated and reproducible across samples before downstream visualization or body-mapping.

## When to use

Use when a deconvoluted GNPS GC-MS feature table must be cleaned before interpretation — e.g. to select reliably annotated, reproducible features prior to spatial mapping, differential analysis, or figure reproduction.

## When NOT to use

- Input is a raw GC-MS chromatogram that has not yet been auto-deconvolved by MSHub/GNPS — cosine and balance scores will not exist yet.
- The goal is broad metabolite discovery where low-confidence annotations should be kept; the cosine >0.9 cutoff intentionally sacrifices recall for annotation confidence.
- Feature tables originate from LC-MS/MS molecular networking rather than GC-MS deconvolution, where the balance-score metric is not defined.

## Inputs

- GNPS_GC headspace deconvolution feature table (with body-location coordinates)
- GNPS_GC liquid-injection deconvolution feature table (with body-location coordinates)
- Per-feature balance scores and spectral cosine similarity values from MSHub/GNPS auto-deconvolution

## Outputs

- filtered_feature_table.csv — merged feature table restricted to balance ≥50% and cosine >0.9 features
- Per-compound intensity profiles (e.g. squalene, hexanoic acid, globulol, phenylene dibenzoate) suitable for 3D visualization

## How to apply

Load the headspace and liquid-injection GNPS deconvolution feature tables from the GNPS_GC release (github.com/bittremieux/GNPS_GC) into pandas DataFrames and merge them into a single combined table keyed on deconvoluted feature identity. Apply two thresholds to the merged table: retain only features with a spectral cosine similarity above 0.9 (annotation reliability vs. the reference spectrum) and a balance score of at least 50% (reproducibility of the fragmentation pattern across all samples, as introduced by the MSHub/GNPS auto-deconvolution method). The high cosine cutoff (0.9 rather than a looser 0.7) prioritizes confident spectral identity over recall, which is appropriate when features feed downstream visualization such as 3D body-map rendering in 'ili. Export the surviving rows as a filtered feature table (filtered_feature_table.csv) and use per-body-location intensity columns for the compounds of interest (e.g. squalene, hexanoic acid, globulol, phenylene dibenzoate).

## Related tools

- **GNPS** (Provides the deconvoluted feature tables with balance scores and spectral cosine similarity annotations that the threshold filter is applied to) — http://gnps.ucsd.edu
- **MSHub** (Auto-deconvolution engine whose non-negative matrix factorization outputs supply the fragmentation patterns scored by balance score and cosine similarity)
- **'ili** (Downstream consumer of the filtered feature table for rendering compound intensity distributions on the 3D human body model)

## Examples

```
filtered = combined[(combined['cosine'] > 0.9) & (combined['balance_score'] >= 50)]; filtered.to_csv('filtered_feature_table.csv')
```

## Evaluation signals

- Every row in the filtered table has cosine similarity >0.9 and balance score ≥50%; verify no feature violates either cutoff.
- Row count of the filtered table is strictly less than (or equal to) the merged headspace + liquid-injection input table.
- Known compounds such as squalene and hexanoic acid survive the filter and show plausible spatial patterns (e.g. squalene present on all body locations but reduced on the feet), matching Figure 2f–i.
- The merged table keys align on deconvoluted feature identity across the headspace and liquid-injection tables before filtering.

## Limitations

- The balance score quantifies reproducibility of fragmentation patterns across samples, so features absent from many samples may be filtered out even if spectrally confident.
- Cosine >0.9 is a stringent cutoff; near-threshold features with slightly degraded spectra (e.g. co-eluting interferences) may be lost despite correct identity.
- The companion GNPS_GC repository README provides no usage/changelog documentation — it only links the Nature Biotechnology manuscript.

## Evidence

- [methods] Filter features to those with a balance score of at least 50% and spectral cosine similarity above 0.9.: [evidence span withheld — anchor status: not_found]
- [readme] GNPS_GC companion repo: [evidence span withheld — anchor status: not_found]
- [results] Cosine filter output example: [evidence span withheld — anchor status: not_found]
