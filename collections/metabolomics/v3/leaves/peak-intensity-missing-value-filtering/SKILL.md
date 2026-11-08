---
name: peak-intensity-missing-value-filtering
description: Use when preparing a raw LC-MS feature/peak intensity table for downstream
  annotation or prioritization and many features have zero or missing intensities
  across samples.
when_to_use_negative:
- Input is already a cleaned feature table where zero-intensity features have been
  removed and missingness has been handled upstream.
- The analysis target is raw signal preprocessing (peak picking, alignment) rather
  than selecting annotated features prior to compound annotation.
- The tool or database in use does not assume a KEGG-style compound/feature structure
  and its own filtering rules would conflict with the 80% rule.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0218
tools:
- name: mWISE
  role: R package whose pipeline consumes the filtered peak list before matchingStage,
    featuresClustering and diffusion-based prioritization
  repo: https://github.com/b2slab/mWISE
- name: xMSannotator
  role: compared annotation tool applied to the same filtered peak lists across the
    three studies in both ionization modes
- name: R
  role: language in which the filtering of peak lists and the whole mWISE workflow
    is implemented
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c00238
    title: 'mWISE: An Algorithm for Context-Based Annotation of Liquid Chromatography–Mass
      Spectrometry Features through Diffusion in Graphs'
  source_content_hash: sha256:6d8f36acf541b2e54e67e121cfde66d058591143a12132cdb56691fe35a5493b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/b2slab/mWISE
  repo_url: https://github.com/b2slab/mWISE
  tools:
  - Diffu Stats
  - FELLA
  - Genomic Ranges
  - R
  - mWISE
  - xMSannotator
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# peak-intensity-missing-value-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filter an LC-MS peak list before annotation by removing features without signal and applying the 80% rule: drop peaks whose intensity equals 0 in more than 20% of samples. mWISE uses this on the public assay peak lists so that matching, clustering and diffusion run on a cleaner feature matrix.

## When to use

Use when preparing a raw LC-MS feature/peak intensity table for downstream annotation or prioritization and many features have zero or missing intensities across samples.

## When NOT to use

- Input is already a cleaned feature table where zero-intensity features have been removed and missingness has been handled upstream.
- The analysis target is raw signal preprocessing (peak picking, alignment) rather than selecting annotated features prior to compound annotation.
- The tool or database in use does not assume a KEGG-style compound/feature structure and its own filtering rules would conflict with the 80% rule.

## Inputs

- input peak list of LC-MS features (m/z, retention time, per-sample intensities) per study and ionization mode
- sample metadata defining which columns are samples

## Outputs

- filtered peak list with features without signal removed and the 80% rule applied
- feature matrix ready for mWISE matchingStage, featuresClustering, clusterBased.filter and diffusion.input/set.diffusion

## How to apply

Start from an input peak list of LC-MS features (m/z, retention time, per-sample intensities) for each study and ionization mode. First remove features without signal, i.e. rows with no detected intensity. Then apply the 80% rule: if the intensity of a feature is equal to 0 in more than 20% of samples (present in <80%), discard that feature. In the mWISE workflow this filtering precedes matchingStage (which matches peaks to KEGG over a set of adducts and fragments), featuresClustering (spectral clustering with DBSCAN and optimized principal components and epsilon) and the clusterBased.filter step, because a high false-positive rate inflates the labels fed to set.diffusion via DiffuStats. The filtered peak lists for the three public studies, in both positive and negative modes, are then scored against known compound references to estimate sensitivity, so consistent filtering is required to reproduce the reported sensitivity mean (SD) of 0.63 (0.07) versus xMSannotator's 0.55 (0.19).

## Related tools

- **mWISE** (R package whose pipeline consumes the filtered peak list before matchingStage, featuresClustering and diffusion-based prioritization) — https://github.com/b2slab/mWISE
- **xMSannotator** (compared annotation tool applied to the same filtered peak lists across the three studies in both ionization modes)
- **R** (language in which the filtering of peak lists and the whole mWISE workflow is implemented)

## Examples

```
R: filtered <- peakList[apply(peakList, 1, function(x) !all(x == 0)), ]; filtered <- filtered[apply(filtered, 1, function(x) sum(x == 0) / length(x) <= 0.2), ]
```

## Evaluation signals

- Every retained feature should be present (non-zero intensity) in at least 80% of samples.
- No feature with zero signal across all samples remains in the filtered matrix.
- Filtered peak lists for the three studies, in positive and negative modes, feed matchingStage with the same adduct/fragment set; row counts should decrease relative to the raw input.
- Recomputing the sensitivity comparison should reproduce mWISE's FELLA/raw-scores sensitivity mean (SD) of 0.63 (0.07) and xMSannotator's 0.55 (0.19).
- Applying clusterBased.filter after featuresClustering should reduce false positives relative to the unfiltered case, e.g. Assay 2 FP dropping from 17598 to 1671.

## Limitations

- The 80% rule discards features missing in more than 20% of samples, which can remove genuinely low-abundance metabolites and shrink the reference set.
- mWISE only offers the data to annotate in KEGG database, so filtering choices are tuned to a KEGG-based annotation context.
- The filtering parameters described here were tuned for the public LC-MS studies and ionization modes in the article; other datasets may require re-tuning.

## Evidence

- [methods] matching Stage: [evidence span withheld — anchor status: not_found]
