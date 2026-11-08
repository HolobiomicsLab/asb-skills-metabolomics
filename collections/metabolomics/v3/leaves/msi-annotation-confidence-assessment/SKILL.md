---
name: msi-annotation-confidence-assessment
description: Use when evaluating ms1-id annotation results on MALDI-MS imaging imzML/ibd
  data — e.g. deciding which annotations are trustworthy, comparing scaled vs non-scaled
  GNPS library searches, or validating against spotted standards.
when_to_use_negative:
- 'Data is LC-MS: the spectral usage cutoff differs (0.20 for LC-MS vs 0.05 for MS
  imaging) and chromatographic rather than spatial correlation is used'
- You need MS/MS-level library annotation with real tandem spectra — ms1-id MSI annotations
  are pseudo MS/MS-based, unlike METASPACE formula-level (MSI level 4) FDR estimation
- Target compounds are expected to produce most fragments below m/z 100, which are
  excluded from the MS imaging library and often absent from MS imaging data
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: ms1-id
  role: Generates pseudo MS/MS spectra from spatially correlated in-source fragments
    in imzML data and performs precursor-tolerant reverse spectral matching against
    indexed GNPS libraries; its annotation output is what the confidence cutoffs are
    applied to
  repo: https://github.com/Philipbear/ms1_id
- name: METASPACE
  role: Source of the mouse liver imzML/ibd benchmark dataset with seven spotted standards
    (dataset ID 2020-12-07_03h16m14s); also a contrast point as formula-level (MSI
    level 4) annotation via target-decoy FDR
  repo: https://metaspace2020.org/dataset/2020-12-07_03h16m14s
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.10.14.618269
    title: 'Structural annotation of full-scan MS data: A unified solution for LC-MS
      and MS imaging analyses'
  source_content_hash: sha256:16c48f0ca882f91074ab7ccf39b19450250a9900f76f333823cb2570c09f5686
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/Philipbear/ms1_id
  tools:
  - METASPACE
  - ms1-id
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# msi-annotation-confidence-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Judges the confidence of ms1-id pseudo MS/MS annotations on MS imaging (imzML) data by applying the paper's MS imaging search cutoffs (score > 0.7, ≥3 matched peaks, spectral usage ≥ 0.05) and benchmarking against spotted ground-truth standards with and without mass-dependent peak intensity scaling (k=10, sqrt transform).

## When to use

Use when evaluating ms1-id annotation results on MALDI-MS imaging imzML/ibd data — e.g. deciding which annotations are trustworthy, comparing scaled vs non-scaled GNPS library searches, or validating against spotted standards.

## When NOT to use

- Data is LC-MS: the spectral usage cutoff differs (0.20 for LC-MS vs 0.05 for MS imaging) and chromatographic rather than spatial correlation is used
- You need MS/MS-level library annotation with real tandem spectra — ms1-id MSI annotations are pseudo MS/MS-based, unlike METASPACE formula-level (MSI level 4) FDR estimation
- Target compounds are expected to produce most fragments below m/z 100, which are excluded from the MS imaging library and often absent from MS imaging data

## Inputs

- ms1_id_annotations_derep.tsv from ms1-id MSI annotation of imzML/ibd data
- Pseudo MS/MS annotation results (tsv) from `ms1_id annotate` on mgf files
- Indexed GNPS MS/MS libraries for MS imaging: gnps_minmz100.pkl and gnps_minmz100_k10.pkl (peak_scale_k=10, peak_intensity_power=0.5)
- Ground-truth list of spotted/known standards (e.g. the seven mouse liver standards)

## Outputs

- Filtered annotation table of confident hits (score > 0.7, ≥3 matched peaks, spectral usage ≥ 0.05)
- Tabulated comparison of matching scores > 0.7 between scaled (k=10) and non-scaled library searches
- Benchmark verdict of annotated vs missing ground-truth standards (expected: 5 of 7 annotated)
- ms1id_msi_annotations.csv from the standards benchmark task

## How to apply

Annotate imzML data with ms1-id using the MSI-specific library (gnps_minmz100, fragments below m/z 100 removed) and both scaled and non-scaled variants (gnps_minmz100.pkl and gnps_minmz100_k10.pkl), then filter results to the MS imaging cutoffs: minimum score 0.7, minimum matched peaks 3, minimum spectral usage 0.05. Confidence rests on the upstream pseudo MS/MS quality: ion images extracted with a 10 ppm mass tolerance must be spatially correlated with at least 50 shared non-zero pixels and a correlation cutoff of 0.85 before a pseudo MS/MS spectrum is searched. For ground-truth validation (e.g. seven spotted standards on mouse liver tissue: ATP, ADP, adenosine, guanine, glutamic acid, glutamine, glutathione), count reverse cosine scores > 0.7 in both the k=10 scaled and non-scaled conditions; the scaled condition is expected to yield more matches, since scaling aligns low-energy in-source fragments with medium/high-collision-energy reference spectra. Annotations failing to appear for expected standards should be audited for insufficient in-source fragment generation rather than treated as library misses.

## Related tools

- **ms1-id** (Generates pseudo MS/MS spectra from spatially correlated in-source fragments in imzML data and performs precursor-tolerant reverse spectral matching against indexed GNPS libraries; its annotation output is what the confidence cutoffs are applied to) — https://github.com/Philipbear/ms1_id
- **METASPACE** (Source of the mouse liver imzML/ibd benchmark dataset with seven spotted standards (dataset ID 2020-12-07_03h16m14s); also a contrast point as formula-level (MSI level 4) annotation via target-decoy FDR) — https://metaspace2020.org/dataset/2020-12-07_03h16m14s

## Examples

```
ms1_id msi --input_dir msi --libs db/gnps_minmz100.pkl db/gnps_minmz100_k10.pkl --n_cores 12
```

## Evaluation signals

- All reported annotations satisfy the MS imaging cutoffs: minimum score 0.7, minimum matched peaks 3, minimum spectral usage 0.05
- Exactly five of the seven spotted standards (ATP, ADP, adenosine, glutamine, glutathione) are confidently annotated in the mouse liver benchmark; guanine (m/z 150.0421) and glutamic acid (m/z 146.0459) correctly fail
- The peak-scaled (k=10) condition yields a higher count of matching scores > 0.7 for ground-truth compounds than the non-scaled condition
- Pseudo MS/MS generation used ion-image correlations with at least 50 shared non-zero pixels and correlation cutoff 0.85, and extraction used 10 ppm mass tolerance
- Output tables are produced in the expected locations: ms1_id_annotations_derep.tsv (or per-input tsv from `ms1_id annotate`) and the benchmark CSV

## Limitations

- Standards that do not generate sufficient in-source fragment ions within the collected m/z range (guanine, glutamic acid) cannot be confidently annotated — failure is a method limitation, not a library miss
- For MS imaging, fragments with m/z < 100 are removed from the indexed library (and are not usually captured in MS imaging data), capping detectable fragment coverage
- Pseudo MS/MS spectra derive from MS1 in-source fragments, so annotation confidence depends on how well low-energy fragments match reference spectra, which motivated the mass-dependent peak scaling step

## Evidence

- [readme] MSI library excludes fragments below m/z 100: [evidence span withheld — anchor status: external_doc]
- [readme] Where annotation results are delivered for confidence assessment: [evidence span withheld — anchor status: external_doc]
- [results] Known unannotatable standards in the mouse liver benchmark: [evidence span withheld — anchor status: not_found]
