---
name: la-icp-ms-image-noise-mitigation
description: Use when an imported LA-ICP-MS elemental image (e.g. 66Zn, 56Fe, 55Mn
  from Agilent/Thermo/PerkinElmer data) shows isolated pixel-scale spikes or regions
  of extremely low or high signal from instrument noise, and you want to suppress
  them without smoothing the rest of the image.
when_to_use_negative:
- Input is already a filtered or calibrated image where spikes have been removed —
  reapplying the filter will not change valid pixels and could smooth real features.
- Spike removal is the wrong choice when the goal is drift correction across the acquisition
  direction; use Pew2's drift tool (polynomial renormalization within a user-defined
  background slice) instead.
- The artifact spans a contiguous region larger than the chosen local window — a local
  median filter cannot distinguish extended features from noise.
edam_operation: http://edamontology.org/operation_3807
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Pew2
  role: GUI Filtering tool that applies a local (mean or median) filter at the position
    of each spike with a user-defined threshold
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Python library used to script batch conversion and process/filter LA-ICP-MS
    image data outside the GUI
  repo: https://github.com/djdt/pewlib
- name: NumPy
  role: Array representation used to hold the image during filtering and to save the
    filtered result as a NumPy archive
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:3a8d421872272e3d9903e53908d67d8ff9a11028ae69deb0637c7899d2e40203
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djdt/pewpew
  tools:
  - NumPy
  - Pew2
  - pewlib
  edam_operation: http://edamontology.org/operation_3807
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# la-icp-ms-image-noise-mitigation

## Summary

Mitigate instrument-noise artifacts in LA-ICP-MS elemental images by identifying individual pixel spikes against a local (mean or median) filter and replacing only the outlying values, using Pew2's Filtering tool or pewlib arrays. This preserves genuine spatial signal while removing sporadic high/low excursions caused by the instrument.

## When to use

Use when an imported LA-ICP-MS elemental image (e.g. 66Zn, 56Fe, 55Mn from Agilent/Thermo/PerkinElmer data) shows isolated pixel-scale spikes or regions of extremely low or high signal from instrument noise, and you want to suppress them without smoothing the rest of the image.

## When NOT to use

- Input is already a filtered or calibrated image where spikes have been removed — reapplying the filter will not change valid pixels and could smooth real features.
- Spike removal is the wrong choice when the goal is drift correction across the acquisition direction; use Pew2's drift tool (polynomial renormalization within a user-defined background slice) instead.
- The artifact spans a contiguous region larger than the chosen local window — a local median filter cannot distinguish extended features from noise.

## Inputs

- Elemental image from LA-ICP-MS line-by-line or spot-wise acquisition imported into Pew2 (raw vendor data, text image, or file-per-line export)
- NumPy array image produced via pewlib import

## Outputs

- Filtered elemental image with spikes replaced by the local mean or median, with valid pixels unchanged
- NumPy archive (.npz) of the filtered image

## How to apply

Load the unfiltered elemental image into Pew2 (or the pewlib/NumPy arrays produced by the import step). Open Pew2's Filtering tool and select the local mean or median filter (the rolling median option's threshold is expressed as distance in medians from the local median). Pick a local window size — the card suggests windows such as 3, 5 or 7 — balancing sensitivity to outliers against the risk of introducing artifacts. Apply the filter at the position of each spike: pixels whose value exceeds the user-defined threshold relative to the local statistic are replaced with the local mean or median, while valid pixels remain untouched. Save the result as a NumPy archive so the filtered image can be re-imported or exported without recomputing.

## Related tools

- **Pew2** (GUI Filtering tool that applies a local (mean or median) filter at the position of each spike with a user-defined threshold) — https://github.com/djdt/pewpew
- **pewlib** (Python library used to script batch conversion and process/filter LA-ICP-MS image data outside the GUI) — https://github.com/djdt/pewlib
- **NumPy** (Array representation used to hold the image during filtering and to save the filtered result as a NumPy archive)

## Examples

```
pewpew
```

## Evaluation signals

- Valid pixels remain unchanged after filtering (compare pre/post arrays pixel-by-pixel outside spike positions).
- Isolated spike positions are replaced with the local median/mean value, not a global statistic.
- Filtered image has no visible regions of extremely low or high signal attributable to instrument noise.
- Aspect ratio and scale bar are preserved (laser spot size and speed were entered correctly after import).
- Saved output loads back as a NumPy archive without schema errors.

## Limitations

- Threshold and window size are user-defined; an inappropriate threshold can either miss spikes or replace genuine signal.
- The filter acts at the position of each spike, so detection depends on the spike being distinguishable from the local statistic.
- Pew2 must have the correct laser spot size and speed entered after import for correct image aspect and scale bar sizing.

## Evidence

- [supplementary] Selective filtering of instrumental artifacts: [evidence span withheld — anchor status: not_found]
- [results] Local filter at spike positions: [evidence span withheld — anchor status: not_found]
- [abstract] Pew2 GUI for LA-ICP-MS processing: [evidence span withheld — anchor status: not_found]
- [methods] pewlib for scripted processing: [evidence span withheld — anchor status: not_found]
