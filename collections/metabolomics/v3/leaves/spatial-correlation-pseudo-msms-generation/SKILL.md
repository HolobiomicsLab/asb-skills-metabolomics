---
name: spatial-correlation-pseudo-msms-generation
description: Use when the input is full-scan (MS1-only) MS imaging data in imzML/ibd
  format — e.g. MALDI-Orbitrap/FTICR datasets — and structural annotation at a level
  beyond molecular formula is needed without any MS/MS scans.
when_to_use_negative:
- The dataset already contains MS/MS scans — use direct MS/MS library matching instead.
- The analysis is chromatographic LC-MS rather than spatial MS imaging — use the chromatographic
  (XIC) correlation variant instead.
- Features fail the spatial co-localization test (fewer than 50 shared non-zero pixels
  or correlation < 0.85), so no reliable composite spectrum can be assembled.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3518
tools:
- name: ms1-id
  role: Python package that implements spatial correlation of ion images to generate
    pseudo MS/MS spectra from full-scan imaging MS data and annotate them against
    MS/MS libraries.
  repo: https://github.com/Philipbear/ms1_id
- name: METASPACE
  role: Public repository from which the MS imaging datasets (e.g. mouse liver with
    spotted standards) are downloaded before running the correlation workflow.
  repo: https://metaspace2020.org
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.10.14.618269
    title: 'Structural annotation of full-scan MS data: A unified solution for LC-MS
      and MS imaging analyses'
  source_content_hash: sha256:d225d6ef0e32fef95a1b3fd24d6aa29053f8394fd8a1525d3af2f9eabfa7ce3f
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# spatial-correlation-pseudo-msms-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates pseudo MS/MS (composite) spectra from MS1-only MS imaging data by spatially correlating ion images across imzML pixels, so that in-source fragments co-localized with a precursor feature can be assembled into a composite spectrum for downstream MS/MS library annotation.

## When to use

Use when the input is full-scan (MS1-only) MS imaging data in imzML/ibd format — e.g. MALDI-Orbitrap/FTICR datasets — and structural annotation at a level beyond molecular formula is needed without any MS/MS scans.

## When NOT to use

- The dataset already contains MS/MS scans — use direct MS/MS library matching instead.
- The analysis is chromatographic LC-MS rather than spatial MS imaging — use the chromatographic (XIC) correlation variant instead.
- Features fail the spatial co-localization test (fewer than 50 shared non-zero pixels or correlation < 0.85), so no reliable composite spectrum can be assembled.

## Inputs

- imzML + ibd MS imaging dataset (full-scan MS1 only)
- detected MS1 feature list / ion m/z values for which ion images are extracted
- preprocessed per-pixel MS spectra (noise-filtered, baseline-corrected, centroided)

## Outputs

- pseudo MS/MS (composite) spectra per precursor feature, in MGF format suitable for library search
- ion images per feature used for spatial correlation
- list of in-source fragment ions correlated to each precursor feature

## How to apply

Load the imzML/ibd dataset and preprocess each MS scan with moving-average noise reduction, apply a 100 Da moving-window baseline (5× mean of the lowest 5% ions), and centroid the peaks if needed. Extract ion images for each detected feature using a 10 ppm mass tolerance. Spatially correlate pairs of ion images requiring at least 50 shared pixels with non-zero intensities between the two ion images, and use a correlation cutoff of 0.85 to decide which ions are fragments of the precursor feature; exclude isotope peaks (determined by MassCube) from the pseudo MS/MS generation. The rationale is that in/post-source fragments co-localized with the same molecular feature in the spatial domain constitute a composite spectrum that can be reverse-matched against MS/MS libraries. Validate the generated pseudo MS/MS spectra before library search, e.g. by checking that a known-spotted standard (such as ATP or adenosine on mouse liver) receives a coherent composite spectrum with a minimum of 50 shared pixels and 0.85 correlation.

## Related tools

- **ms1-id** (Python package that implements spatial correlation of ion images to generate pseudo MS/MS spectra from full-scan imaging MS data and annotate them against MS/MS libraries.) — https://github.com/Philipbear/ms1_id
- **METASPACE** (Public repository from which the MS imaging datasets (e.g. mouse liver with spotted standards) are downloaded before running the correlation workflow.) — https://metaspace2020.org

## Examples

```
ms1_id msi --input_dir msi --libs db/gnps_minmz100.pkl db/gnps_minmz100_k10.pkl --n_cores 12
```

## Evaluation signals

- Expected that 5 of 7 spotted chemical standards (ATP, ADP, adenosine, glutamine, glutathione) on mouse liver tissue are successfully annotated after generating pseudo MS/MS spectra; guanine and glutamic acid are expected to fail due to insufficient in-source fragments within the collected m/z range.
- Correlation cutoff invariant: every included fragment ion pair must share ≥ 50 pixels with non-zero intensity and a Pearson correlation ≥ 0.85.
- m/z tolerance invariant: ion images are extracted at a 10 ppm mass tolerance from the imzML data.
- Isotope-exclusion invariant: peaks flagged as isotopes by MassCube are absent from the generated pseudo MS/MS spectra.
- Generated pseudo MS/MS spectra are written as MGF files that are parseable by the downstream ms1-id annotate/library-search step.

## Limitations

- The approach requires sufficient in-source fragment ions within the collected m/z range; guanine (m/z 150.0421) and glutamic acid (m/z 146.0459) could not be confidently annotated because they did not generate sufficient in-source fragments above m/z 100 in the demonstrated dataset.
- Almost all MS imaging metabolomics data are MS1 scans, so the composite spectrum quality depends entirely on the co-localization signal — poor spatial separation yields unreliable pseudo MS/MS spectra.
- Fragment ions with m/z < 100 are removed for MS imaging library indexing, as they are not usually included in MS imaging data, limiting matching for compounds whose key fragments fall below 100 Da.
- The pseudo MS/MS spectra are composite, so reverse matching against high-energy reference MS/MS libraries requires the separate mass-dependent peak-intensity-scaling step to be effective.

## Evidence

- [methods] Correlation cutoff of 0.85 used to assemble pseudo MS/MS spectra in the spatial domain.: [evidence span withheld — anchor status: not_found]
- [methods] Ion images are extracted using a 10 ppm mass tolerance.: [evidence span withheld — anchor status: not_found]
- [methods] MS1 imaging scans are preprocessed with moving-average noise reduction and 100 Da moving-window baseline.: [evidence span withheld — anchor status: not_found]
- [results] Five of seven spotted standards annotated; guanine and glutamic acid fail for lack of sufficient in-source fragments.: [evidence span withheld — anchor status: not_found]
- [results] Guanine and glutamic acid could not be confidently annotated because they did not generate sufficient in-source fragment ions within the collected m/z range.: [evidence span withheld — anchor status: not_found]
- [readme] MS imaging library indexing removes fragments with m/z < 100.: [evidence span withheld — anchor status: external_doc]
- [readme] ms1_id annotates imaging MS data in imzML files.: [evidence span withheld — anchor status: external_doc]
