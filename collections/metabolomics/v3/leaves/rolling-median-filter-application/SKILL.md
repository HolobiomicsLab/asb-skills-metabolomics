---
name: rolling-median-filter-application
description: Use when an imported LA-ICP-MS elemental image in Pew2 contains discrete
  high- or low-intensity spikes from instrument noise and you need to selectively
  replace those outliers without distorting surrounding valid pixels.
when_to_use_negative:
- The data requires uniform smoothing across the entire image rather than selective
  removal of discrete spikes — a blanket mean/median convolution would alter valid
  pixels, whereas this skill's threshold logic is designed to leave them untouched
- The image already contains calibration or segmentation results that would be invalidated
  by re-filtering the underlying raw intensities
- Noise is broad-band or baseline-level rather than discrete spikes; the Pew2 drift-correction
  or standardization tools, not the spike filter, are the appropriate choice
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Pew2
  role: GUI application providing the Filtering tool with the Rolling Median option,
    threshold definition, and window-size selection
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Python library that implements the import and export of LA-ICP-MS data so
    filtered images can be batch-processed or saved programmatically
  repo: https://github.com/djdt/pewlib
- name: NumPy
  role: Underlying array representation of the elemental image; filtered output is
    a NumPy archive
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:317f8c6f2d55d5916c768b26e6bfa295f22890a52e012dabb67d73a80b24dffb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djdt/pewpew
  tools:
  - NumPy
  - Pew2
  - pewlib
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# rolling-median-filter-application

## Summary

Applies a local (rolling) median filter within Pew2 to remove instrument-noise spikes from LA-ICP-MS elemental images: each pixel is compared against a user-defined threshold relative to the local median, and only outlying values are replaced by the local median, leaving valid pixels untouched.

## When to use

Use when an imported LA-ICP-MS elemental image in Pew2 contains discrete high- or low-intensity spikes from instrument noise and you need to selectively replace those outliers without distorting surrounding valid pixels.

## When NOT to use

- The data requires uniform smoothing across the entire image rather than selective removal of discrete spikes — a blanket mean/median convolution would alter valid pixels, whereas this skill's threshold logic is designed to leave them untouched
- The image already contains calibration or segmentation results that would be invalidated by re-filtering the underlying raw intensities
- Noise is broad-band or baseline-level rather than discrete spikes; the Pew2 drift-correction or standardization tools, not the spike filter, are the appropriate choice

## Inputs

- Unfiltered LA-ICP-MS elemental image imported into Pew2 (Agilent batch directory .b, Perkin Elmer Elan .xl, Thermo iCap CSV/LDR, Nu Instruments Vitesse, imzML, or CSV/text image)
- User-defined spike threshold expressed in medians from the local median
- Local window size (e.g., 3, 5, or 7 pixels)

## Outputs

- Filtered elemental image with spike values replaced by local median values
- Filtered image saved as a NumPy archive (e.g., .npz via pewlib export)

## How to apply

Load the unfiltered elemental image into Pew2 via drag-and-drop import (e.g., Agilent MassHunter batch, Perkin Elmer Elan 'XL', Thermo Fischer Qtegra export, or a text image/file-per-line CSV). Open the Filtering tool and select the 'Rolling Median' filter, whose threshold is defined as distance in medians from the local median. Choose a local window size (e.g., 3, 5, or 7) balancing sensitivity to genuine outliers against risk of introducing smoothing artifacts. Apply the filter: at each spike position, Pew2 compares the pixel value against the local median-derived threshold and replaces only values exceeding that threshold with the local median, leaving valid pixels unchanged. The article notes that spikes and regions of extremely low or high signal 'may be removed by applying a local (mean or median) filter at the position of each spike', so the tool is selective rather than a uniform smoothing operation.

## Related tools

- **Pew2** (GUI application providing the Filtering tool with the Rolling Median option, threshold definition, and window-size selection) — https://github.com/djdt/pewpew
- **pewlib** (Python library that implements the import and export of LA-ICP-MS data so filtered images can be batch-processed or saved programmatically) — https://github.com/djdt/pewlib
- **NumPy** (Underlying array representation of the elemental image; filtered output is a NumPy archive)

## Examples

```
pewpew  # open Filtering tool → Rolling Median; threshold in medians, window 3–7
```

## Evaluation signals

- Valid pixels adjacent to spikes remain numerically identical after filtering (only spike positions are modified)
- The frequency distribution of replaced values matches the local median of the chosen window, not the local mean
- Visual inspection of the filtered image shows isolated spikes removed while tissue-level elemental gradients (e.g., Mn/Zn colocalization regions) are preserved
- The chosen window size does not introduce noticeable smoothing artifacts in otherwise homogeneous regions
- Filtered image exports cleanly as a NumPy archive and reloads into Pew2 with correct aspect/scale-bar metadata intact

## Limitations

- Spike detection depends on a user-defined threshold relative to the local median; a threshold too low will replace genuine biological signal, too high will leave spikes in place
- The filter is applied only 'at the position of each spike', so it does not correct broad drift or baseline artifacts — Pew2's drift tool should be used for that
- Filtering is a local operation; it does not use calibration standard data or isotopic context

## Evidence

- [results] Spikes in data and regions of extremely low or high signal resulting from instrument noise may be removed by applying a local (mean or median) filter at the position of each spike.: [evidence span withheld — anchor status: not_found]
- [readme] Drag-and-drop import of batches (.b directories), with one sample per line: [evidence span withheld — anchor status: external_doc]
