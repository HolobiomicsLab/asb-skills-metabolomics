---
name: three-level-kmeans-image-segmentation
description: Use when an LA-ICP-MS elemental image (e.g. 56Fe from a MassHunter batch)
  needs automatic background/ROI separation without hand-set thresholds, as in Pew2's
  segmentation tool.
when_to_use_negative:
- The input is already a segmented or labeled image — Pew2's segmentation step is
  meant to produce the mask from a raw elemental image.
- Spot-wise or line-by-line data lacking a spatial image grid — segmentation operates
  on 2D images; use Pew2's spot-wise/line-by-line import modes instead.
- Supervised ROI definition where user-defined thresholds are required — use the manual
  thresholding path in Pew2 rather than the unsupervised k-means path.
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: Pew2
  role: Host application providing the segmentation tool with built-in k-means, mean,
    and Otsu thresholding used to produce ROI masks from elemental images.
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Python library implementing import/export and processing of LA-ICP-MS data;
    can be used to batch-convert data prior to segmentation.
  repo: https://github.com/djdt/pewlib
- name: NumPy
  role: Backend array format used by Pew2; also the format of the precalibrated microarray-core
    export fed to the segmentation routine.
  repo: https://github.com/djdt/pewpew
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:79047c182446d8c96e6b67a745e9f6db6947bad9e3c62cf5afcc59e0f4422ad6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djdt/pewpew
  tools:
  - NumPy
  - Pew2
  - Python
  - pewlib
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# three-level-kmeans-image-segmentation

## Summary

Unsupervised three-level k-means segmentation of LA-ICP-MS elemental images (e.g. 56Fe) in Pew2 to isolate ROIs from background for downstream feature selection, without manual threshold tuning.

## When to use

Use when an LA-ICP-MS elemental image (e.g. 56Fe from a MassHunter batch) needs automatic background/ROI separation without hand-set thresholds, as in Pew2's segmentation tool.

## When NOT to use

- The input is already a segmented or labeled image — Pew2's segmentation step is meant to produce the mask from a raw elemental image.
- Spot-wise or line-by-line data lacking a spatial image grid — segmentation operates on 2D images; use Pew2's spot-wise/line-by-line import modes instead.
- Supervised ROI definition where user-defined thresholds are required — use the manual thresholding path in Pew2 rather than the unsupervised k-means path.

## Inputs

- LA-ICP-MS elemental image in Pew2 (e.g. 56Fe channel imported from an Agilent MassHunter batch, Perkin Elmer Elan 'XL', Thermo Qtegra export, or delimited-text image)
- Optional precalibrated NumPy export of a microarray core as supplied in the supplementary material

## Outputs

- Binary mask separating ROI (tissue) from background for the given elemental image
- Segmented elemental image suitable for background removal and downstream image feature selection

## How to apply

In Pew2, load an elemental image (e.g. 56Fe imported from an Agilent MassHunter batch via drag-and-drop) and run the built-in k-means thresholding with three levels: segment(Fe56, kmeans(Fe56, 3)). Extract the highest k level from the k-means bounds array and apply it as a mask via mask(Fe56, masked == 3), yielding a binary mask that separates tissue from background. This masks output can then be used for background removal and image feature selection. Because the algorithm is unsupervised, the key decision point is accepting the three-level result without further manual threshold tuning; if the mask pattern (cf. Figure 5(b)) does not cleanly separate the tissue from the background on the 56Fe image, fall back to alternative built-in options such as mean thresholding or Otsu's method, which are offered alongside k-means in the same segmentation tool.

## Related tools

- **Pew2** (Host application providing the segmentation tool with built-in k-means, mean, and Otsu thresholding used to produce ROI masks from elemental images.) — https://github.com/djdt/pewpew
- **pewlib** (Python library implementing import/export and processing of LA-ICP-MS data; can be used to batch-convert data prior to segmentation.) — https://github.com/djdt/pewlib
- **NumPy** (Backend array format used by Pew2; also the format of the precalibrated microarray-core export fed to the segmentation routine.) — https://github.com/djdt/pewpew

## Examples

```
from pewlib import import_masshunter; from pewlib.calc import segment, kmeans, mask; img = import_masshunter('batch.b/56Fe'); mask(img, segment(img, kmeans(img, 3)) == 3)
```

## Evaluation signals

- The three-level k-means mask (Figure 5(b)) separates tissue from background on the 56Fe image without any manual threshold adjustment.
- The mask pattern for the pancreatic adenocarcinoma microarray core matches the pattern reported for Pew2 segmentation of the same core in the paper.
- Applying the mask removes the background signal and leaves the elemental-feature image (e.g. 56Fe) intact for feature selection.
- If the mask is unsatisfactory, the alternative built-in mean-thresholding or Otsu's-method outputs can be compared against it as sanity checks.

## Limitations

- K-means is one of several offered segmentation options; the article does not report quantitative benchmarking against the alternatives for this particular image.
- Requires an image (2D) input; spot-wise or line-by-line data without a spatial grid cannot be segmented with this skill.
- Mask quality is not validated beyond the visual comparison shown in the paper's Figure 5; users must inspect masks for their own data.

## Evidence

- [results] The masks produced by each algorithm can be used for removal: [evidence span withheld — anchor status: not_found]
