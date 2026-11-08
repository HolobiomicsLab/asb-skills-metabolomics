---
name: connected-component-feature-detection
description: Use when a random scattered-sample optical image (flatbed scanner or
  cell phone camera, samples >300 μm) must be converted into a list of discrete target
  coordinates with per-region morphological measurements before MALDI-TOF MS geometry
  generation.
when_to_use_negative:
- Samples smaller than 300 μm — the article states 'For samples smaller than 300 μm,
  microMS is more effective'
- Images with moderate distortion or imperfect perspective that have not yet passed
  macroMS's optical correction tools, since detected coordinates would map inaccurately
  to stage positions
- Already-curated target lists where features are manually specified; detection/filtering
  would be redundant
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: macroMS
  role: Web platform whose sample-finding and target-filtering functionalities perform
    the connected-component detection and filtering on uploaded optical images
  repo: github.com/macroMS/macroMS_web
- name: microMS
  role: Source code reused by macroMS for feature finding; implements scikit-image-based
    detection of scattered targets
- name: scikit-image
  role: Python library (via reused microMS code) providing thresholding, inversion,
    and connected-region detection with morphological descriptors
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:7d277de07b0c57704567bf85154703620b2ade92c562c3546770a2acb30df3ec
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - macroMS
  - microMS
  - scikit-image
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# connected-component-feature-detection

## Summary

Detects and characterizes individual sample targets in optical images by thresholding/inversion and connected-component labeling with scikit-image, as reused from microMS source code within the macroMS target list generation workflow for optically guided MALDI-TOF MS.

## When to use

Use when a random scattered-sample optical image (flatbed scanner or cell phone camera, samples >300 μm) must be converted into a list of discrete target coordinates with per-region morphological measurements before MALDI-TOF MS geometry generation.

## When NOT to use

- Samples smaller than 300 μm — the article states 'For samples smaller than 300 μm, microMS is more effective'
- Images with moderate distortion or imperfect perspective that have not yet passed macroMS's optical correction tools, since detected coordinates would map inaccurately to stage positions
- Already-curated target lists where features are manually specified; detection/filtering would be redundant

## Inputs

- Optical image of scattered macroscopic samples (flatbed scanner or cell phone camera capture, samples >300 μm)
- Expected target size range, circularity criteria, and nearest-neighbor spacing expectations for the sample population

## Outputs

- Detected target list with coordinates plus per-target morphological measurements (area/size, circularity, nearest-neighbor distances)

## How to apply

Load the sample optical image and isolate features as foreground using scikit-image thresholding followed by image inversion. Label connected regions and compute per-region morphological descriptors: area/size, circularity, and nearest-neighbor distances. Apply macroMS's population filtering rules — retain only regions within the accepted size range, acceptably circular, and consistent with expected nearest-neighbor spacing — to discard debris and merged or broken regions. Reuse of the microMS source code, which implements the scikit-image Python library, is the article's stated implementation basis; downstream steps (optical correction, TSP ordering, geometry emission) build on the filtered coordinate list. Note the workflow assumes the ~1 to 2 s/sample throughput target, so detection should be automated rather than manual.

## Related tools

- **macroMS** (Web platform whose sample-finding and target-filtering functionalities perform the connected-component detection and filtering on uploaded optical images) — github.com/macroMS/macroMS_web
- **microMS** (Source code reused by macroMS for feature finding; implements scikit-image-based detection of scattered targets)
- **scikit-image** (Python library (via reused microMS code) providing thresholding, inversion, and connected-region detection with morphological descriptors)

## Examples

```
from skimage import measure; labels = measure.label(inverted_thresh_img); regions = measure.regionprops(labels)
```

## Evaluation signals

- Detected target list matches the expected target count and locations on the sample image
- Per-region descriptors (area/size, circularity, nearest-neighbor distances) are present for every emitted target
- Filtered population obeys the stated rules: all retained regions fall within the accepted size range, are acceptably circular, and match expected nearest-neighbor spacing
- Total target number stays within the reported ~10,000 upper limit for macroMS
- Detection runs as part of the automated web workflow consistent with the ~2 s/sample throughput claim

## Limitations

- Upper limit for target number was found to be ~10,000; larger sample counts exceed demonstrated capacity
- Reliability demonstrated at 300 μm sample size; smaller samples are better served by microMS
- Optical defects inherent to common imaging devices (moderate distortion, imperfect perspective) must be corrected separately before coordinates can be trusted
- No version information for macroMS is stated and the qualitative microMS comparison lacks quantitative benchmarks

## Evidence

- [abstract] Here, we describe an imaging protocol and macroMS, an online software suite: "Here, we describe an imaging protocol and macroMS, an online software suite"
- [intro] our lab developed the microMS software suite14 for optically guided MS, which enables accurate targeting of cells scattered across a surface such as a microscope slide: "our lab developed the microMS software suite14 for optically guided MS, which enables accurate targeting of cells scattered across a surface"
- [discussion] The platform provides functionalities for sample finding, target filtering, and device coordinate generation through a simple web interface: "The platform provides functionalities for sample finding, target filtering, and device coordinate generation through a simple web interface"
- [discussion] the upper limit for target number was found to be ~10,000: "the upper limit for target number was found to be ~10,000"
